# 데이터 정합성 — OCR 적용 사례

## 1. 가정하는 데이터 모델

실제 운영 구현을 확인한 기록이 아니라 학습용 설계다. 상태의 원본은 관계형 DB, 입력·큰 결과는 객체 저장소, 작업 전달은 Kafka라고 가정한다.

| 데이터 | 주요 필드 | 보호 규칙 |
|---|---|---|
| jobs | job_id, tenant_id, request_hash, model_version, status, attempt, lease_until, next_retry_at | 요청 키 UNIQUE, 조건부 상태 전이 |
| results | job_id, model_version, result_uri, checksum | job_id와 model_version 조합 UNIQUE |
| outbox | event_id, job_id, event_type, payload, published_at | job 변경과 같은 DB 트랜잭션 |

## 2. 정상 처리 흐름

1. API는 불변 입력 참조와 모델 버전을 검증한다.
2. jobs와 `JobRequested` outbox 행을 같은 트랜잭션에 저장한다.
3. Relay가 outbox를 Kafka로 전달한다. 중복 전달은 허용한다.
4. Worker가 상태와 attempt를 원자적으로 획득하고 추론한다.
5. 결과 파일을 attempt별 경로에 저장하고 저장 성공을 확인한다.
6. 짧은 DB 트랜잭션에서 소유권을 검증하고 results와 성공 상태를 확정한다.
7. Kafka offset을 커밋한다. 캐시·통계는 비동기로 반영한다.

## 3. 장애별 결과

| 장애 시점 | 남은 상태 | 복구 |
|---|---|---|
| jobs 저장 후 이벤트 전송 전 | 미발행 outbox 존재 | Relay 재전송 |
| 추론 중 Worker 종료 | RUNNING과 만료 예정 lease | 만료 스캐너가 재시도 예약 |
| 결과 파일 저장 후 DB 확정 전 | 참조 없는 파일 가능 | 재시도, 유예 기간 후 고아 파일 정리 |
| DB 확정 후 offset commit 전 | 이미 성공한 job | 재전달 시 성공 확인 후 commit |
| 캐시 갱신 실패 | 원본만 최신 | 재전달 또는 원본 재구축 |

## 4. API 계약

접수 응답의 `202 Accepted`는 추론 완료가 아니라 영속적인 작업 접수를 뜻한다. 조회는 현재 상태, 재시도 가능 여부, 완료 시 결과 참조를 제공한다. 결과가 준비되지 않았다면 성공으로 표시하지 않는다.

완료 직후 결과 조회에 일관성이 필요하면 원본 DB를 읽는다. 결과 파일 다운로드 실패는 작업 상태와 별도로 관측하고 저장소 장애를 조사한다.

## 5. 완료 기준

각 장애 지점에서 프로세스를 종료해도 작업이 성공 또는 명시적인 최종 실패로 수렴해야 한다. 성공 작업의 결과 누락, 오래된 Worker의 덮어쓰기, 무기한 RUNNING이 없어야 한다.

연결: [메시징 흐름](../02_messaging_eda/applied-to-ocr.md), [복구 전략](../03_reliability_engineering/recovery-strategy.md)

# 📌 AI Model Serving 아키텍처 학습 로드맵 (심화 버전)

이 로드맵은 단순 개념 학습이 아니라,
분산 환경에서 안전한 AI 모델 서빙 시스템을 설계할 수 있는 역량 확보를 목표로 한다.


목차
---
1. 문제 인식 능력
2. 데이터 정합성 전략
3. 메시지 처리 의미론
4. 상태 전이 안전 설계
5. 장애 예측 설계
6. 복구 전략 설계
7. AI 모델 서빙 특화 안정성
8. 아키텍처 의사결정 프레임워크
---

# 🎯 1. 문제 인식 능력

## 질문
- 이 시스템은 어디서 깨질 수 있는가?
- 중복 실행은 언제 발생하는가?
- 데이터는 어디서 꼬일 수 있는가?

## 학습 항목
- 분산 환경에서 발생하는 장애 유형
- 네트워크 분리 상황
- 재시도 발생 원인
- 중복 메시지 발생 원인

---

# 🟡 2. 데이터 정합성 전략

## 핵심 질문
- 어떤 읽기와 쓰기에 강한 일관성이 필요한가?
- 어떤 반복 요청의 효과를 멱등하게 만들어야 하는가?
- 조건부 업데이트로 해결 가능한가?

## 학습 항목
- Strong vs Eventual
- Idempotency Key 설계
- CAS(조건부 업데이트)
- Optimistic / Pessimistic Lock

## 적용 목표
OCR 상태 전이 보호

---

# 🟠 3. 메시지 처리 의미론

## 핵심 질문
- At Least Once의 위험은?
- Exactly Once는 왜 어려운가?

## 학습 항목
- Kafka Offset Commit 전략
- Consumer 재시작 시 동작
- DLQ 패턴
- Retry 전략

## 적용 목표
중복 처리에도 안전한 Consumer 설계

---

# 🔵 4. 상태 전이 안전 설계

## 핵심 질문
- 상태가 중복 변경되면 어떻게 되는가?
- 상태 전이 조건을 어떻게 보호할 것인가?

## 학습 항목
- 상태 머신 설계
- 조건부 업데이트
- 멱등 상태 전이 전략

---

# 🟣 5. 장애 예측 설계

## 핵심 질문
- 장애는 언제 감지 가능한가?
- 어떤 지표를 조합해야 하는가?

## 학습 항목
- Composite Alarm
- Kafka Lag 기반 판단
- GPU Utilization 기반 판단
- Latency 분산 분석

---

# 🔴 6. 복구 전략 설계

## 핵심 질문
- 자동 복구 가능한가?
- 수동 개입 기준은?

## 학습 항목
- Runbook 설계
- Failover 전략
- Scale Out 전략
- 구조 전환 전략 (Standard → Batch → Async Writer)

---

# ⚫ 7. AI Model Serving 특화 안정성

## 핵심 질문
- GPU는 왜 병목이 되는가?
- 직렬화와 배치는 어떻게 다른가?

## 학습 항목
- GPU 직렬화
- Micro-batching
- Shape Bucket 전략
- Latency vs Throughput Tradeoff

---

# 🧠 8. 아키텍처 의사결정 프레임워크

## 핵심 질문
- 비용 vs 안정성 중 무엇을 우선할 것인가?
- 동기에서 비동기로 전환해야 하는 시점은?

## 학습 항목
- Strong 적용 판단 기준
- Idempotent 적용 기준
- Async 전환 판단 기준
- 운영 전환 전략

---

# 🏁 최종 목표

기술을 사용하는 개발자가 아니라,

구조를 판단하고,
위험을 예측하고,
Tradeoff를 설명할 수 있는 설계자 수준에 도달


## 사전 학습: Level 0과 분산 시스템 기본기

HTTP timeout은 서버의 처리 실패를 뜻하지 않을 수 있다. DB 트랜잭션, UNIQUE 제약, 프로세스·스레드 차이, 큐와 backpressure를 먼저 익힌다. 네트워크 분리 중 응답 가능성과 최신 상태 보장이 충돌하는 상황을 설명하고, 메시지 순서·트랜잭션 격리·선형화 가능성을 구분한다.

## 실습 로드맵과 완료 기준

기간은 고정하지 않고 각 단계의 검증 결과를 남긴 뒤 다음 단계로 진행한다. 아래는 앞으로 수행할 실습이며 완료된 테스트 목록이 아니다.

| 순서 | 읽을 문서 | 실습 산출물 | 완료 기준 |
|---|---|---|---|
| 1 | [정합성](01_data_consistency/strong-vs-eventual.md), [멱등성](01_data_consistency/idempotency.md) | 동일 키 동시 요청 결과 | 논리 job 하나, 다른 입력 충돌 검출 |
| 2 | [상태 전이](01_data_consistency/state-transition-safety.md) | 두 Worker 경쟁·lease 만료 로그 | 오래된 attempt의 완료 거절 |
| 3 | [Offset](02_messaging_eda/offset-commit-strategy.md) | 저장 후 종료·역순 완료 실험 | 미완료 offset 누락 없음 |
| 4 | [Retry/DLQ](02_messaging_eda/dlq-retry-pattern.md), [Outbox](02_messaging_eda/applied-to-ocr.md) | 전송 중 종료와 poison 입력 실험 | 정상 작업 진행, 실패 작업 추적 가능 |
| 5 | [알람](03_reliability_engineering/alert-design.md), [복구](03_reliability_engineering/recovery-strategy.md) | 대시보드·runbook·장애 기록 | 탐지와 복구 시간 측정 |
| 6 | [Batching](04_ai_model_serving/micro-batching.md), [성능](04_ai_model_serving/performance-tradeoff.md) | 입력 분포별 비교 표 | 지연·처리량·메모리 tradeoff 설명 |
| 7 | [ADR](99_architecture_decisions/strong-vs-idempotent-decision.md) | 선택지·근거·재검토 조건 | 측정과 불변식으로 선택 정당화 |

## 최종 설계 리뷰 질문

- 성공 상태인데 결과가 없는 순간이 생길 수 있는가?
- 메시지를 두 번 읽거나 Worker가 늦게 돌아오면 무엇이 보호하는가?
- DB 저장과 Kafka 발행 사이에서 종료되어도 복구되는가?
- 입력 lag가 감소했는데 결과 저장은 멈춰 있을 수 있는가?
- 증설할 자원을 어떤 지표로 선택했고, 효과를 어떻게 검증하는가?
- 결과 토픽이나 캐시가 지연될 때 사용자에게 어떤 상태를 반환하는가?

이 질문에 정상 경로와 실패 경로를 모두 그려 설명하고, 각 주장에 실험 결과를 연결하는 것을 학습 완료 기준으로 삼는다.

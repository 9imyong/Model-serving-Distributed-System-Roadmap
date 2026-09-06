# Kafka Message Semantics

## 1. 무엇을 몇 번 처리한다는 뜻인가?

| 의미론 | 전형적인 순서 | 실패 시 결과 |
|---|---|---|
| At most once | offset commit → 처리 | 처리 전 종료하면 건너뜀 |
| At least once | 영속적인 처리 → offset commit | commit 전 종료하면 재처리 |
| Exactly once | 지원되는 트랜잭션 경계 안에서 출력과 offset 원자 확정 | 해당 경계의 중복 효과 방지 |

Kafka를 사용한다는 사실만으로 애플리케이션 전체의 전달 보장이 결정되지는 않는다. Producer 설정, 복제·보존 정책, Consumer 처리 순서, 외부 저장소가 함께 영향을 준다.

## 2. Exactly once의 범위

Kafka transaction은 Kafka 출력 레코드와 입력 offset을 함께 확정할 수 있다. 소비자는 `read_committed`로 커밋된 트랜잭션의 레코드를 읽는다. 외부 PostgreSQL 저장이나 객체 업로드는 이 트랜잭션에 자동 참여하지 않는다. Producer idempotence도 새로운 애플리케이션 요청의 비즈니스 중복을 제거하는 장치는 아니다. [Kafka 설계 문서](https://kafka.apache.org/41/design/design/)

OCR에서는 GPU 추론이 반복될 수 있음을 받아들이고, 최종 DB 효과를 멱등하게 만드는 구성이 출발점이다.

## 3. 순서와 병렬성

일반적인 Consumer Group에서 파티션은 한 시점에 그룹 내 한 Consumer에 할당된다. 순서는 파티션 내부에 존재하며 전체 토픽에 대한 전역 순서는 없다. 같은 job_id를 key로 쓰더라도 파티션 수 변경이나 다른 토픽을 거치는 재시도에는 추가 주의가 필요하다.

한 Consumer가 메시지를 병렬 처리하면 완료 순서는 읽은 순서와 달라질 수 있다. [offset 전략](offset-commit-strategy.md)에서 미완료 구간을 건너뛰지 않는 규칙을 적용한다.

## 4. 실습

결과 저장 직후 프로세스를 종료한다. 재시작 시 동일 메시지가 다시 읽히는지, 결과는 하나로 유지되는지 관찰한다. 이어서 처리 전에 commit하는 실험을 격리된 환경에서 수행하고 작업이 누락되는 차이를 기록한다.

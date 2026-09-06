# ADR-002: Async Writer 분리 조건

- 상태: 제안 — 저장 병목 측정 후 채택 여부 결정
- 범위: 추론 완료부터 DB 결과 확정까지

## 배경

동기 경로에서는 Worker가 결과 저장을 기다리는 동안 다음 추론을 진행하지 못할 수 있다. 추론과 저장의 독립 확장이 필요할 때 결과 토픽과 Writer를 분리하는 안을 검토한다. 우선 단계별 지연을 측정하고 DB 쿼리·연결 풀 문제를 해결한다.

## 선택지

| 방식 | 장점 | 비용 |
|---|---|---|
| Worker가 직접 DB 저장 | 완료 의미와 실패 경계가 단순 | 저장 지연이 Worker에 전파 |
| 메모리 큐 + 저장 thread | 프로세스 내부 중첩 처리 | 큐만 믿고 commit하면 종료 시 유실 |
| Kafka 결과 토픽 + Writer | 영속 버퍼, 저장 계층 독립 확장 | 추가 토픽·지연·상태 관리 |

## 제안 흐름

```text
추론 Worker → 불변 결과 파일 저장 → 결과 이벤트 + 입력 offset 원자 확정
결과 토픽 → Writer → 소유권 검증 + DB 결과/성공 확정 → 결과 offset commit
```

Kafka transaction은 Kafka 이벤트와 offset의 경계만 보호한다. Writer의 외부 DB 효과에는 계속 멱등성이 필요하다. [Kafka 전달 의미론](https://kafka.apache.org/41/design/design/)

## 상태와 소유권을 어떻게 넘기는가?

이 저장소의 기본 상태 머신은 동기 저장 경로다. Async Writer를 채택하려면 `RESULT_PENDING` 상태와 다음 복구 규약을 먼저 추가해야 한다.

1. 추론 Worker가 파일 저장 후 현재 attempt를 검사하여 DB에 결과 참조와 RESULT_PENDING, 결과 발행 outbox를 함께 기록한다.
2. 이 DB commit을 입력 메시지 처리 완료 경계로 삼아 입력 offset을 커밋할 수 있다. Relay는 결과 이벤트를 전달한다.
3. Writer는 RESULT_PENDING과 attempt를 검증하고 결과·SUCCEEDED를 함께 확정한다.
4. 스캐너는 RESULT_PENDING을 재추론하지 않고 미발행 outbox와 Writer 처리를 복구한다.

위 규약은 앞의 Kafka transaction 중심 흐름에 대한 **DB outbox 대안**이다. 두 흐름을 혼합해 이중 발행하지 않는다. 구체 구현에서는 DB 중심 상태와 맞는 outbox 대안을 우선 검증한다. RUNNING lease가 Writer 대기 중 만료되는 문제를 상태와 책임 인계로 해소해야 한다.

## 채택·되돌리기 기준

저장 대기가 실제 병목이고 end-to-end deadline에 여유가 있을 때 검토한다. Writer 완료율이 장기적으로 입력률보다 낮으면 토픽은 장애를 늦출 뿐 해결하지 못한다.

되돌릴 때는 신규 작업을 동기 경로로 보내고 기존 결과 토픽과 RESULT_PENDING을 끝까지 drain한다. 진행 중 작업의 처리 경로를 기록하여 두 경로가 결과를 중복 확정하지 않게 한다. 장애 주입 검증 전에는 채택 상태로 바꾸지 않는다.

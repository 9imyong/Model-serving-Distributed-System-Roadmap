# 🚀 AI Model Serving & Distributed System Study

> 분산 환경에서 안전한 AI 모델 서빙 시스템을 설계하기 위한 구조적 학습 기록

이 저장소는 단순 이론 정리가 아니라,  
Kafka 기반 OCR 모델 서빙 시스템을 실제로 설계·운영하며 정리한 **분산 시스템 신뢰성 설계 학습 기록**입니다.

---

# 🎯 학습 목적

```
- 분산 환경에서 데이터 정합성을 어떻게 유지할 것인가?
- 이벤트 기반 구조에서 중복·재시도·장애 상황을 어떻게 제어할 것인가?
- AI 모델 서빙 서버의 성능과 안정성을 동시에 설계할 수 있는가?
- 장애 예측·대응·복구까지 포함한 신뢰성 중심 아키텍처를 설계할 수 있는가?
```

---

# 🧭 전체 학습 로드맵

```
Level 0 : 백엔드 기본기
Level 1 : 데이터 정합성 전략
Level 2 : 분산 시스템 이론
Level 3 : Event Driven Architecture (Kafka)
Level 4 : Reliability Engineering (SRE 관점)
Level 5 : AI Model Serving 안정화 전략
Level 6 : 아키텍처 설계 판단 능력
```


---

## 🟡 1. 데이터 정합성 전략

```
- Strong Consistency vs Eventual Consistency
- Idempotency Key 설계
- Retry 전략
- 상태 전이 안전 설계 (State Transition Safety)
- Conditional Update (CAS)
- Optimistic / Pessimistic Lock
```

**목표:**  
중복 실행, 중복 저장, 상태 꼬임 방지

---

## 🔵 2. Kafka 기반 Event Driven Architecture

```
- Kafka Message Semantics
- At Least Once vs Exactly Once
- Offset Commit 전략
- Consumer Group & Partition 구조
- DLQ & Retry 패턴
- Async Writer 구조 설계
```

**목표:**  
이벤트 기반 구조에서 안전한 메시지 처리 흐름 설계

---

## 🟣 3. Reliability Engineering (SRE 관점)

```
- Alert 설계
- Composite Alarm 전략
- Kafka Lag 기반 장애 예측
- GPU 병목 감지
- Auto Recovery 설계
- Failover 전략
- Runbook 작성
```

**목표:**  
장애가 발생해도 무너지지 않는 시스템 설계

---

## 🔴 4. AI Model Serving 안정화 전략
```
- GPU 직렬화 (Concurrency Control)
- Micro-batching 전략
- Shape Bucket 정렬
- Latency vs Throughput Tradeoff
- Worker 수 vs GPU 수 관계
- 모델 Warmup 전략
- 메모리 관리 전략
```
**목표:**  
성능과 안정성을 동시에 확보하는 AI 서빙 구조 설계

---

# 🛠 실전 적용 예시 (OCR 모델 서빙 시스템)

상태 전이 보호를 위한 조건부 업데이트 예시:

```sql
UPDATE jobs
SET status='RUNNING'
WHERE job_id=? AND status='REQUESTED';
```

영향받은 행이 1개일 때만 작업 획득에 성공한 것으로 처리합니다. 이 조건부 갱신만으로 장애 후 복구와 결과 저장의 멱등성까지 보장되지는 않으므로 lease·attempt token·결과 유일성 제약을 함께 설계합니다.

📈 현재 학습 초점
```
분산 환경에서의 상태 전이 안전성
Kafka 기반 메시지 처리 안정성
Strong vs Idempotency 적용 판단
장애 예측 및 자동 대응 전략
AI 모델 서빙 병목 구조 분석
```
🏁 최종 지향점

AI Model Serving Architect

Distributed Backend Engineer

Reliability 중심 설계 엔지니어

📌 방향성

이 문서는 단순한 기술 요약이 아니라,
구조를 설계할 수 있는 엔지니어가 되기 위한 학습 기록입니다.

폴더별 깃 커밋

docs: add 99_architecture_decisions
docs: add 04_ai_model_serving
docs: add 03_reliability_engineering
docs: add 02_messaging_eda
docs: add 01_data_consistency
docs: add roadmap.md

## 문서 읽는 순서와 범위

아래 문서의 OCR 구조, 수치, SQL은 학습용 설계 예시입니다. 이 저장소에서 실제 구현이나 운영 성능을 검증했다는 의미는 아닙니다. SQL은 PostgreSQL과 named parameter 표기를 가정하며 그대로 실행하는 migration은 아닙니다. ADR은 검증 전 제안으로 기록합니다.

| 단계 | 문서 |
|---|---|
| 1. 정합성 | [Strong vs Eventual](01_data_consistency/strong-vs-eventual.md) · [멱등성](01_data_consistency/idempotency.md) · [상태 전이](01_data_consistency/state-transition-safety.md) · [OCR 적용](01_data_consistency/applied-to-ocr.md) |
| 2. 메시징 | [Kafka 의미론](02_messaging_eda/kafka-semantics.md) · [Offset](02_messaging_eda/offset-commit-strategy.md) · [Retry/DLQ](02_messaging_eda/dlq-retry-pattern.md) · [OCR 적용](02_messaging_eda/applied-to-ocr.md) |
| 3. 신뢰성 | [알람](03_reliability_engineering/alert-design.md) · [복합 알람](03_reliability_engineering/composite-alarm.md) · [복구](03_reliability_engineering/recovery-strategy.md) · [OCR 적용](03_reliability_engineering/applied-to-ocr.md) |
| 4. 모델 서빙 | [GPU 직렬화](04_ai_model_serving/gpu-serialization.md) · [Micro-batching](04_ai_model_serving/micro-batching.md) · [Worker/GPU](04_ai_model_serving/worker-gpu-scaling.md) · [성능 비교](04_ai_model_serving/performance-tradeoff.md) |
| 5. 설계 판단 | [정합성 ADR](99_architecture_decisions/strong-vs-idempotent-decision.md) · [Async Writer ADR](99_architecture_decisions/async-writer-decision.md) · [Scaling ADR](99_architecture_decisions/scaling-strategy.md) |

[실습과 완료 기준은 roadmap.md](roadmap.md)에서 확인합니다. 먼저 동기 결과 저장 경로를 이해한 뒤 Async Writer 확장안을 읽는 순서를 권장합니다.

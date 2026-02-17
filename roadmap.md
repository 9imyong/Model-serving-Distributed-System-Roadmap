
---

# 🟢 Level 0 — 백엔드 기본기

- [x] 트랜잭션 이해
- [x] Isolation Level 이해
- [x] Unique Key 설계
- [x] 상태 전이 모델링

---

# 🟡 Level 1 — 데이터 정합성 전략

- [ ] Strong vs Eventual 차이 명확히 설명 가능
- [ ] Idempotency 전략 설계 가능
- [ ] 조건부 업데이트(CAS) 구현 가능
- [ ] 상태 전이 안전 설계 가능

📌 적용 목표:
OCR job 상태 전이 보호

---

# 🟠 Level 2 — 분산 시스템 이론

- [ ] CAP Theorem 이해
- [ ] At Least Once vs Exactly Once 차이 설명 가능
- [ ] Outbox Pattern 이해
- [ ] Saga Pattern 이해

📌 적용 목표:
Kafka 기반 구조에서 정합성 판단 가능

---

# 🔵 Level 3 — Event Driven Architecture

- [ ] Offset Commit 전략 설계
- [ ] Consumer 재시작 시 동작 이해
- [ ] DLQ 설계 가능
- [ ] Batch vs Stream 전략 구분

📌 적용 목표:
OCR Consumer 안정성 강화

---

# 🟣 Level 4 — Reliability Engineering

- [ ] Alert 설계 가능
- [ ] Composite Alarm 설계 가능
- [ ] Lag 기반 장애 예측 가능
- [ ] Runbook 작성 가능

📌 적용 목표:
장애 자동 판단 구조 설계

---

# 🔴 Level 5 — AI Model Serving 최적화

- [ ] GPU 직렬화 이해
- [ ] Micro-batching 적용 가능
- [ ] Worker vs GPU 수 관계 설명 가능
- [ ] Latency vs Throughput Tradeoff 설명 가능

📌 적용 목표:
AI 서빙 성능 안정화

---

# ⚫ Level 6 — 아키텍처 설계 판단

- [ ] Strong을 써야 할 위치 판단 가능
- [ ] Idempotent로 충분한 영역 구분 가능
- [ ] 동기 → 비동기 전환 시점 판단 가능
- [ ] 비용 vs 안정성 Tradeoff 설명 가능

📌 적용 목표:
구조 설계 의사결정 가능

---

# 📈 현재 진행 상황

현재 집중 영역:

- 데이터 정합성 전략
- Kafka Offset 안정성
- 상태 전이 보호 설계
- 장애 예측 구조 정리

---

# 🔄 운영 원칙

- 이 문서는 학습 진행에 따라 지속적으로 업데이트한다.
- 각 단계는 OCR 모델 서빙 시스템에 실제 적용하며 보강한다.
- 단순 개념 암기가 아닌, 설계 판단 능력 확보를 목표로 한다.

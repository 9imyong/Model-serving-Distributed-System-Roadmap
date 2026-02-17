# Kafka Message Semantics

Kafka는 기본적으로 At Least Once 처리를 제공한다.

---

## 1. At Least Once

- 메시지가 최소 1번 이상 처리됨
- 중복 가능성 존재

---

## 2. Exactly Once

- 정확히 한 번만 처리
- 구현 난이도 높음

---

## 3. 실무 판단

- 대부분 시스템은 At Least Once + Idempotent 처리
- Exactly Once는 비용이 높음

---

## 4. OCR 적용 고민

- Consumer 재시작 시 중복 처리 가능성 존재
- 따라서 멱등성 설계 필수
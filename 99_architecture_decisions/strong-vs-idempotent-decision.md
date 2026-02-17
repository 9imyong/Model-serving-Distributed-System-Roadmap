# Strong vs Idempotent 설계 판단 기록

## 문제

OCR 모델 서빙 시스템에서
모든 상태 전이를 Strong하게 보장해야 하는가?

---

## 선택지

1. Strong Consistency 전면 적용
2. Idempotent + 조건부 업데이트

---

## 판단

- 결제 시스템이 아니므로 완전 Strong은 불필요
- 상태 전이는 조건부 업데이트로 보호
- 결과 저장은 멱등 처리

---

## 결론

Strong은 최소화하고,
Idempotent + 안전한 상태 전이 전략 채택

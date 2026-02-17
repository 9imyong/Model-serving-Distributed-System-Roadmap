# GPU 직렬화 전략

## 1. 문제

GPU는 동시 추론 요청 시 메모리 충돌 또는 OOM 발생 가능

---

## 2. 해결 전략

- Semaphore 기반 직렬화
- Worker 수 제한
- Micro-batching 적용

---

## 3. Tradeoff

- 직렬화 → 안정성 증가
- 하지만 → Latency 증가 가능

---

## 4. OCR 적용

- GPU 당 1 inference 보장
- Batch 모드는 운영 전환 전략으로 활용

# Idempotency 전략

## 1. 문제 상황

- API 재시도
- Kafka Consumer 재시작
- 네트워크 단절 후 재요청

동일 요청이 여러 번 실행될 수 있다.

---

## 2. 단순 처리 시 문제점

- 중복 데이터 저장
- 상태 전이 꼬임
- 비용 증가

---

## 3. 해결 전략

- Idempotency Key 사용
- DB Unique Key 활용
- 조건부 업데이트 (CAS)

예시:

```sql
UPDATE jobs
SET status='RUNNING'
WHERE job_id=? AND status='REQUESTED';
```

## 4. OCR 시스템 적용

job_id를 Primary Key로 사용

상태 전이 시 조건부 업데이트 적용

Offset Commit은 처리 완료 후 수행


---

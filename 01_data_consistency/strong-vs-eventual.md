# Strong Consistency vs Eventual Consistency

## 1. 문제 정의

분산 환경에서는 모든 노드가 동일한 최신 상태를 즉시 공유하기 어렵다.

이때 선택해야 하는 전략이:
- Strong Consistency
- Eventual Consistency

---

## 2. Strong Consistency

모든 읽기 요청이 항상 최신 상태를 보장한다.

### 장점
- 데이터 무결성 보장
- 상태 꼬임 방지

### 단점
- 성능 저하 가능성
- 확장성 제한

### 사용 사례
- 결제
- 계좌 이체
- 재고 차감

---

## 3. Eventual Consistency

일시적으로 데이터 불일치가 발생할 수 있으나, 시간이 지나면 일관성을 회복한다.

### 장점
- 높은 확장성
- 성능 유리

### 단점
- 순간적 데이터 불일치 가능

### 사용 사례
- 로그 적재
- 알림
- OCR 결과 저장

---

## 4. OCR 시스템 적용 고민

- 상태 전이는 Strong에 가깝게 설계
- 결과 저장은 Idempotent + Eventually Consistent 방식 고려

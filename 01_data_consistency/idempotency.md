# Idempotency 전략

## 1. 재시도는 정상 경로다

API 응답이 유실되거나, 결과 저장 직후 Worker가 종료되면 호출자는 성공 여부를 모른 채 재시도한다. 멱등성의 목표는 실행 횟수를 반드시 한 번으로 제한하는 것이 아니라 **동일 요청의 비즈니스 효과가 중복되지 않게 하는 것**이다. 추론 연산은 반복되어 GPU 비용이 발생할 수 있다.

## 2. 키 설계

| 키 | 용도 | 주의점 |
|---|---|---|
| `(tenant_id, idempotency_key)` | API 재요청 식별 | 다른 사용자의 요청과 충돌 방지 |
| `job_id` | 하나의 논리 작업 식별 | 재전송 시 그대로 유지 |
| `event_id` | 전달 이벤트 중복 식별 | 논리 작업과 전달 횟수를 구분 |
| `(job_id, model_version)` | 결과 유일성 | 작업의 모델 버전을 접수 시 고정 |

같은 idempotency key에 다른 입력이 오면 기존 결과를 무조건 반환하지 않는다. 입력 URI의 불변 버전 또는 콘텐츠 해시와 옵션을 정규화한 `request_hash`를 비교하고, 다르면 충돌 응답을 반환한다. 키 보존 기간은 클라이언트 재시도와 메시지 재처리 기간을 포함하도록 정한다.

## 3. 원자적 보호

`SELECT`로 존재 여부를 확인한 다음 `INSERT`하면 두 요청이 모두 확인을 통과할 수 있다. DB의 UNIQUE 제약을 최종 방어선으로 두고, 충돌하면 기존 행과 요청 해시를 확인한다.

```sql
-- PostgreSQL 예시. 실제 migration은 별도로 작성한다.
CREATE UNIQUE INDEX jobs_request_key
ON jobs (tenant_id, idempotency_key);

UPDATE jobs
SET status = 'RUNNING', attempt = attempt + 1
WHERE job_id = :job_id AND status = 'REQUESTED'
RETURNING attempt;
```

반환 행이 1개일 때만 소유권 획득에 성공했다. 0개면 최신 상태를 읽어 완료·진행 중·실패를 구분한다. 진행 중인 작업을 단순히 성공 처리하고 메시지를 버리면, 기존 Worker 사망 시 작업이 고립될 수 있다. [lease 복구](state-transition-safety.md)가 함께 필요하다.

## 4. 결과 확정

현재 attempt의 소유권을 확인한 트랜잭션 안에서 결과 유일성을 검사하고 결과 저장과 성공 전이를 함께 확정한다. 오래된 attempt가 새 결과를 덮어쓰지 못하게 한다. 외부 파일은 attempt별 불변 경로에 먼저 저장하고 DB에는 선택된 결과 포인터만 기록한다. 참조되지 않는 파일은 별도 정리한다.

`ON CONFLICT DO NOTHING`만으로 서로 다른 결과가 동일함을 보장하지 않는다. 충돌 시 모델 버전·입력 해시 등 결과의 정체성을 확인해야 한다.

## 5. 확인 실습

동일 키로 10개 요청을 동시에 보내 논리 작업이 하나인지 확인한다. 결과 저장 후 offset commit 전에 종료하고 재시작해 결과 행이 늘지 않는지 확인한다. 다른 입력으로 같은 키를 보내 충돌이 검출되는지도 확인한다.

다음: [상태 전이 안전성](state-transition-safety.md)

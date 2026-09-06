# Micro-batching 전략

## 1. 여러 요청을 하나의 실행으로 묶는다

Micro-batching은 짧게 모은 독립 요청을 batch로 실행하는 방식이다. batch가 커지면 GPU 활용과 처리량이 좋아질 수 있지만, 모으는 대기 시간과 메모리가 늘어난다. Triton dynamic batching은 stateless 모델에서 batch 크기와 대기 지연을 설정하는 기능을 제공한다. [NVIDIA Triton Batchers](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/batcher.html)

## 2. Scheduler의 기준

설명용 정책은 `최대 8개 또는 첫 요청 대기 5ms 도달 시 실행`이다. 운영 권장 수치가 아니며 실제 모델이 해당 batch 차원을 지원해야 한다.

```text
도착 요청을 호환 bucket에 추가
→ 최대 개수 또는 메모리 예산에 도달하면 실행
→ 가장 오래된 요청의 대기 한도에 도달해도 실행
→ 결과를 원래 job_id 순서에 매핑
```

낮은 트래픽에서는 batch가 채워지지 않으므로 타이머가 필요하다. 요청별 deadline이 더 짧으면 그 예산을 우선한다. 입력별 결과 매핑이 틀리면 성능보다 심각한 정합성 문제가 된다.

## 3. OCR Shape Bucket

이미지 높이·너비나 인식 crop 폭이 크게 다르면 가장 큰 shape로 padding하며 낭비가 생긴다. 비슷한 크기를 같은 bucket으로 모으고 총 픽셀 수에도 상한을 둔다. 모델·backend가 지원하는 입력 shape 범위에서만 묶는다.

bucket이 많아지면 각각의 도착률이 낮아져 대기가 늘어난다. 드문 큰 입력도 대기 상한 이후 실행되도록 하여 starvation을 막는다. detection과 recognition은 입력 분포와 batch 정책이 다를 수 있다.

## 4. 실패 처리

batch OOM이면 해당 시도의 결과를 성공으로 확정하지 않는다. 허용된 횟수 안에서 batch 분할을 시도하고, 단일 입력도 실패하면 입력 정책에 따라 격리한다. 이미 완료한 다른 job까지 재실행하더라도 결과 확정은 멱등하게 처리한다.

## 5. 비교 실험

batch 크기 1·2·4·8과 대기 한도를 바꿔 처리량, end-to-end p95/p99, peak memory, 실제 batch 크기 분포를 기록한다. 작은 입력만 있는 실험과 큰 입력이 섞인 실험을 구분한다. 처리량 이득이 있어도 deadline을 넘으면 채택하지 않는다.

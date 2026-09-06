# GPU 직렬화 전략

## 1. GPU 동시 실행은 자원 예산 문제다

GPU가 동시 추론을 원천적으로 못 하는 것은 아니다. 동시 요청마다 activation·workspace가 추가되고, 프로세스마다 모델과 CUDA context가 복제되면 메모리가 부족해질 수 있다. 모델 구현에 공유 가변 상태가 있다면 thread safety도 따로 확인해야 한다.

출발점으로 GPU별 추론 실행 경로를 하나로 제한하고, 측정 결과에 따라 batch와 동시성을 늘린다. 이는 특정 환경에서의 보수적인 설계 선택이며 모든 GPU의 최적값이 1이라는 뜻은 아니다.

## 2. Semaphore의 범위

```text
CPU 전처리 → bounded queue → GPU 담당 실행기 → CPU 후처리
                              동시 실행 한도 1
```

프로세스 내부 semaphore는 다른 프로세스를 통제하지 못한다. API Worker가 4개이고 각자 semaphore 1개와 모델을 가지면 GPU에는 최대 4개 실행 경로가 생긴다. GPU별 단일 소유 프로세스나 중앙 스케줄러로 전체 동시성을 관리한다.

CUDA 연산은 비동기로 제출될 수 있으므로 CPU 함수 반환이 GPU 실행 완료와 같은지 확인한다. 엄격한 완료 경계는 backend의 동기화·event 규약에 맞춰 구현한다. 무분별한 전체 동기화는 성능을 저하시킬 수 있다.

## 3. 직렬화만으로 해결되지 않는 것

단일 요청 자체가 너무 크면 여전히 OOM이 난다. 입력 최대 크기, 페이지 분할, batch 메모리 한도가 필요하다. 큐도 무제한이면 CPU RAM과 대기 시간이 증가하므로 크기와 대기 deadline을 제한한다.

## 4. 실험 순서

1. 모델 한 벌과 대표 입력으로 warmup한다.
2. 동시성 1에서 p95, 처리량, peak memory를 측정한다.
3. 동일 입력 분포로 동시성 2 이상을 비교한다.
4. OOM·deadline 초과·처리량을 함께 보고 실행 한도를 선택한다.

선택한 값과 GPU 종류, 모델 버전, 정밀도, 입력 shape를 기록해야 재현할 수 있다.

다음: [Micro-batching](micro-batching.md), [Worker와 GPU](worker-gpu-scaling.md)

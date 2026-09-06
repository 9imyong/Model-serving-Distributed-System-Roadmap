# Worker 수 vs GPU 수 관계

## 1. Worker라는 이름을 분리한다

API 프로세스, Kafka Consumer, CPU 전처리 실행기, GPU 모델 인스턴스는 서로 다른 역할이다. API Worker 수를 늘리는 설정이 모델 복제 수까지 늘리는지 확인해야 한다.

| 역할 | 주된 제한 요소 | 확장 시 확인 |
|---|---|---|
| API | 연결·CPU·접수 DB | 유입 제한, DB 여유 |
| Consumer | 파티션·poll·내부 큐 | 할당과 backpressure |
| 전처리 | CPU·다운로드 대역폭 | GPU 공급 부족 여부 |
| 추론 실행기 | GPU 메모리·연산 | 모델 복제와 batch |
| Writer | DB 처리량·잠금 | 연결 수와 쓰기 병목 |

## 2. 초기 배치 예시

GPU 2장에 각 1개의 추론 소유 프로세스를 두고 CPU 전처리를 별도 풀로 운영하는 구성을 실험할 수 있다. 이때 Consumer 수와 GPU 수가 반드시 같을 필요는 없지만 큐를 통한 전달과 완료 추적이 필요하다.

일반적인 Consumer Group에서 파티션이 2개면 Consumer를 4개 띄워도 동시에 파티션을 소유하는 Consumer는 최대 2개다. 추론을 별도 풀로 전달하는 구조에서는 GPU 병렬성이 이 숫자와 일대일로 같지는 않다. 파티션 수와 GPU 실행 슬롯을 구분한다.

## 3. 메모리 예산

```text
예상 GPU 메모리
≈ 모델 가중치 복제 + 실행 중 activation/workspace
  + CUDA context/allocator 오버헤드 + 여유 공간
```

activation은 batch와 shape에 따라 변한다. 위 식은 산정 틀이며 정확한 선형 공식이 아니다. 측정된 최악 입력 peak를 기준으로 배치 수를 정한다.

## 4. 늘릴 때와 줄일 때

GPU 공급이 끊기면 전처리·다운로드를 조사한다. GPU가 포화이고 큐 나이가 증가하면 batch 효율 또는 GPU 증설을 검토한다. DB가 포화이면 추론 Worker 증설을 멈춘다.

증설에는 이미지 준비·모델 다운로드·warmup 시간이 포함된다. 축소할 때는 새 획득을 멈추고 drain한 뒤 종료한다. 갑작스러운 종료도 lease와 멱등성이 처리해야 한다.

실습: GPU 수를 고정하고 API/CPU Worker만 증가시켜 어느 지점부터 처리량이 늘지 않는지 기록한다.

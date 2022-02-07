---
title: "pytorch_10. pytorch troubleshooting"
excerpt: "OOM 예시와 해결방법 / GPU 메모리 디버깅 툴"

categories:
  - boostcamp PyTorch Basic
tags:
  - [boostcamp, boostcamp PyTorch Basic]

toc: true
toc_sticky: true

date: 2022-01-28 10:14
last_modified_at: 2022-01-28 10:14
---

# PyTorch Troubleshooting

## 강의 소개
* GPU OOM 예시와 해결법
* GPU 메모리 디버깅 툴
* 초보자가 쉽게 하는 실수들

## OOM: Out of Memory
* 왜, 어디서 발생했는지 알기 어려움.
* error backtracking이 너무 깊은 곳까지 도달.
* 메모리의 이전 상황 파악이 어려움 - 메모리의 지속적인 모니터링이 필요함.

### 해결
* 배치 크기 축소 ➡ GPU clean ➡ run: 제일 쉽다. 해결된다면 일단 코드는 정상적으로 작성됨.
  ```python
  # 배치 크기 테스트
  try:
      train(batch_size)
  except RuntimeError: 
      for _ in range(batch_size):
          train(1)  # batch size 1로 계속 돌려본다.
  # backward 위해서 레이어마다 activation을 유지하므로
  # 데이터 샘플 하나씩 넣어보며 점진적으로 activation 양을 증가시켜서
  # OOM까지 허용되는 batch size를 찾는 방식인 것 같다.
  ```
* GPUUtil 사용: GPU 상태를 모니터링
  * nvidia-smi와 유사
  * iteration마다 메모리 점유를 확인
    * iteration마다 지속적으로 메모리 점유가 늘어난다? = 메모리 누수 중
* `torch.cuda.empty_cache()`: 사용되지 않는 GPU 상의 cache 정리 (가비지 컬렉터 실행)
  * 가용 메모리 확보
  * 파이썬의 `del`과 구분 - `del`은 메모리와 이름의 연결을 끊는 역할. 가비지 컬렉터가 작동하기 전까지는 메모리를 점유.
  * 학습 전에 한번 실행하는걸 권장.
* tensor 변수는 GPU 메모리 사용
  ```python
  total_loss = 0.
  for X, y in train_data:
      opt.zero_grad()
      pred = model(X)
      loss = loss_fn(pred, y)
      loss.backward()
      opt.step()
      total_loss += loss  # 텐서를 누적시킴 - 그래디언트 트래킹이 되면서 메모리를 점유
      # total_loss도 backward가 가능하므로 각 미니배치의 loss 텐서가 계속 메모리를 점유한다.
  ```
  * 1-d 텐서는 파이썬 객체로 변환: `loss.item()`, `float(loss)`, $\cdots$
  * `del` 사용
    ```python
    for X, y in train_data:
        ...
    
    # for loop를 나와도 변수 X, y는 메모리 점유 중. del로 참조 끊고 가비지 컬렉팅.
    print(X)
    ```
* `torch.no_grad()`: 명시적으로 backward 패스를 막음.
  ```python
  def test(dataloader, model, loss_fn):
    ...
    model.eval()  # 모델을 evaluation 모드로 설정. 
    with torch.no_grad():  # 그래디언트 계산을 하지 않도록 설정 - 메모리 사용량 감소
        for X, y in dataloader:
            X, y = X.to(device), y.to(device)
            pred = model(X)
            ...
  ```
  * 이 컨텍스트 하에서 발생한 모든 연산은 `requires_grad=False` 처리된다.
  * backward를 위한 메모리 점유가 없음. 주로 inference 시점에서 사용.
* FP16 연산 - mixed precision

### 유사한 에러
* CUDNN_STATUS_NOT_INIT: cuda 등의 설치, 설정 관련.
* device-side-assert: OOM의 일종으로 생각할 수 있음. 
* [에러정리(brstar96님 블로그)](https://brstar96.github.io/shoveling/device_error_summary/){: .btn .btn--info}

## 기타
* colab에서 너무 큰 네트워크 지양 (특히 LSTM)
* CNN은 입력 shape 실수가 생기기 쉬우니 `torchsummary` 등을 이용하자.
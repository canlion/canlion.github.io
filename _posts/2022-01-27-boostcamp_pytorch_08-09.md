---
title: "pytorch_08-19. multi-gpu / hp tuning"
excerpt: "model parallel & data parallel / 파라미터 튜닝을 위한 Ray Tune"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-27 10:49
last_modified_at: 2021-01-27 10:49
---

# Multi-GPU 학습

GPU가 발전하면서 거대한 데이터, 거대한 네트워크가 트렌드. 

## 개념 정리
* single vs multi
* gpu vs node
  * node: 한 대의 시스템
* single node single gpu: 한 대의 시스템의 한 개 gpu
* single node multi gpu: 한 대의 시스템의 다수 gpu
* multi node multi gpu: 여러 대의 시스템의 다수 gpu

## multi gpu

### Model Parallel
다중 gpu에 모델을 나누는 방식. 과거부터 사용되던 방식(ex-alexnet)이지만 모델의 병목, 파이프라인 어려움 등으로 고난이도.

[PyTorch:SINGLE-MACHINE MODEL PARALLEL BEST PRACTICES](https://pytorch.org/tutorials/intermediate/model_parallel_tutorial.html){: .btn .btn--info}

![model_parallel](/assets/images/post/220127/boostcamp_pytorch_08-09/model_parallel.png){: .align-center}

### Data Parallel
데이터를 gpu에 나눠서 할당, 결과의 평균을 취한다.

#### 일반적인 Data Parallel
1. 모든 gpu에 동일한 모델 전송.
2. 한 개의 미니 배치 데이터를 gpu 수만큼 쪼개어 각 gpu에 할당.
3. 연산 수행. 
4. 각 gpu의 모델 output을 메인 gpu로 전송.
5. 메인 gpu에서 각 gpu로부터 전송된 output에 대응하는 loss의 평균으로 그래디언트 계산.
6. 메인 gpu에서 각 gpu로 loss 그래디언트 전송.
7. 각 gpu에서 backward 수행, 그래디언트 계산
8. 각 gpu에서 메인 gpu로 그래디언트 전송
9. 메인 gpu에서 그래디언트 평균으로 가중치 갱신
10. 1~9 반복

![data_parallel](/assets/images/post/220127/boostcamp_pytorch_08-09/data_parallel.png){: .align-center}

메인 gpu에서 loss와 loss 그래디언트를 계산하고 가중치 업데이트를 담당하므로 다른 gpu보다 메모리 사용량이 많을 수 있음(gpu 사용 불균형 발생). 모든 gpu에서 동일한 크기의 데이터를 처리해야하므로 메인 gpu 맞춰 미니 배치 크기를 줄여야할 수도 있음. + GIL

```python
parallel_model = torch.nn.DataParallel(model)
...
pred = parallel_model(X)
loss = loss_func(pred, y)
loss.mean().backward()  # loss 평균내어 그래디언트 계산
optimizer.step()
```

#### DistributedDataParallel
위의 일반적인 data parallel은 한 프로세스가 메인 gpu에 붙어서 모든 작업을 담당. distributed data parallel은 각 gpu에 프로세스가 붙어서 주어진 데이터에 forward, backward 진행. 모든 gpu에서 그래디언트가 준비된다면 allreduce 연산을 통해 그래디언트 평균내어 각 gpu에서 모델 업데이트 수행.

![dist_data_parallel](/assets/images/post/220127/boostcamp_pytorch_08-09/dist_data_parallel.png){: .align-center}

```python
sampler = torch.utils.data.distributed.DistributedSampler(data)
trainloader = torch.utils.data.DataLoader(
    data,
    batch_size=20,
    shuffle=False,
    pin_memory=True,  # gpu로 데이터를 보내는 과정을 간소화하여 빠르게
    num_workers=(num_gpu*4)  # 경험적인 부분인 듯.
)
...
```

[PyTorch:GETTING STARTED WITH DISTRIBUTED DATA PARALLEL](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html?highlight=distributed){: .btn .btn--info}

<br>

# Hyperparameter Tuning

**하이퍼파라미터**: 머신러닝 모델 학습에서 사람이 선택해야하는 파라미터 (lr, lr decay, ...)

* 성능 향상을 위한 시도
  * 모델 변경
    * 이미 여러 모델이 존재하고 어떤 모델이 좋은지도 다 안다.
  * 🌟 데이터 검수, 추가
  * 하이퍼파라미터 튜닝
    * 이익이 그렇게 크지는 않다.
    * 조금이라도 성능 향상을 위해 정말 처절하게 마지막까지 매달리는 방법
  * 기타$\cdots$

## hyperparameter tuning
모델 스스로 학습하지 않는 값을 어떻게 설정해야하는가? - 학습률, 모델 구조, 모델 크기, optimizer, ...

이전에는 네트워크 성능을 재현하는 일에 하이퍼파라미터가 굉장히 중요했지만 데이터의 양으로 밀어붙이며 점차 그 중요성은 덜해지는 추세. 데이터가 많아지면서 하이퍼파라미터 튜닝도 만만치 않은 작업이 되어버림. 하이퍼파라미터에 따라 성능 차이가 생길 수 있으므로 학습 초기에 적절한 하이퍼파라미터값을 찾기위한 노력이 될수도 있고, 최후에 약간의 성능이라도 쥐어짜기위한 도전이 될 수도 있다.

이전에는 random search, grid search를 함께 사용하였으나 최근에는 베이지안 기반 기법들이 대세. (BOHB, 2018, [paper](https://ml.informatik.uni-freiburg.de/wp-content/uploads/papers/18-ICML-BOHB.pdf))

### random + grid

#### random search
* 무작위 지점 학습 수행
* 성능이 잘 나오는 대략적인 지점을 탐색

![random](/assets/images/post/220127/boostcamp_pytorch_08-09/random_search.png){: .align-center}

#### grid search
* 보통 일정 log값 간격으로 몇개의 하이퍼파라미터 포인트를 만들고 학습 수행
* random search 후 대략적인 지점에서 디테일하게 하이퍼 파라미터 튜닝

![grid](/assets/images/post/220127/boostcamp_pytorch_08-09/grid_search.png){: .align-center}

## Ray

* 하이퍼파라미터 튜닝 툴. multi node multi processing 지원
* ML/DL 병렬처리 위한 모듈
  * 분산병렬 ML/DL 모듈의 표준
* 하이퍼파라미터 서치를 위한 모듈 제공

[PyTorch:HYPERPARAMETER TUNING WITH RAY TUNE](https://pytorch.org/tutorials/beginner/hyperparameter_tuning_tutorial.html){: .btn .btn--info}

---
title: "pytorch_04-05. PyTorch AutoGrad & Optimizer / dataset"
excerpt: "Module, Parameter, Optimizer / Dataset"

categories:
  - boostcamp PyTorch Basic
tags:
  - [boostcamp, boostcamp PyTorch Basic]

toc: true
toc_sticky: true

date: 2022-01-25 10:36
last_modified_at: 2022-01-25 10:36
---
<br>

## Module & Parameter & Backward

### nn.Module
네트워크는 레이어(블럭)의 연속. 여러 레이어를 조합해 또 다른 레이어를 정의한다.

`nn.Module`은 레이어의 base class. input, output, forwad, backward(& parameter)를 정의해야한다.

### nn.Parameter
`Tensor`를 상속한 객체. `nn.Module`의 속성이 되면 `requires_grad = True`가 되어 AutoGrad의 대상이 된다. 대부분의 레이어는 이미 구현되어있어 `nn.Parameter`를 직접 사용할 일은 거의 없다.

`nn.Parameter` 대신 `Tensor`를 사용하면 `nn.Module`의 파라미터로 인식되지않고 학습의 대상도 아니다.

### backward
레이어의 파라미터들의 그래디언트 계산. 모듈 단계에서 직접 지정 가능.
```python
for x, y in data_iterator:
    ...
    optimizer.zero_grad()  # 레이어마다 그래디언트를 보관하는데 그것을 초기화함.
    y_hat = model(inputs)
    loss = loss_func(y, y_hat)
    loss.backward()  # loss 미분하여 그래디언트 계산
    optimizer.step()  # 파라미터 갱신
```

<br>

## Dataset
**모델에 데이터를 어떻게 넣을까**

### 구성요소
* `Dataset`: 데이터들을 로드하고 순회하며 하나의 데이터를 반환.
* `Transforms`: 하나의 데이터를 처리 - 데이터를 텐서로 변환, augmentation 적용.
* `DataLoader`: batch 처리, shuffle 등 개별 데이터를 묶어 모델에 피딩.
* `Dataset` --[`Transforms`]--> `DataLoader` --> `Model`

### Dataset class

데이터 입력 형태를 정의하는 클래스. 데이터 형태에 따라서 다르게 정의됨.

`__getitem__(self, idx)`메소드를 통해 데이터 순회시에 개별 데이터를 어떻게 반환할지 정의한다. `DataLoader`에서 호출된다.

데이터에 대한 모든 처리를 `Dataset`에서 처리할 필요 없이 필요한 시점에서 처리하면 된다.

데이터에 대한 표준화된 처리를 정의하므로 계속 재사용하거나 타인이 사용할 수 있어 중요한 요소.

### DataLoader class

데이터의 배치 처리, 셔플을 수행하는 클래스.

모델 피딩 직전의 데이터 변환 수행. 주로 데이터의 Tensor화, 배치 처리가 메인. 병렬적인 데이터 전처리 코드가 핵심.

* 읽어볼 파라미터: `sampler`, `collate_fn`, ...
---
title: "pytorch_01-03. PyTorch Introduction / Basics / Project"
excerpt: "딥러닝 프레임워크 종류 / 파이토치 작동 구조 / 기초"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-24 10:10
last_modified_at: 2022-01-24 10:10
---
<br>

# 딥러닝 프레임워크

## 종류
1. 텐서플로우 - 구글
    * Define and run
    * production & scalability부분에서 강점
2. 파이토치 - 페이스북(메타)
    * Dynamic Computation Graph - 실행 시점에서 그래프 작성
    * 쉽고 빠르게 아이디어의 구현 가능
3. 기타

## computational graph
연산 과정을 그래프로 표현

* **Define and Run**
  * 그래프를 먼저 정의, 실행 시점에 데이터를 흘려넣어줌
* **Define by Run (Dynamic Computational Graph)**
  * 실행을 하면서 그래프를 생성하는 방식

## 파이토치

**Numpy + AutoGrad + Functions**

* numpy: numpy 구조를 가지는 tensor 객체로 array 표현
* **auto-grad**: 자동 미분 지원
* functions: 다양한 DL 관련 함수, 모델 지원 (layers, dataset, multi-gpu, ...)

<br>

# 파이토치

**Numpy + AutoGrad**

## Tensor
* 다차원 array를 표현하는 pytorch class
* 사실상 numpy ndarray, TF의 Tensor와 동일
* Tensor를 생성하는 함수도 numpy와 거의 동일

### view와 reshape의 차이? - 일단 view를 써라.

### 행렬곱 - mm과 matmul
* mm: 차원 broadcasting 미지원
* matmul: 차원 broadcasting 지원 - 편하지만 실수할 수 있다. 

내적은 dot. **dot으로는 행렬곱이 불가!**

## device
연산 장치 (CPU | GPU) - `tensor.device`

메모만 간단히 하고 코드 공부는 직접
{: .notice}

<br>

# 파이토치 프로젝트 구조 이해하기

공유와 배포, 개발 용이성, 유지보수, 타인에 의한 재현 가능성을 고려하여 OOP와 모듈을 통해 프로젝트를 구성해야함.

다양한 프로젝트 템플릿이 존재하므로 수정하여 사용 가능. 

* 추천 레포지터리
  * [link: victoresque-pytorch-template](https://github.com/victoresque/pytorch-template){: .btn .btn--info}
  * [link: FrancescoSaverioZuppichini-PyTorch-Deep-Learning-Template](https://github.com/FrancescoSaverioZuppichini/PyTorch-Deep-Learning-Template){: .btn .btn--info}
  * [link: PyTorchLightning Template](https://github.com/PyTorchLightning/deep-learning-project-template){: .btn .btn--info}
  * [link: Pytorch Lightning + NNI Boilerplate](https://github.com/davinnovation/pytorch-boilerplate){: .btn .btn--info}

* 참고
  * [link: PyTorchLightning](https://www.pytorchlightning.ai/){: .btn .btn--info}
  
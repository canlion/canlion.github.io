---
title: "AI Math 5 - 딥러닝의 학습방법"
excerpt: "비선형 모델 신경망, softmax, 활성함수, 역전파"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-19 17:45
last_modified_at: 2021-01-19 17:45
---

# 신경망

선형 모델은 단순한 데이터 해석에 작동. 분류나 복잡한 패턴의 문제는 비선형 모델인 신경망이 필요하다.

신경망은 선형모델과 비선형 함수의 결합으로 이루어진다.

## 선형모델

기호
* $\boldsymbol{\mathrm{X}}$: 데이터
* $\boldsymbol{\mathrm{W}}$: 선형변환. 벡터를 $d$ 차원에서 $p$ 차원으로 보낸다.
* $\boldsymbol{\mathrm{b}}$: 절편 행렬. 모든 행이 같다. (모든 데이터에 같은 값을 더한다.)

$$\begin{matrix}
  \begin{bmatrix}
    - & \boldsymbol{\mathrm{o}}_1 & - \\ 
    - & \boldsymbol{\mathrm{o}}_2 & - \\
    & \vdots & \\
    - & \boldsymbol{\mathrm{o}}_n & - \\
  \end{bmatrix}
  & = &
  \begin{bmatrix}
    - & \boldsymbol{\mathrm{x}}_1 & - \\ 
    - & \boldsymbol{\mathrm{x}}_2 & - \\
      & \vdots & \\
    - & \boldsymbol{\mathrm{x}}_n & - \\
  \end{bmatrix}
  &
  \begin{bmatrix}
    w_{11} & w_{12} & \cdots & w_{1p} \\
    w_{21} & w_{22} & \cdots & w_{2p} \\
    \vdots & \vdots & \ddots & \vdots \\
    w_{d1} & w_{d2} & \cdots & w_{dp} \\
  \end{bmatrix}
  &+&
  \begin{bmatrix}
    \vert & \vert & \cdots & \vert \\
    b_1 & b_2 & \cdots & b_p \\
    \vert & \vert & \cdots & \vert \\
  \end{bmatrix}
  \\
  \boldsymbol{O} & & \boldsymbol{\mathrm{X}} & \boldsymbol{\mathrm{W}} & & \boldsymbol{\mathrm{b}}
  \\
  (n\times p) & & (n\times d) & (d\times p) & & (n\times p)
\end{matrix}$$

![layer](/assets/images/post/220119/boostcamp_ai_math_5/layer_0.png)

## softmax

$$
\mathrm{softmax}(\boldsymbol{\mathrm{o}}) =
\left(
  \frac{\mathrm{exp}\left(o_1\right)}{\sum^p_{k=1}\mathrm{exp}\left(o_k\right)},
  \cdots,
  \frac{\mathrm{exp}\left(o_p\right)}{\sum^p_{k=1}\mathrm{exp}\left(o_k\right)}
\right)
$$

* 모델의 출력을 확률로 변환해주는 연산. 입력이 특정 클래스에 속할 확률로 해석하여 분류문제에 사용할 수 있다.
![linear_cls](/assets/images/post/220119/boostcamp_ai_math_5/linear_classifier.png){: .align-center}
* 코드로 구현할때는 오버플로우를 예방하기위해 $\boldsymbol{\mathrm{o}}$에서 최댓값을 빼고 계산한다.
  ```python
  # o: (n, p)
  # 각 행마다의 최댓값. 데이터 단위로 최댓값을 뺀다.
  o = o - np.max(o, axis=-1, keepdims=True)  # (n, p) - (n, 1)
  denumerator = np.exp(o)
  numerator = np.sum(denumerator, axis=-1, keepdims=True)  # (n, 1)
  softmx = denumerator / numerator  # (n, p) / (n, 1)
  ```
* softmax의 입력 중 최댓값인 원소에 대응되는 softmax의 출력이 가장 크기때문에 inference시에는 softmax를 생략하고 출력 중 최댓값인 원소에 대응되는 클래스를 선택한다.

## 신경망

신경망은 선형모델과 활성함수를 합성한 함수

### 활성함수 activation

* 비선형 함수로 선형모델의 출력물의 각 원소에 적용되는 함수.
  * 활성함수로부터 출력된 벡터를 **잠재벡터, 히든벡터, 뉴런**이라 한다.
* 선형모델을 비선형모델로 변화시킨다. 
* 입력벡터 $-$[선형 모델 + 활성함수]$\rightarrow$ **뉴런** 
* 종류
  * 시그모이드 sigmoid - $\sigma(x)=\frac{1}{1+e^{-x}}$
  * 하이퍼볼릭 탄젠트 tanh - $\mathrm{tanh}(x)=\frac{e^x-e^{-x}}{e^x+e^{-x}}$
  * ReLU - ${\mathrm{ReLU}(x)=\mathrm{max}(0, x)}$
    * 현재 가장 많이 사용

### 신경망

* 뉴런으로 이루어진 모델
* 선형모델 + 활성함수: 전통적인 신경망 "퍼셉트론"
* [선형모델 + 활성함수]의 반복: 다층 신경망 - **활성함수가 없다면 일반적인 선형모델과 다름없다.**
  * 2-layers neural network
    ![2layers](/assets/images/post/220119/boostcamp_ai_math_5/2_layers.png){: .align-center}

#### 순전파 forward propagation

$$
\begin{matrix}
  \boldsymbol{\mathrm{Z}}^{(1)} = \boldsymbol{\mathrm{XW}}^{(1)}+\boldsymbol{\mathrm{b}}^{(1)} \\
  \boldsymbol{\mathrm{H}}^{(1)} = \sigma(\boldsymbol{\mathrm{Z}}^{(1)}) \\
  \vdots \\
  \boldsymbol{\mathrm{Z}}^{(l)} = \boldsymbol{\mathrm{H^{(l-1)}}}\boldsymbol{\mathrm{W}}^{(l)}+\boldsymbol{\mathrm{b}}^{(l)} \\
  \boldsymbol{\mathrm{H}}^{(l)} = \sigma(\boldsymbol{\mathrm{Z}}^{(l)}) \\
  \vdots \\
  \boldsymbol{\mathrm{O}}^{(L)} = \boldsymbol{\mathrm{Z}}^{(L)}
\end{matrix}
$$

#### 신경망에서 층을 쌓는 이유

**univiersal approximation theorem**: 이론적으로 2층 신경망으로도 임의의 연속 함수를 근사할 수 있다. 허나 실제로 잘 작동하지는 않음. 층을 깊게 쌓을수록 원하는 함수를 표현하는데 필요한 노드, 뉴런의 수가 급격히 줄어들어 효율적인 학습이 가능하게된다. 층이 얇을수록 굉장히 넓은 신경망이 필요하다.

#### 역전파 알고리즘

경사하강법을 통해 가중치를 학습시킬때 가중치마다 그래디언트를 계산해야한다. 신경망은 층이 매우 많기때문에 미분의 연쇄법칙을 통해 손실함수부터 시작하여 그래디언트 계산하고 저층으로 전파하면서 각 층의 가중치 그래디언트를 계산한다.

연쇄법칙을 이용해 미분을 계산하려면 각 노드의 값을 보관하고 있어야하므로 역전파 알고리즘은 메모리 사용량이 많다.

**예제 - 2층 신경망**

손실함수: $\mathcal{L}$

![back_prop](/assets/images/post/220119/boostcamp_ai_math_5/backprop.png){: .align-center}

$$\begin{matrix}
\frac{\partial\mathcal{L}}{\partial{\mathrm{W}}^{(1)}_{ij}} & = & \sum_{l,r,k}
\frac{\partial\mathcal{L}}{\partial\boldsymbol{\mathrm{o}}_l}
\frac{\partial\mathcal{o}_l}{\partial{\mathrm{h}_r}}
\frac{\partial\mathcal{h}_r}{\partial{\mathrm{z}_k}}
\frac{\partial\mathcal{z}_k}{\partial{\mathrm{W}^{(1)}}_{ij}}
\\
& = & \sum_{l,r,k}
\frac{\partial\mathcal{L}}{\partial\boldsymbol{\mathrm{o}}_l}
\left(\mathrm{W}^{(2)}_{rl}\right)
\left(\sigma'(\mathrm{z}_k)\delta_{rk}\right)
\left( x_i\delta_{jk}\right)
\\
& = & \sum_{l}
\frac{\partial\mathcal{L}}{\partial\boldsymbol{\mathrm{o}}_l}
\mathrm{W}^{(2)}_{jl}
\sigma'(\mathrm{z}_j)
x_i \\
\end{matrix}$$

$\delta_{jk},\delta_{rk}$ : 연결된 노드로부터의 역전파만을 선별하기위해 $\delta$는 $\mathrm{W}^{(1)}_{ij}$와 연결된 노드일때만 1, 아니면 0. 위의 2층 신경망에서는 $j=k=r$인 경우만 $\delta=1$이다.
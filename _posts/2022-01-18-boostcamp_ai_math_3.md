---
title: "AI Math 3,4 - 경사하강법-0"
excerpt: "경사하강법"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-18 14:59
last_modified_at: 2022-01-18 14:59
---
# 경사하강법
## 미분
변수의 움직이에 따른 함수값의 변화 측정하는 방법. 최적화에 사용. 파이썬에서는 sympy 모듈을 통해 미분 계산 가능.
$$f'(x) = \lim_{h\rightarrow0}\frac{f(x+h)-f(x)}{h}$$

미분은 함수의 어떤 점에서의 기울기를 계산.

$f(x)$의 값을 증가시키고 싶다면 $x$에 $f'(x)$을 더하고 (**경사상승법**), 감소시키고 싶다면 $f'(x)$를 뺀다. (**경사하강법**)


* 경사하강법
  ```
  가중치 초기화
  그래디언트 = 그래디언트_계산(가중치)
  while 그래디언트 > threshold:  # 정확히 극값에 도달하기는 불가능.
      가중치 = 가중치 - 학습률 * 그래디언트
      그래디언트 = 그래디언트_계산(가중치)
  ```

변수가 벡터인 경우 편미분(partial differentiation) 활용. 성분이 $d$개인 벡터는 $d$번 편미분이 필요하고, 계산된 벡터는 그래디언트 벡터 $\nabla f$로 표현. 그래디언트 벡터의 각 성분은 그 성분의 축을 따라 함수의 기울기를 의미.

$$\partial_{x_i}f(\boldsymbol{\mathrm{x}})=\lim_{h\rightarrow0}\frac{f(\boldsymbol{\mathrm{x}}+h\boldsymbol{\mathrm{e}_{i}})+f(\boldsymbol{\mathrm{x}})}{h}$$

$$\nabla f=\left( \partial_{x_1}f,\partial_{x_2}f,\cdots,\partial_{x_d}f \right)$$

$h$에 편미분하려는 변수의 위치만 1인 단위벡터를 곱해 다른 변수의 개입을 차단.

* 경사하강법
  ```
  가중치 초기화
  그래디언트 = 그래디언트_계산(가중치)
  while norm(그래디언트) > threshold:  # 벡터이므로 norm 계산
      가중치 = 가중치 - 학습률 * 그래디언트
      그래디언트 = 그래디언트_계산(가중치)
  ```

## 경사하강법

복습
* 선형회귀식을 구할 때 무어-팬로즈 역행렬을 이용할 수 있다. 

### 경사하강법을 이용한 최적화 방법
* 선형회귀의 목적식: L2 norm $\lVert \boldsymbol{y} - \boldsymbol{\mathrm{X}\beta} \rVert_2$
  * 목적식을 최소화하는 $\boldsymbol{\beta}$를 구하자.
  * 여기서 L2 norm은 데이터가 $n$개 이므로 $\sqrt{\frac{1}{n}\sum_{i=1}^n(y_i-\boldsymbol{\mathrm{X}}_i\boldsymbol{\beta})^2}$
  * $\partial_{\beta_k}\lVert \boldsymbol{y} - \boldsymbol{\mathrm{X}\beta} \rVert_2=-\frac{(\boldsymbol{\mathrm{X}^\top})_k(\boldsymbol{y} - \boldsymbol{\mathrm{X}\beta})}{n\lVert \boldsymbol{y} - \boldsymbol{\mathrm{X}\beta} \rVert_2}$
    * $(\boldsymbol{\mathrm{X}^\top})_k$ : $\boldsymbol{\mathrm{X}}$의 $k$번째 열벡터
    * $\nabla_{\beta}\lVert \boldsymbol{y} - \boldsymbol{\mathrm{X}\beta} \rVert_2=-\frac{\boldsymbol{\mathrm{X}^\top}(\boldsymbol{y} - \boldsymbol{\mathrm{X}\beta})}{n\lVert \boldsymbol{y} - \boldsymbol{\mathrm{X}\beta} \rVert_2}$
* 경사하강법:
  * $\boldsymbol{\beta}^{(t+1)}=\boldsymbol{\beta}^{(t)}-\lambda\nabla_{\beta}\lVert \boldsymbol{y} - \boldsymbol{\mathrm{X}\beta} \rVert_2$, ($\lambda$: 학습률)
  * L2 norm의 제곱하여 루트를 제거하고 목적식으로 사용할 수 있다. 그래디언트를 구하는 과정이 간단해진다.
    * $\nabla_\beta\lVert\boldsymbol{y}-\boldsymbol{\mathrm{X}\beta}\rVert_2^2=-\frac{2}{n}\boldsymbol{\mathrm{X}}^\top(\boldsymbol{y}-\boldsymbol{\mathrm{X}\beta})$
    * 분모가 사라졌지만 0보다 큰 스칼라로 취급하여 제거해도 그래디언트의 방향을 바꾸지 않는다.
  * ```
    for t in range(T):  # 종료 조건: 학습 횟수
        error = y - X @ 가중치
        grad = - X.T @ error
        가중치 = 가중치 - 학습률 * grad
    ```
    * 학습 횟수가 너무 적으면 최적화를 완료하지 못한다.
    * 적절한 학습률, 학습 횟수 필요.

### 한계
* 미분 가능한 볼록 함수와 적절한 학습률, 학습 횟수에 대해서만 수렴이 보장
* 비선형 회귀 문제의 경우 목적식이 볼록 함수가 아닐 수 있어 목적식의 최저점으로의 수렴을 보장할 수 없다.
  * ![non_linear](/assets/images/post/220118/boostcamp_ai_math_3/non_linear.png)
  * **극소값이지만 목적식의 최소값은 아니다.**

### 확률적 경사 하강법, SGD
stochastic gradient descent

모든 데이터를 이용해 그래디언트를 계산하는 것이 아닌 데이터 일부를 이용해 그래디언트를 계산하는 방식.

|그래디언트 계산에 사용되는 데이터 수 | 명칭 |
|:-----------------------------:|:-----:|
|1|SGD|
|m (1<m<n)|mini-batch SGD|

$$\theta^{(t+1)}\leftarrow\theta^{(t)}-\widehat{\nabla_\theta\mathcal{L}(\theta^{(t)})}$$
* $\widehat{\nabla_\theta\mathcal{L}(\theta^{(t)})}$: 미니 배치로부터 계산한 그래디언트

볼록이 아닌 목적식도 확률적으로 최적화가 가능하다. 그래디언트를 계산할때 일반 경사하강법에 비해 데이터를 덜 사용하므로 연산량이 낮고, 확률적이긴하지만 전체 데이터를 모두 사용하여 계산한 그래디언트와 비슷한 방향을 가질 것이다. (미니 배치에 포함된 데이터 조합에 의존하겠지만)

**매번 미니 배치가 달라지므로 목적식도 매번 달라진다.**
* 미니배치가 배치의 특성을 가질 것이므로 배치로부터 계산한 그래디언트와 유사한 방향을 가진 그래디언트가 계산될 것이다.
* **가중치가 업데이트되며 원래 목적식의 극소점 부근(local minima)에 도착해도 미니 배치를 사용하면 매번 목적식이 변화하며 확률적으로 극소점이 아니게되어 탈출할 가능성이 있다.**

일반 경사하강법보다 덜 정확한 방향으로 움직이지만 목적식의 최소점을 향해 가중치를 업데이트하며 적은 연산으로 빠르게 가중치를 갱신하므로 수렴이 빠르다.

* 정리
  * 일반 경사하강법보다 적은 연산으로 빠르게 가중치 업데이트 - 수렴이 빠르다
    * 그래디언트가 향하는 방향이 덜 정확하지만 비슷하긴하다.
  * 볼록 함수가 아니더라도 최적화 가능
    * 미니 배치의 구성에 따라서 목적식이 달라지므로 local minima에 빠졌을때 확률적으로 극소점이 아니게되어 탈출 가능성이 있다.
  * 학습률, 학습횟수와 더불어 미니 배치 크기도 중요하다.
    * 너무 작다면 일반 경사하강법보다 수렴이 느리다.
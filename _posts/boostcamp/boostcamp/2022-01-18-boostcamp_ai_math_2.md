---
title: "AI Math 2 - 행렬"
excerpt: "행렬"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-18 10:14
last_modified_at: 2022-01-18 10:14
---

# 행렬
벡터(행벡터)를 원소로 가지는 2차원 배열. 벡터가 공간상의 한 점을 의미하므로 행렬은 여러 점들의 집합. 한 점은 하나의 데이터로 생각할 수 있어 $x_{ij}$는 $i$번째 데이터의 $j$번째 변수의 값.

$$\boldsymbol{\mathrm{X}}=
\begin{bmatrix} \boldsymbol{\mathrm{x}}_1 \\ 
                \boldsymbol{\mathrm{x}}_2 \\
                \vdots \\
                \boldsymbol{\mathrm{x}}_n
\end{bmatrix}=
\begin{bmatrix} x_{11} & x_{12} & \cdots & x_{1m} \\
                x_{21} & x_{22} & \cdots & x_{2m} \\
                \vdots & \vdots &        & \vdots \\
                x_{n1} & x_{n2} & \cdots & x_{nm} \\
\end{bmatrix}
\begin{matrix} \boldsymbol{\mathrm{x}}_1 \\ 
               \boldsymbol{\mathrm{x}}_2 \\
               \vdots \\
               \boldsymbol{\mathrm{x}}_n
\end{matrix} = (x_{ij})
$$

## 전치 행렬
행렬의 행과 열을 교환. 행렬과 벡터 모두에 적용될 수 있다. $n\times m$ 행렬이 $m\times n$ 행렬이 되고 행벡터는 열벡터로, 열벡터는 행벡터가 된다.

$$\boldsymbol{\mathrm{X}}^\top=(x_{ji})$$

## 연산
* 덧셈, 뺄셈, 성분곱$\bigodot$: 성분끼리 연산
* 스칼라곱: 성분에 스칼라곱
* 행렬 곱셈: 
  * $$\boldsymbol{\mathrm{XY}}=\left( \sum_k x_{ik}y_{kj} \right)$$
  * 두 행렬 곱의 $i$행 $j$열의 성분은 $\boldsymbol{\mathrm{X}}$의 $i$행 행벡터와 $\boldsymbol{\mathrm{Y}}$의 $j$열 열벡터의 내적. $\boldsymbol{\mathrm{X}}$의 열의 수와 $\boldsymbol{\mathrm{Y}}$의 행의 수가 같아야함.

### numpy 연산
* 행렬@행렬: 행렬곱
* 행렬*행렬: 성분곱
* np.inner: 두 행렬의 **행벡터**끼리 내적. $\boldsymbol{\mathrm{XY}}^\top$

## 행렬은 벡터공간의 연산자
linear transform(행렬곱)을 통해 벡터를 다른 차원의 공간으로 보내는 방식으로 패턴추출, 데이터 압축에 사용된다.

$$
\begin{matrix}
  \begin{bmatrix}
    z_1 \\ z_2 \\ \vdots \\ z_n
  \end{bmatrix} & = &
  \begin{bmatrix}
    a_{11} & a_{12} & \cdots & a_{1m} \\
    a_{21} & a_{22} & \cdots & a_{2m} \\
    \vdots & \vdots &  & \vdots \\
    a_{n1} & a_{n2} & \cdots & a_{nm} \\
  \end{bmatrix} &
  \begin{bmatrix}
    x_1 \\ x_2 \\ \vdots \\ x_m
  \end{bmatrix} \\
  \boldsymbol{\mathrm{z}} & & \boldsymbol{\mathrm{A}} & \boldsymbol{\mathrm{x}}
\end{matrix}$$

![linear_transform](/assets/images/post/220118/boostcamp_ai_math_2/linear_transform.png){: .align-center}

## 역행렬 (inverse matrix)
행렬 연산을 거꾸로 되돌리는 행렬. 행과 열의 수가 같고 행렬식determinant가 0이 아닌 경우에만 역행렬이 존재한다.
`np.linalg.inv`로 계산.

$$\boldsymbol{\mathrm{AA}}^{-1}=\boldsymbol{\mathrm{A^{-1}A}}=\boldsymbol{\mathrm{I}}$$

### 역행렬이 없는 경우? - 유사 역행렬, 무어-펜로즈 역행렬

$\boldsymbol{A}\in\mathbb{R}^{n\times m}$
* $\boldsymbol{A}$의 행이 열보다 많을때, $n>m$ <br>
  $\boldsymbol{A}^+=(\boldsymbol{A}^\top\boldsymbol{A})^{-1}\boldsymbol{A}^\top$<br>
  $\boldsymbol{A}^+\boldsymbol{A}=\boldsymbol{I}$
* $\boldsymbol{A}$의 열이 행보다 많을때, $n<m$ <br>
  $\boldsymbol{A}^+=\boldsymbol{A}^\top(\boldsymbol{A}\boldsymbol{A}^\top)^{-1}$ <br>
  $\boldsymbol{A}\boldsymbol{A}^+=\boldsymbol{I}$

`np.linalg.pinv`로 계산.

### 활용
* 연립방정식의 풀이 (식의 수가 변수의 수보다 적거나 같아야함.)
* 선형회귀 (데이터의 수가 변수의 수보다 많음.)
  * $\boldsymbol{Xw}=\boldsymbol{y}$
  * 연립방정식은 풀리지 않음. 그러므로 $\boldsymbol{Xw}=\hat{\boldsymbol{y}}\approx\boldsymbol{y}$ 인 $\boldsymbol{w}$가 목표
  * $\boldsymbol{w}=\boldsymbol{X}^+\boldsymbol{y}$
---
title: "AI Math 1 - 벡터"
excerpt: "벡터, 노름(norm)"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-17 17:11
last_modified_at: 2022-01-17 17:11
---
# 벡터
* 숫자를 원소로 가지는 리스트 또는 배열
  * 열벡터: $$\boldsymbol{\mathrm{x}}=\begin{bmatrix} 1\\ 2\\ 3 \end{bmatrix}$$, 행벡터: $\boldsymbol{\mathrm{x}}^\top=\begin{bmatrix}1&2&3\end{bmatrix}$
  * 벡터의 차원: 원소의 수.
* 공간의 한 점을 의미, 원점으로부터의 상대적인 위치를 표현.
* 벡터의 연산
  * 두 벡터의 성분곱(Hadamard product): 벡터의 같은 위치의 원소끼리 곱.
  * 두 벡터의 덧셈, 뺄셈: 한 벡터로부터의 상대적 위치 이동 - 원점이 아닌 한 벡터로부터의 상대적인 위치 표현.
* 벡터의 노름(norm)
  * 원점에서부터의 거리
  * norm의 종류
    * $\boldsymbol{\mathrm{x}}^\top=\begin{bmatrix}x_1&x_2&\cdots&x_d\end{bmatrix}$
    * L1 norm: $\lVert\boldsymbol{\mathrm{x}}\rVert_1=\sum^d_{i=1}\lvert x_i\rvert $, 성분 절대값의 합 = 좌표축을 따라 움직이는 거리의 합.
      * ```python3
        x = np.array([1, 2, 3])
        x_abs = np.abs(x)
        l1_x_norm = np.sum(x_abs)
        ```
    * L2 norm: $\lVert\boldsymbol{\mathrm{x}}\rVert_2=\sqrt{\sum^d_{i=1}\lvert x_i\rvert^2}$, 성분 절대값 제곱의 합. 유클리드 거리.
      * ```python3
        x = np.array([1, 2, 3])
        x_square = x * x  # 성분곱
        x_square_sum = np.sum(x_square)
        l2_x_norm = np.sqrt(x_square_sum)
        ```
    * 차원 d에 무관하게 성립
  * norm의 종류에 따라 기하학적 성질 다름 - ex) 원 - 원점으로부터 같은 거리에 있는 점들
    * |L1                                                                |L2                                                                |
    |------------------------------------------------------------------|------------------------------------------------------------------|
    |![l1](/assets/images/post/220117/boostcamp_ai_math_1/l1_norm.png)|![l2](/assets/images/post/220117/boostcamp_ai_math_1/l2_norm.png)|
  * 두 벡터 사이의 각 - 제 2 코사인 법칙
    * $\mathrm{cos}\theta = \frac{\lVert\boldsymbol{\mathrm{x}}\rVert^2_2+\lVert\boldsymbol{\mathrm{y}}\rVert^2_2-\lVert\boldsymbol{\mathrm{x-y}}\rVert^2_2}{2\lVert x\rVert_2\lVert y\rVert_2}=\frac{\boldsymbol{\mathrm{x}}\cdot\boldsymbol{\mathrm{y}}}{\lVert x\rVert_2\lVert y\rVert_2}$

* 내적의 해석
  * 정사영된 벡터의 길이와 관련 - 정사영의 길이를 $\lVert\boldsymbol{\mathrm{y}}\rVert$만큼 조정
    * ![내적](/assets/images/post/220117/boostcamp_ai_math_1/dot_prod.png)

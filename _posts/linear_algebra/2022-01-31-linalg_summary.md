---
title: "선형대수 복습 & 정리"
excerpt: "선형대수 정리하기"

categories:
  - linear algebra
tags:
  - [linear algebra]

toc: true
toc_sticky: true

date: 2022-01-31T19:25:00+09:00
last_modified_at: 2022-03-05T20:12:00+09:00
---

# 선형대수

|--------|----------|
|![딥러닝을위한선형대수학](https://www.hanbit.co.kr/data/books/B9479195027_m.jpg)|![mml](https://mml-book.github.io/static/images/mml-book-cover.jpg)|

선형대수를 체계적으로 공부하지 않아서 지식이 부정확하거나 파편화되어있어 기초가 부족한 상태;

# 선형대수 사전 지식

## 1.1 $\boldsymbol{Ax}$

\# 행렬-벡터 곱셈, $\boldsymbol{Ax}$, 행렬의 영공간과 랭크

<br>
🌝. $\boldsymbol{Ax}$는 $\boldsymbol{A}$의 열의 일차결합. $\boldsymbol{A}$의 열의 일차결합은 $\boldsymbol{A}$의 열공간을 모두 채운다.

$\boldsymbol{A}$의 열공간: $\boldsymbol{A}$의 열들의 모든 일차결합($\boldsymbol{Ax}$로 만들 수 있는 모든 벡터)으로 이루어진 벡터공간

* 그룹, $\mathrm{G}:=(\mathcal{G}, \otimes)$
  * 집합 $\mathcal{G}$와 연산 $\otimes:\mathcal{G}\times\mathcal{G}\rightarrow\mathcal{G}$
    * $\mathcal{G}$는 $\otimes$에 닫혀있다.
    * $\otimes$에 대해 결합법칙 성립: $(x\otimes y)\otimes z = x \otimes (y\otimes z)$
    * 항등원 $e$ 존재: $\exists e \in \mathcal{G}: x\otimes e = e\otimes x = x$
    * 역원 $a$ 존재: $\exists a \in \mathcal{G}: x\otimes a = a\otimes x = e$
  * 아벨 그룹, abelian group
    * $\otimes$에 대해 교환법칙이 성립하는 그룹: $x\otimes y = y\otimes x$
* 벡터공간, $\mathrm{V}=(\mathcal{V}, +, \cdot)$
  * $\mathcal{V}$는 벡터 집합
  * $(\mathcal{V}, +)$가 아벨 그룹
    * 항등원, 역원, 교환법칙, 결합법칙
  * 분배 법칙 성립 / $\forall\boldsymbol{x,y}\in\mathcal{V}, \forall\lambda,\psi\in\mathbb{R}$
    * $\lambda\cdot(\boldsymbol{x}+\boldsymbol{y}) = \lambda\cdot\boldsymbol{x}+\lambda\cdot\boldsymbol{y}$
    * $(\lambda+\psi)\cdot\boldsymbol{x}=\lambda\cdot\boldsymbol{x}+\psi\cdot\boldsymbol{x}$
  * $\cdot$ 결합법칙 성립
    * $\lambda\cdot(\psi\cdot\boldsymbol{x})=(\lambda\psi)\cdot\boldsymbol{x}$
  * $\cdot$ 항등원 존재
    * $1\cdot \boldsymbol{x}=\boldsymbol{x}$

🌝. 독립, independent: 벡터들의 일차결합이 $\boldsymbol{0}$이 되는 경우는 오직 '각 벡터의 일차결합 계수가 모두 0인 경우'.
* $\mathbb{R}^3$의 3개 벡터가 일차독립이면 그 벡터들은 가역행렬을 생성.
* $\boldsymbol{Ax}=\boldsymbol{0}$의 해는 $\boldsymbol{x}=(0, 0, 0)$이 유일. $\boldsymbol{Ax}=\boldsymbol{b}$의 해는 $\boldsymbol{x}=\boldsymbol{A}^{-1}\boldsymbol{b}$가 유일.

$\mathbb{R}^3$의 3개 벡터가 일차독립이면 기저이므로 $\boldsymbol{v}\in\mathbb{R}^3$도 3개 벡터의 일차결합으로 표현 가능 $\rightarrow \boldsymbol{Ax}=\boldsymbol{v}$. 여기서 $\boldsymbol{x}$는 유일하므로 $\boldsymbol{A}$는 단전사. 그러므로 역행렬 존재.

🌝. 한 공간의 모든 벡터는 기저 벡터의 일차 결합으로 표현할 수 있다.

* 한 공간의 기저는 여럿일 수 있지만 기저의 기저벡터 수는 항상 일정.

🌝. 랭크, rank: 행렬의 일차독립인 열의 수 = 열공간의 차원 = 행공간의 차원

* **행렬 분해, factorization**: $\boldsymbol{A}=\boldsymbol{CR}$
  * $\boldsymbol{C}$: $\boldsymbol{A}$의 일차독립인 열벡터로 이루어진 행렬. 열공간의 기저.
    * $\boldsymbol{R}$의 $j$번째 열은 $\boldsymbol{C}$의 열들을 선형결합하여 $\boldsymbol{A}$의 $j$번째 열을 표현하는 방법.
  * $\boldsymbol{R}$: 0행을 제외한 기약행 사다리꼴행렬, $\mathrm{rref}(\boldsymbol{A})$. $\boldsymbol{R}$의 행벡터는 $\boldsymbol{A}$의 행공간의 기저. (즉, 일차독립이다.)
    * $\boldsymbol{C}$의 $i$번째 행은 $\boldsymbol{R}$의 행들을 선형결합하여 $\boldsymbol{A}$의 $i$번째 행을 표현하는 방법.
* 랭크 정리: 일차독립인 열의 수 = 일차독립인 행의 수

🌝. $\boldsymbol{A}=\boldsymbol{CMR}$

* $\boldsymbol{A}=\boldsymbol{CR}$의 $\boldsymbol{R}$은 $\boldsymbol{A}$을 기약행 사다리꼴로 변형하면서 독립인 행을 그대로 가져오지 않음. 
* $\boldsymbol{A}=\boldsymbol{CMR}$의 $\boldsymbol{R}$은 $\boldsymbol{A}$ 독립인 행을 그대로 가져오고 $r\times r$ 혼합행렬 $\boldsymbol{M}$이 중계역할을 해준다.
* $\boldsymbol{M}=(\boldsymbol{C^\top C})^{-1}(\boldsymbol{C^\top A R^\top})(\boldsymbol{RR^\top})^{-1}$
* 행렬이 크면 클수록 $\boldsymbol{A}=\boldsymbol{CR}$, $\boldsymbol{A}=\boldsymbol{CMR}$이 중요. $\boldsymbol{A}$의 데이터를 유지하는 성질.


## 1.2 $\boldsymbol{AB}$

\# 행렬곱, 열과 행의 곱

<br>
🌝. 내적을 이용한 행렬의 곱셈: $\boldsymbol{AB=C}$, $c_{ij}=\sum_{k=1}^n a_{ik}b_{kj}$

$\boldsymbol{C}$의 $i$행, $j$열 성분은 $\boldsymbol{A}$의 $i$행과 $\boldsymbol{B}$의 $j$열의 내적

$\boldsymbol{A}=(m\times n), \boldsymbol{B}=(n\times p)$라면 곱셈 횟수는 $mnp$.  $n$번의 곱셈 연산이 필요한 행과 열의 내적이 $mp$번 필요함.

🌝. 열벡터와 행벡터의 곱셈 = 랭크 1인 행렬

$$\boldsymbol{uv}^\top=
\begin{bmatrix} 1 \\ 2 \\ 3 \end{bmatrix}
\begin{bmatrix} 4 & 5 & 6 \end{bmatrix} =
\begin{bmatrix} 4 & 5 & 6 \\ 8 & 10 & 12 \\ 12 & 15 & 18 \end{bmatrix}$$

영행렬이 아닌 $\boldsymbol{uv}^\top$는 모든 열이 $\boldsymbol{u}$의 배수, 모든 행이 $\boldsymbol{v}^\top$의 배수: 랭크 1인 행렬

🌝. $\boldsymbol{AB}$ = 랭크 1인 행렬의 합

$$
\boldsymbol{AB}=
\begin{bmatrix}
  | & & | \\
  \boldsymbol{a}_1 & \cdots & \boldsymbol{a}_n \\
  | & & | \\
\end{bmatrix}
\begin{bmatrix}
  - & \boldsymbol{b}^*_1 & - \\
  & \vdots & \\
  - & \boldsymbol{b}^*_n & - \\
\end{bmatrix} =
\sum_{k=1}^n \boldsymbol{a}_k\boldsymbol{b}^*_k
$$

$\boldsymbol{A}$의 열들을 선형결합하여 $\boldsymbol{AB}$의 $j$번째 열을 만들 때 $\boldsymbol{a}_k$의 계수는 $\boldsymbol{b}^*_k$의 $j$번째 성분이다.

= $\boldsymbol{b}^*_k$의 $j$번째 성분은 $\boldsymbol{AB}$의 $j$번째 열을 만들 때 필요한 $\boldsymbol{a}_k$의 수이다.

필요한 곱셈 연산 횟수는 마찬가지로 $mnp$. $mp$만큼의 곱셈이 필요한 랭크 1 행렬 계산이 $n$번 필요함.

$c_{ij}=\sum_{k=1}^n a_{ik}b_{kj}$ = $\boldsymbol{a}_k\boldsymbol{b}^*_k$의 $(i, j)$ 성분을 모두 더하기

🌝. $\boldsymbol{S}=\boldsymbol{Q\Lambda Q}^\top$

행렬 분해의 한 종류. $\boldsymbol{S}=\boldsymbol{S}^\top$는 대칭 행렬. **모든 대칭행렬은 $n$개의 실수 고윳값과 $n$개의 정규직교인 고유벡터를 가진다.**

**고윳값, 고유벡터, 직교벡터**

* 행렬의 고유벡터는 행렬에 곱해도 방향이 바뀌지 않고 고윳값 $\lambda$에 의해 크기만 변하는 벡터.

$$\boldsymbol{Sx} = \lambda\boldsymbol{x}$$

* 정규직교인 벡터들은 서로 직교이고 크기가 1인 벡터들이다.

$$\boldsymbol{q}_i \cdot \boldsymbol{q}_j = \begin{cases} 0 & (i \neq j) \\ 1 & (i = j) \end{cases}$$

$\boldsymbol{S}$는 대칭행렬, $\boldsymbol{Q}$는 $\boldsymbol{S}$의 직교 단위 고유벡터 $\boldsymbol{q}_1,\cdots, \boldsymbol{q}_n$이 열인 직교행렬이다.

$$
\boldsymbol{SQ}=
\boldsymbol{S}\begin{bmatrix}
  \boldsymbol{q}_1 & \cdots & \boldsymbol{q}_n
\end{bmatrix}=
\begin{bmatrix}
  \lambda_1\boldsymbol{q}_1 & \cdots & \lambda_n\boldsymbol{q}_n
\end{bmatrix}=
\begin{bmatrix}
  \boldsymbol{q}_1 & \cdots & \boldsymbol{q}_n
\end{bmatrix}
\begin{bmatrix}
\lambda_1 & & \\
& \ddots & \\
& & \lambda_n
\end{bmatrix}
=
\boldsymbol{Q\Lambda}
$$

$\boldsymbol{\Lambda}$는 대각성분이 $\boldsymbol{S}$의 고윳값인 대각행렬이다. 대각행렬 $\boldsymbol{\Lambda}$의 왼쪽에 행렬을 곱하면 행렬의 각 열에 각 $\lambda_i$를 곱하는 것이고, 오른쪽에 행렬을 곱하면 행렬의 각 행에 각 $\lambda_i$를 곱하는 것이다. (열과 행의 곱)

$\boldsymbol{Q}$는 직교 행렬이므로 $\boldsymbol{Q^\top Q}=\boldsymbol{I}, \boldsymbol{Q}^{-1}=\boldsymbol{Q}^\top$

$\boldsymbol{S}=\boldsymbol{Q\Lambda Q}^\top$에서 각 $\lambda_k$와 $\boldsymbol{q}_k$는 랭크 1인 행렬 $\lambda_k\boldsymbol{q}_k\boldsymbol{q}^\top_k$을 만들고 이 행렬은 대칭행렬이다.


## 1.3 네 가지 기본 부분공간

\# 열공간, 행공간, 영공간, 좌영공간

<br>

🌝. $m \times n$ 행렬 $\boldsymbol{A}$는 2개의 $\mathbb{R}^m$ 부분공간과 2개의 $\mathbb{R}^n$ 부분공간을 갖는다.

* 열공간 $C(\boldsymbol{A})$: $\boldsymbol{A}$의 열들의 모든 일차 결합 ($\subseteq \mathbb{R}^m$)
* 행공간 $C(\boldsymbol{A}^\top)$: $\boldsymbol{A}$의 행들의 모든 일차 결합 ($\subseteq \mathbb{R}^n$)
* 영공간 $N(\boldsymbol{A})$: $\boldsymbol{Ax}=\boldsymbol{0}$의 모든 해 ($\subseteq \mathbb{R}^n$)
* 좌영공간 $N(\boldsymbol{A}^\top)$: $\boldsymbol{A^\top y}=\boldsymbol{0}$의 모든 해 ($\subseteq \mathbb{R}^m$)

행공간과 영공간, 열공간과 좌영공간은 직교한다.

🌝. 셈 법칙: $r$개의 독립인 일차방정식 $\boldsymbol{Ax}=\boldsymbol{0}$에는 일차독립인 해가 $n-r$개 있다.

$\boldsymbol{A}$에 $r$개의 일차독립인 열이 있고 $n-r$개의 종속인 열들을 독립인 열들로 표현할 수 있다. 그러므로 영공간의 차원, 기저의 수가 $n-r$개.

🌝. $\boldsymbol{AB}$와 $\boldsymbol{A} + \boldsymbol{B}$의 랭크
* $\text{rank}(\boldsymbol{AB})\leq \text{rank}(\boldsymbol{A}), \text{rank}(\boldsymbol{B})$
  * $\boldsymbol{AB}$는 $\boldsymbol{A}$의 열들의 일차결합으로 $C(\boldsymbol{AB})\subseteq C(\boldsymbol{A})$
  * $\boldsymbol{AB}$는 $\boldsymbol{B}$의 행들의 일차결합으로 $C((\boldsymbol{AB})^\top)\subseteq C(\boldsymbol{B}^\top)$
* $\text{rank}(\boldsymbol{A}+\boldsymbol{B}) \leq \text{rank}(\boldsymbol{A}) + \text{rank}(\boldsymbol{B})$
  * 두 행렬의 합은 열들의 결합으로 각 행렬 열공간 기저의 결합이다.
* $\text{rank}(\boldsymbol{A^\top A})=\text{rank}(\boldsymbol{AA^\top})=\text{rank}(\boldsymbol{A})=\text{rank}(\boldsymbol{A}^\top)$
  * $N(\boldsymbol{A^\top A}) = N(\boldsymbol{A})$
    * $\boldsymbol{Ax}=0$이면 $\boldsymbol{A^\top Ax}=0, N(\boldsymbol{A})\subseteq N(\boldsymbol{A^\top A})$ 
    * $\boldsymbol{A^\top Ax}=0$이면 $\boldsymbol{x^\top A^\top Ax}=(\boldsymbol{Ax})^\top\boldsymbol{Ax}=\Vert\boldsymbol{Ax}\Vert^2=0$이므로 $\boldsymbol{Ax}=0, N(\boldsymbol{A^\top A})\subseteq N(\boldsymbol{A})$
* $m \times r$인 $\boldsymbol{A}$와 $r \times n$인 $\boldsymbol{B}$의 랭크가 $r$이면 $\text{rank}(\boldsymbol{AB})=r$
  * $\text{rank}(\boldsymbol{A^\top A})=\text{rank}(\boldsymbol{BB^\top})=r$이므로 $\boldsymbol{A^\top A, BB^\top}$은 가역행렬. 가역행렬끼리의 곱도 가역행렬이므로 $r = \text{rank}(\boldsymbol{A^\top ABB^\top}) \leq \text{rank}(\boldsymbol{ABB^\top}) \leq \text{rank}(\boldsymbol{AB}) \leq \text{rank}(\boldsymbol{A}) = r$

랭크가 $r$인 행렬끼리 곱한다고 무조건 랭크가 $r$인 것은 아니다.
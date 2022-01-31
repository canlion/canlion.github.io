---
title: "선형대수 복습 & 정리"
excerpt: "선형대수 정리하기"

categories:
  - linalg
tags:
  - [linalg]

toc: true
toc_sticky: true

date: 2022-01-31 19:25
last_modified_at: 2022-01-31 19:25
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
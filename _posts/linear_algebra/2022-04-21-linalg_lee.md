---
title: "선형대수 입문 - 1강. 행렬과 행렬식"
excerpt: "이상엽math 선형대수학 강의"

categories:
  - linear algebra
tags:
  - [linear algebra]

toc: true
toc_sticky: true

date: 2022-04-21T12:50:00+09:00
last_modified_at: 2022-04-21T12:50:00+09:00
---

선형대수를 많이 잊어버려서 내용정리된 책을 읽는건 너무 어렵고 입문 강의부터 다시..

🤖 유튜브 이상엽math 채널의 선형대수학 강의: [링크](https://www.youtube.com/playlist?list=PL127T2Zu76FuVMq1UQnZv9SG-GFIdZfLg)

<br>

# 🤖 1강. 행렬과 행렬식

## 1. 행렬

### 용어 및 표기

* 행렬은 대문자로 표현: $\mathrm{A}$
* 성분(component, entry): 행렬의 구성원. 항, 원소
  * 표기: $\mathrm{a_{ij}}$ - $\mathrm{A}$의 i행, j열의 성분
  * 행렬을 $\mathrm{A} = (\mathrm{a_{ij}})=(\mathrm{a_{ij}})_{m\times n}$으로 표현하기도 한다.
* 주대각선(main diagonal): $\mathrm{a_{11}}$부터 시작하는 대각선 위치
* 대각성분: 주대각선에 위치한 성분
* 전치행렬(transpose matrix): 주대각선을 축으로 뒤집은 행렬. $\mathrm{A}=(\mathrm{a_{ij}})$를 전치하면 $\mathrm{A}^\top=(\mathrm{a_{ji}})$
* 대칭행렬(symmetric matrix): $\mathrm{A} = \mathrm{A}^\top$인 행렬. 전치해도 바뀌지 않음.
* 정사각행렬(square matrix): 행의 수와 열의 수가 같음. 모양이 정사각형.
* 단위행렬(unit matrix, identity matrix): $\mathrm{I}$, 모든 대각성분은 1이고 나머지 성분은 0인 행렬. 행과 열의 수가 $\mathrm{n}$인 단위행렬은 $\mathrm{I=I_n=I_{n\times n}}$으로 표기

### 행렬의 연산

$\mathrm{m\times n}$ 행렬 $\mathrm{A=(a_{ij}), B=(b_{ij})}$에 대해
* 덧셈, 뺄셈: $\mathrm{A\pm B} = (a_{ij} \pm b_{ij})$, 같은 위치의 성분끼리 더하고 뺀다.
* 상수배, 스칼라곱: $c\mathrm{A} = (ca_{ij})$, 모든 성분에 상수 $\mathrm{c}$를 곱한다.

$\mathrm{m\times n}$ 행렬 $\mathrm{A}=(a_{ij})$와 $\mathrm{n\times r}$ 행렬 $\mathrm{B}=(b_{ij})$에 대해
* 행렬곱: $\mathrm{AB=}c_{ij}$는 $\mathrm{m\times r}$ 행렬
  * $$c_{ij}=\sum_{k=1}^n a_{ik}b_{kj}=$$ A의 i행과 B의 j열의 내적
  * **일반적으로 교환법칙 성립하지 않는다.**
    * 나중에 더 배워야하지만 행렬의 곱은 함수의 합성과 같은 맥락. 함수의 합성도 일반적으로 교환법칙이 성립하지 않음.

**함수의 합성과 행렬의 곱**

행렬의 곱에 (x, y) 벡터를 곱하면 함수의 합성과 같음. 행렬에 벡터를 곱하는 것과 함수에 값을 대입하는 것은 같다? 행렬은 함수다?

$$f(x, y) = (ax+by, cx+dy), g(x, y) = (px+qy, rx+sy)$$

$$f\circ g=((ap+br)x+(aq+bs)y, (cp+dr)x+(cq+ds)y)$$

$$\mathrm{F}=
\begin{bmatrix}a&b\\c&d\end{bmatrix}, 
\mathrm{G}=
\begin{bmatrix}p&q\\r&s\end{bmatrix}
$$

$$\mathrm{FG}=
\begin{bmatrix}ap+br&aq+bs\\cp+dr&cq+ds\end{bmatrix}$$

## 2. 연립일차방정식

### 연립일차방정식의 행렬 표현

$$\begin{cases}
x+2y=5\\2x+3y=8
\end{cases}$$

* 첨가행렬로 표현:
  $$\begin{bmatrix}1&2&5\\2&3&8\end{bmatrix}$$, 미지수의 계수와 상수를 한덩어리로 표현 -> 가우스-조던 소거법 적용
* 계수행렬, 미지수행렬, 상수행렬로 표현:
  $$\begin{bmatrix}1&2\\ 2&3\end{bmatrix} \begin{bmatrix}x\\y\end{bmatrix}=\begin{bmatrix}5\\8\end{bmatrix}$$ -> 역행렬로 풀이

### 가우스-조던 소거법

기본 행 연산을 반복해 첨가행렬을 기약행사다리꼴로 변환하여 해를 구한다.

**기약행사다리꼴**

* 각 행의 첫 0이 아닌 성분(피벗)이 1이고 그 성분이 위치한 열에서 다른 성분들은 모두 0인 행렬
* 임의의 행의 피벗은 위의 행들의 피벗보다 오른쪽에 있어야한다.

**기본 행 연산**

* 한 행을 상수배
* 한 행을 상수배하여 다른 행에 더함
* 두 행을 교환

손으로 연립방정식을 풀면서 미지수를 소거하는 과정이 기본 행 연산과 같다.

### 역행렬

연립일차방정식 $\mathrm{AX=B}$에서 $\mathrm{A^{-1}}$이 존재하면 $\mathrm{X=A^{-1}B}$. 역행렬이 없다면 연립방정식은 해가 없던지(불능), 해가 무수히 많다(부정).

## 3. 행렬식

### 행렬식

determinant. 정사각행렬 $\mathrm{A}$를 하나의 수로 대응시키는 특별한 함수.

$$\mathrm{det(A)=\vert A\vert}$$

* 0x0 행렬: $\mathrm{det( ) = 0}$
* 1x1 행렬: $\mathrm{det}(a) = a$
* 2x2 행렬: $$\mathrm{det\left(\begin{matrix}a_{11}&a_{12}\\a_{21}&a_{22}\end{matrix}\right)}=a_{11}a_{22}-a_{12}a_{21}$$

3x3 행렬의 경우 조금 복잡해지는데...

$$\mathrm{
  det\left(\begin{matrix}
    a_{11}&a_{12}&a_{13} \\
    a_{21}&a_{22}&a_{23} \\
    a_{31}&a_{32}&a_{33} \\
  \end{matrix}\right)}
  = a_{11}\mathrm{M_{11}} - a_{12}\mathrm{M_{12}} + a_{13}\mathrm{M_{13}}
$$

여기서 소행렬식 $\mathrm{M}_{ij}$는 $\mathrm{A}$에서 i행과 j열을 제거하고 남은 행렬의 행렬식이다.

$$\mathrm{M}_{11} = \mathrm{det}\left(\begin{matrix}a_{22}&a_{23}\\a_{32}&a_{33}\end{matrix}\right)$$

$a_{11}\mathrm{M_{11}} - a_{12}\mathrm{M_{12}} + a_{13}\mathrm{M_{13}}$에서 각 항들이 +, -, +, -, ... 가 반복되는 점을 주의하고 꼭 1행을 선택하여 계산할 필요는 없음. 행도 좋고 열도 좋음. 0이 많은 행이나 열을 선택하면 계산이 편하다.

2x2 행렬도 위와 같은 과정으로 $a_{11}a_{22}-a_{12}a_{21}$ 식이 유도된다.

사루스 법칙을 이용하면 3x3 행렬의 판별식을 쉽게 구할 수 있다.

### 역행렬

행렬은 단위행렬이 항등원 역할. 행렬에 역행렬을 곱하면 단위 행렬이 나오므로 역행렬은 역원의 역할.

$$\mathrm{AI=A,\quad AA^{-1}=I}$$

**행렬식이 0이면 역행렬은 존재하지 않는다.**

여인수 cofactor는 다음과 같이 계산된다. 행렬의 특정 행과 열을 제거하고 남은 행렬의 행렬식에 제거한 행과 열에 따라 부호를 붙인다.

$$\mathrm{C_{ij}=(-1)^{i+j}M_{ij}}$$

행렬 $\mathrm{A}$의 여인수들로 이루어진 행렬을 여인수 행렬이라하고, 여인수 행렬의 **전치** 행렬을 수반행렬 adj(A)이라 한다.

$$\mathrm{adj(A)=\begin{bmatrix}
C_{11} & C_{21} & C_{31} \\
C_{12} & C_{22} & C_{32} \\
C_{13} & C_{23} & C_{33} \\
\end{bmatrix}}$$

여기서 $\mathrm{A}$와 수반행렬 $\mathrm{adj(A)}$를 곱하면?

$$\mathrm{A\cdot adj(A)}=\text{?}$$

1행 1열 성분은 $a_{11}\mathrm{C_{11}}+a_{12}\mathrm{C_{12}}+a_{13}\mathrm{C_{13} = \mathrm{det(A)}}$, +-+-... 반복이 안되는건 여인수에 이미 부호가 붙었기 때문이다. 1행 2열 성분은 0 (어떻게 설명해야.... 한 쌍의 원소가 소행렬식의 계수로 한 번, 소행렬식의 원소로 한 번씩 포함되어 0이 된다. 전개해보면 그렇다). 쭉쭉 풀어보면

$$
\mathrm{A\cdot adj(A)=
  \begin{bmatrix}
    \mathrm{det(A)}&0&0\\
    0&\mathrm{det(A)}&0\\
    0&0&\mathrm{det(A)}\\
  \end{bmatrix}}
= \mathrm{det(A)\cdot I}
$$

$$\mathrm{A^{-1}=\frac{1}{det(A)}adj(A)}$$

그러므로 **$\mathrm{det(A)}=0$ 이면 역행렬은 존재하지 않는다.**

행렬 $\mathrm{A}$와 역행렬 $\mathrm{A}^{-1}$은 교환법칙이 성립한다. 보기좋게 $\mathrm{A^{-1}=B}$라 하면

$$
\begin{matrix}
\mathrm{AB} & = & \mathrm{I} \\
& = & \mathrm{BIB^{-1}} \\
& = & \mathrm{BABB^{-1}} \\
& = & \mathrm{BAI} \\
& = & \mathrm{BA}
\end{matrix}
$$

### 크래머 공식

가우스-조던 소거법이나 역행렬을 이용해 모든 해를 한 번에 구하는 방식이 아닌 미지수 중 하나의 해를 구하는 공식.

$\mathrm{AX=B}$에서 $\mathrm{A}$가 정사각 행렬일 때

$$
x_j = \frac{\mathrm{det(A_j)}}{\mathrm{det(A)}} =
\frac{
  \begin{bmatrix}
    a_{11} & \cdots & b_1 & \cdots & a_{1n} \\
    a_{21} & \cdots & b_2 & \cdots & a_{2n} \\
    \vdots & & \vdots & & \vdots \\
    a_{n1} & \cdots & b_n & \cdots & a_{nn} \\
  \end{bmatrix}
}{
  \begin{bmatrix}
    a_{11} & \cdots & a_{1j} & \cdots & a_{1n} \\
    a_{21} & \cdots & a_{2j} & \cdots & a_{2n} \\
    \vdots & & \vdots & & \vdots \\
    a_{n1} & \cdots & a_{nj} & \cdots & a_{nn} \\
  \end{bmatrix}
}
$$

$\mathrm{A_j}$는 j번째 열을 행렬 $$B=\begin{bmatrix}b_1\\b_2\\\vdots\\b_n\end{bmatrix}$$으로 교체한 행렬.

예를 들어 역행렬을 이용한다면 $x_2$는 다음과 같이 구해진다. 

$$\mathrm{X=A^{-1}B=}
\frac{1}{\mathrm{det(A)}}\begin{bmatrix}
  \mathrm{C_{11}}&\mathrm{C_{21}}&\mathrm{C_{31}}\\
  \mathrm{C_{12}}&\mathrm{C_{22}}&\mathrm{C_{32}}\\
  \mathrm{C_{13}}&\mathrm{C_{23}}&\mathrm{C_{33}}\\
\end{bmatrix}
\begin{bmatrix}
  b_1\\b_2\\b_3
\end{bmatrix}
$$

$$x_2 = \frac{1}{\mathrm{det(A)}}(b_1\mathrm{C_{12}}+b_2\mathrm{C_{22}}+b_3\mathrm{C_{32}})$$

크레이머 공식을 사용해보면 (행렬식 구할 때 교체한 2열을 여인수의 계수로 사용한다.)

$$\mathrm{det(A_2)}=\mathrm{det}\left(
\begin{matrix}
  a_{11} & b_1 & a_{13} \\
  a_{21} & b_2 & a_{23} \\
  a_{31} & b_3 & a_{33} \\
\end{matrix}
\right) = b_1\mathrm{C_{12}} + b_2\mathrm{C_{22}} + b_3\mathrm{C_{32}}$$

$$x_2 = \frac{1}{\mathrm{det(A)}}(b_1\mathrm{C_{12}}+b_2\mathrm{C_{22}}+b_3\mathrm{C_{32}})$$

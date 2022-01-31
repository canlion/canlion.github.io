---
title: "AI Math 9 - CNN 첫걸음"
excerpt: "Convolution 연산과 역전파"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-21 10:21
last_modified_at: 2022-01-21 10:21
---

# Convolution

## 복습
MLP는 입력벡터와 가중치벡터의 내적을 계산 후 활성함수를 적용한 (= 선형변환 + 활성함수), 뉴런이 입력벡터의 모든 성분과 연결된 fully connected 구조. 출력 벡터의 성분 수만큼 가중치 벡터가 필요해서 가중치 행렬이 매우 커진다.

![fully_conn](/assets/images/post/220121/boostcamp_cnn_first_step/fully_conn.png){: .align-center}

$$h_i=\sigma\left(\sum^p_{j=1}W_{ij}x_j\right)$$

## convolution 연산
fully connected 구조와 달리 동일한 커널(가중치)이 입력벡터 상에서 움직여가며 입력벡터의 일부(커널 크기만큼)와 작은 규모의 선형변환 + 활성함수 연산을 반복함.

![conv](/assets/images/post/220121/boostcamp_cnn_first_step/conv.png){: .align-center}

$$h_i=\sigma\left(\sum^k_{j=1}V_jx_{i+j-1}\right), \text{k:커널사이즈}$$

커널의 크기는 고정이고 입력벡터의 모든 조각에 공통적으로 적용되므로 가중치의 수가 매우 적다.

### conv의 수학적 의미
* 신호를 커널을 이용해 국소적으로 증폭 또는 감소시켜 정보를 추출 또는 필터링. 
* CNN에서 사용하는 conv 연산은 convolution이 아니고 cross-correlation이다.
* 커널은 변화하지 않음 (translation invariant)

**영상처리에서 많이 사용된다.**
* 커널에 따라서 이미지에서 경계선 추출, 노이즈 제거 등의 효과를 얻을 수 있다.

### 2차원 conv 연산

$$[f*g](i,j) = \sum_{p,q}f(p,q)g(i+p,j+q)$$

![conv_1ch](/assets/images/post/220121/boostcamp_cnn_first_step/conv_1ch.png){: .align-center}


* 출력의 크기 - 입력 (H, W), 커널 크기 ($K_H$, $K_W$), 출력 크기 ($O_H$, $O_W$) 일 때, 

$$\begin{matrix}O_H=H-K_H+1\\O_W=W-K_W+1\end{matrix}$$

* 채널이 여러개인 2차원 입력은 각 채널마다 별도의 커널(커널 묶음)을 적용한다.

![conv](/assets/images/post/220121/boostcamp_cnn_first_step/conv_multi_ch.png){: .align-center}

* 채널이 여러개인 출력은 커널 묶음이 여러개면 된다.

![conv](/assets/images/post/220121/boostcamp_cnn_first_step/multi_ch_conv.png){: .align-center}

### conv 역전파

![conv_2d](/assets/images/post/220121/boostcamp_cnn_first_step/conv_2d_x_5.png){: .align-center}

그래디언트 역전파에도 conv 연산이 사용된다.

#### dx
![back_prop_dx](/assets/images/post/220121/boostcamp_cnn_first_step/back_prop_dx.png){: .align-center}

$$\frac{\partial\mathcal{L}}{\partial x_5}=\frac{\partial\mathcal{L}}{\partial o_4}w_1+\frac{\partial\mathcal{L}}{\partial o_3}w_2+\frac{\partial\mathcal{L}}{\partial o_2}w_3+\frac{\partial\mathcal{L}}{\partial o_1}w_4$$

$$\vdots$$

$$\frac{\partial\mathcal{L}}{\partial x_1}=\frac{\partial\mathcal{L}}{\partial o_1}w_1$$

#### dw
![back_prop_dw](/assets/images/post/220121/boostcamp_cnn_first_step/back_prop_dw.png){: .align-center}
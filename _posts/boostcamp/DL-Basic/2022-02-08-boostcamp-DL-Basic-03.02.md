---
title: "DL Basic 03.02] Optimization"
excerpt: "최적화 알고리즘 & regularaization"

categories:
  - boostcamp DL Basic
tags:
  - [boostcamp, boostcamp DL Basic]

toc: true
toc_sticky: true

date: 2022-02-08T21:11:00+09:00
last_modified_at: 2022-02-08T21:11:00+09:00
---

# Optimization

## Optimization algorithms

### 경사하강법과 배치 크기

1. Stochastic gradient descent, SGD, 확률적경사하강법
    * 하나의 샘플을 이용해 경사하강법 수행
2. **Mini-batch gradient descent, 미니배치 경사하강법**
    * 데이터셋 일부를 이용해 경사하강법 수행
3. Batch gradient descent, 배치 경사하강법
    * 데이터셋 전체를 이용해 경사하강법 수행

미니배치 경사하강법을 사용하자. 매번 다른 데이터를 이용하므로 local miniaml에 갇히더라도 미니배치에 따라 손실함수 형태가 바뀌며 local miniaml이 아니게될 수 있어 탈출 가능성이 있다. 배치 경사하강법에 비해 데이터를 덜 사용해 연산이 빠르면서도 전체 데이터와 비슷한 방향의 그래디언트를 기대할 수 있다. 배치크기가 너무 작은 경우 너무 잦은 업데이트로 오히려 수렴이 느리다.

#### Batch-size Matters

[paper: ON LARGE-BATCH TRAINING FOR DEEP LEARNING:
GENERALIZATION GAP AND SHARP MINIMA](https://arxiv.org/pdf/1609.04836.pdf){: .btn .btn--info}

![sharp & flat minimul](/assets/images/post/220208/boostcamp-DL-Basic-03.02/sharp_flat_minimul.png){: .align-center}

위 논문에서는 큰 배치사이즈의 경우 날카로운 극소점에 도달하고, 작은 배치사이즈의 경우 완만한 극소점에 도달한다고 한다. 완만한 극소점에 도달하는 경우 일반화 성능이 더 좋다고한다.

### 1. Gradient Descent

$$W_{t+1}\leftarrow W_t-\eta g_t$$ 

* $\eta$: 학습률
* $g_t$: 그래디언트

### 2. Momentum

$$
\begin{matrix}
a_{t+1} \leftarrow \beta a_t + g_t \\
W_{t+1} \leftarrow W_t - \eta a_{t+1} \\
\end{matrix}
$$

* $\beta$: 모멘텀 계수
* $a_t$: 그래디언트 누적 항

이전 미니배치의 그래디언트를 활용. 이전 그래디언트의 크기를 조금 줄이면서 현재 그래디언트와 더해 가중치를 업데이트한다. 가중치 갱신때마다 갱신 방향이 급격하게 바뀌지 않는다. 이전 미니배치의 그래디언트를 활용하여 한번에 더 많은 양의 데이터를 처리하는 것과 비슷한 효과를 가지므로 수렴이 빠르다. 

### 3. Nesterov Accelerated Gradient

![nesterov momentum](/assets/images/post/220208/boostcamp-DL-Basic-03.02/nesterov.png){: .align-center}

$$
\begin{matrix}
a_{t+1} \leftarrow \beta a_t + \nabla\mathcal{L}(W_t-\eta\beta a_t) \\
W_{t+1} \leftarrow W_t - \eta a_{t+1} \\
\end{matrix}
$$

* $\nabla\mathcal{L}(W_t-\eta\beta a_t)$: lookahead gradient


모멘텀 알고리즘의 그래디언트 누적 항 계산에 현재의 그래디언트 대신 가중치를 모멘텀으로 갱신한 후의 그래디언트(lookahead gradient)를 사용하는 알고리즘. (관성과 관성을 통해 이동한 지점에서의 그래디언트를 이용)

극소점 부근에서 모멘텀때문에 수렴이 늦어지는 현상을 완화해준다.

### 4. Adagrad

$$\begin{matrix}
G_t = G_{t-1} + g^2_t\\
W_{t+1} = W_t - \frac{\eta}{\sqrt{G_t+\epsilon}}g_t \\
\end{matrix}$$


* $\epsilon$: 분모가 0이 되지 않도록하는 작은 수
* $G_t$: 그래디언트의 제곱 합 - 가중치 갱신때마다 계속하여 누적하여 점차 커짐

Adaptive learning rate 방식. 가중치 갱신을 반복하며 많이 변한 가중치는 조금 변화시키고 적게 변한 가중치는 많이 변화시킨다. 학습률을 그래디언트 제곱의 합으로 나누어 가중치별로 갱신되는 정도를 조절한다. 가중치 갱신이 이루어질수록 **학습률이 작아지며 결국 학습이 안되는 현상이 발생한다.** ($G_t \rightarrow \infty$)


### 5. Adadelta

$$
\begin{matrix}
G_t=\gamma G_{t-1} + (1-\gamma)g^2_t \\
W_{t+1}=W_t-\frac{\sqrt{H_{t-1}+\epsilon}}{\sqrt{G_t}+\epsilon} \\
H_t = \gamma H_{t-1} + (1-\gamma)(\Delta W_t)^2 \\
\end{matrix}
$$

Adagrad의 학습률이 0이 되는 것을 억제하기 위해 지수이동평균을 통해 그래디언트 제곱을 누적함. (최신 그래디언트가 더 중요한 비중 차지 / 일반 이동 평균은 윈도우 크기만큼의 그래디언트를 보관해야하므로 비효율적)

학습률이 없다.

$H$항이 뭔지 모르겠다. 논문 읽어보고 다시 작성할 것.
{: .notice}

### 6. RMSprop

$$\begin{matrix}
G_t = \gamma G_{t-1} + (1-\gamma)g^2_t\\
W_{t+1} = W_t - \frac{\eta}{\sqrt{G_t+\epsilon}}g_t \\
\end{matrix}$$

Adagrad 알고리즘의 개선판. Adadelta와 마찬가지로 학습률이 0이 되는 것을 방지하기위해 그래디언트 제곱 합을 지수이동평균으로 계산.

### 7. Adam

$$
\begin{matrix}
m_t=\beta_1 m_{t-1} + (1-\beta_1)g_t \\
v_t = \beta_2 v_{t-1} + (1-\beta_2)g_t^2 \\
W_{t+1} = W_t - \frac{\eta}{\sqrt{v_t+\epsilon}}\frac{\sqrt{1-\beta_2^t}}{1-\beta_1^t}m_t
\end{matrix}
$$

Adaptive Moment Estimation. 모멘텀과 adaptive learning rate 방식을 적절히 결합한 방식. 

? 이것도 논문을 읽어보자.
{: .notice}

## Regularization

학습을 방해해서 학습 데이터에 fitting되지 않고 generalization 성능을 높이는 기술

### Early Stopping

오버피팅되기 전에 학습을 중단. 학습 loss와 테스트 loss를 비교하며 학습 중단 시기를 결정하자.

### Parameter Norm Penalty

$$\text{total loss} = \text{loss}(\mathcal{D}; W) + \frac{\alpha}{2}\Vert W\Vert^2_2$$

가중치의 크기가 커지는 것을 방해하는 방법. loss에 가중치의 제곱을 더해 가중치 크기만큼 그래디언트를 증가시킨다. 신경망을 더 부드러운 형태로 만들어 오버피팅을 억제한다.

### Data Augmentation

데이터 수가 적을때는 일반적인 머신러닝 알고리즘이 효율적이지만 데이터가 많으면 많을수록 표현력이 큰 신경망의 성능이 좋다. 데이터는 많으면 많을수록 좋지만 구할 수 있는 데이터는 한정적이므로 보유한 데이터를 회전, 확대 및 축소, crop 등의 방법을 통해 데이터의 수를 늘린다.

### Noise Robustness

데이터 또는 가중치에 랜덤한 노이즈를 추가한다. 왜 잘되는지는 모른다고 한다.

### Label smoothing

![label smoothing](/assets/images/post/220208/boostcamp-DL-Basic-03.02/label_smoothing.png){: .align-center}

출처: [arXiv:1905.04899](https://arxiv.org/pdf/1905.04899.pdf)
{: .text-center}

데이터 샘플을 섞는다. 결정 경계를 부드럽게 만든다는데... 잘 모르겠다.

### Dropout

![dropout](/assets/images/post/220208/boostcamp-DL-Basic-03.02/dropout.png){: .align-center}

forward 과정에서 무작위로 일부 뉴런의 값을 0으로 만든다. 왜 잘되는지 이유를 모르지만 신경망이 좀 더 robust한 feature를 학습한다고 해석한다. 

### Batch Normalization

$$\mu_B = \frac{1}{m}\sum^m_{i=1}x_i$$

$$\sigma^2_B=\frac{1}{m}\sum^m_{i=1}(x_i-\mu_B)^2$$

$$\hat{x}_i=\frac{x_i-\mu_B}{\sqrt{\sigma^2_B+\epsilon}}$$

$$y_i = \gamma \hat{x}_i + \beta$$

학습 과정에서 미니 배치의 평균과 분산을 계산해 미니 배치를 정규화하고, 미니배치의 평균과 분산은 지수이동평균을 통해 누적시키며 보관한다. $\gamma, \beta$항은 학습되는 가중치로 정규화한 미니배치를 scaling, shift 시킨다. 정규화된 미니배치의 평균 0, 분산 1을 적절하게 변환하는 역할인 듯 싶다. 추론 과정에서는 미니배치의 평균, 분산 대신에 학습 과정에서 누적시키며 보관하던 평균, 분산을 이용한다.

논문에서는 internal covariate shift, 레이어별로 입력의 분포가 다른 문제를 해결한다고 하였으나 배치 정규화에 관한 후속 논문들은 동의하지 않는 분위기라고 한다.

배치 정규화 외에도 레이어 정규화, 인스턴스 정규화, 그룹 정규화 등 여러 변형 알고리즘이 나왔다.


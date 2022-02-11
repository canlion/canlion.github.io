---
title: "DL Basic 07] Sequential Model - RNN"
excerpt: "sequential model / RNN"

categories:
  - boostcamp DL Basic
tags:
  - [boostcamp, boostcamp DL Basic]

toc: true
toc_sticky: true

date: 2022-02-09T20:56:00+09:00
last_modified_at: 2022-02-09T20:56:00+09:00
---

# Sequential Model

**sequential data**: 연속된 데이터 (비디오, 음성, 연속된 동작, ...). 길이를 알 수 없다.

## naive sequence model

시퀀스 모델은 연속된 데이터가 입력되면 그 다음 데이터를 예측. $t$가 커질수록 고려해야하는 과거 데이터가 많아진다.

![sequential model](/assets/images/post/220209/boostcamp-DL-Basic-07/sequence_model.png){: .align-center}

## 자기회귀 모델, autoregressive model

현재 $t$ 시점 직전 $\tau$개의 데이터만을 고려하는 모델. $\tau$보다 먼 과거의 데이터는 잃게된다.

![autoregressive model](/assets/images/post/220209/boostcamp-DL-Basic-07/autoregressive.png){: .align-center}

## markov model (first-order autoregressive model)

$t$ 시점의 데이터는 오직 직전 $t-1$ 시점의 데이터에만 의존적이라는 가정을 가진 모델. 많은 과거 데이터를 활용하지 않게 된다. (교수님 비유: 수능 성적은 오직 수능 전날의 공부에 달려있다.)

$$p(x_1, \cdots, x_T)=p(x_T|x_{T-1})p(x_{T-1}|x_{T-2})\cdots p(x_2|x_1)p(x_1)=\prod^T_{t=1}p(x_t|x_{t-1})$$

## 잠재 자기회귀 모델, latent autoregressive model

과거의 데이터를 함축한 hidden state를 활용한다.

![latent autoregressive model](/assets/images/post/220209/boostcamp-DL-Basic-07/latent_autoregressive.png){: .align-center}

$$\hat{x}=p(x_t|h_t)$$
$$h_t=g(h_{t-1}, x_{t-1})$$

# 순환신경망, Recurrent Neural Network

## Vanilla RNN

RNN은 $t$시점의 출력 $h_t$ 계산에 $x_t$와 $t-1$시점의 출력 $h_{t-1}$을 활용한다.
activation으로 tanh 대신 relu를 사용하기도 한다.

![RNN](/assets/images/post/220209/boostcamp-DL-Basic-07/RNN.png){: .align-center}

$$h_t = \text{tanh}(Ux_t + b_{x}+Wh_{t-1}+b_{h})$$

내부에 dense 레이어가 두개가 있다고 볼 수 있다.

```python
rnn = torch.nn.RNN(
    input_size=8,
    hidden_size=16,
    nonlinearity='tanh',
    batch_first=True,
)

# 하나의 모델을 여러번 반복 사용한다. 당연히 모든 시점에서 가중치는 같은 객체.
for name, param in rnn.named_parameters():
    print(f'name: {name}, shape: {param.shape}')
# name: weight_ih_l0, shape: torch.Size([16, 8])
# name: weight_hh_l0, shape: torch.Size([16, 16])
# name: bias_ih_l0, shape: torch.Size([16])
# name: bias_hh_l0, shape: torch.Size([16])

# t=0 hidden state 생성
h_0 = torch.zeros(1, 1, 16)  # 배치크기, rnn 스택 수, hidden state dim=output dim
# 문장으로 예를 들면 5개 단어를 가진 문장 하나. 각 단어는 길이 8의 벡터로 표현된다.
x = torch.rand(1, 5, 8)

out, h_n = rnn(x, h_0)

print(f'output shape: {out.shape}')
# output shape: torch.Size([1, 5, 16])
print(f'final hidden state shape: {h_n.shape}')
# final hidden state shape: torch.Size([1, 1, 16])

# out: 모든 시점의 hidden state 기록
# h_n: 마지막 시점의 hidden state
print(torch.equal(out[0, -1], h_n[0, -1]))
# True
```

### gradient vanishing / exploding

activation으로 tanh, ReLU를 사용하므로 먼 과거의 시점의 그래디언트가 희미해지거나 엄청나게 커지기 쉽다.

### 장기 기억에 약하다. (long term dependencies)

먼 과거의 hidden state는 계속하여 activation이 적용되어 작아지므로 시간이 갈수록 살아남기 힘들어진다. 문장을 해석한다면 문장을 읽어나가면서 점차 문장의 처음을 잊어버리는 것과 같다.

## LSTM, Long Short Term Memory

[출처: C. Olah blog - Understanding LSTM Networks](https://colah.github.io/posts/2015-08-Understanding-LSTMs/){: .btn .btn--info}

RNN의 단점을 보완한 유닛.

![LSTM](https://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-chain.png){: .align-center}
![operation](https://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM2-notation.png){: .align-center}

### cell state

![cell state](https://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-C-line.png){: .align-center}

vanilla RNN의 long term dependencies 문제를 보완하는 코어 아이디어. cell state를 통해 과거의 정보를 전달하며 매 순간 현재 데이터와 이전 hidden state(이전 출력)를 이용해 cell state에 정보를 유지, 추가, 제거한다.

#### forget gate: 과거 정보의 유지, 제거

![forget gate](https://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-f.png){: .align-center}

forget gate는 이전 출력과 현재 데이터를 이용해 cell state의 정보를 유지하거나 제거한다. 시그모이드 $\sigma$는 (0, 1) 범위의 가중치를 생성하고, 이 가중치를 cell state에 pointwise 곱연산하여 어떤 정보를 어떤 강도로 유지할지를 결정한다.

#### input gate: 현재 정보의 추가

![input gate](https://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-i.png){: .align-center}

이전 출력과 현재 데이터를 이용해 cell state에 정보를 추가한다. 시그모이드 $\sigma$의 출력은 forget gate의 가중치와 비슷하게 어떤 정보를 어떤 강도로 cell state에 추가할지를 결정한다. tanh의 출력은 cell state에 추가할 정보들이며 시그모이드의 출력과 곱해져 각 정보의 강도가 정해진다. (가중치가 0이라면 추가되지 않는다.)

#### forget gate & input gate : cell state 업데이트

![update cell state](https://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-C.png){: .align-center}

$$
\begin{matrix}
C_t & = & f_t * C_{t-1} & + & i_t * \tilde{C}_t \\
&&\text{과거 정보 버리기} & & \text{현재 정보 추가하기} \\
\end{matrix}
$$

#### output gate

![update cell state](https://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-o.png){: .align-center}

업데이트된 cell state와 이전 출력, 현재 데이터를 통해 출력을 계산한다. 이번에도 시그모이드 출력은 어떤 정보를 출력할지 결정하는 가중치 역할을 한다.

## GRU, Gated Recurrent Unit

![GRU](/assets/images/post/220209/boostcamp-DL-Basic-07/GRU.png){: .align-center}

LSTM에서 cell state를 제거하고, 게이트 및 파라미터 수를 줄여서 학습 난이도를 낮추고 일반화 성능을 높인 유닛. 과거의 정보는 hidden state를 통해 전달한다.

### reset gate & update gate

$$z_t=\sigma(W_z\cdot[h_{t-1}, x_t])$$
$$r_t=\sigma(W_r\cdot[h_{t-1}, x_t])$$
$$\tilde{h}_t=\text{tanh}(W\cdot[r_t*h_{t-1},x_t])$$
$$h_t=(1-z_t)*h_{t-1}+z_t*\tilde{h}_t$$

hidden state 업데이트시에 과거의 정보와 현재의 정보의 가중치의 합은 1이다.

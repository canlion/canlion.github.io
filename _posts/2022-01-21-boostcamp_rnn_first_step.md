---
title: "AI Math 10 - RNN 첫걸음"
excerpt: "시퀀스 데이터의 개념과 특징, RNN, BPTT, 기울기 소실"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-21 13:38
last_modified_at: 2021-01-21 13:38
---
# RNN

시계열/시퀀스 데이터에 적용하는 네트워크. 데이터 샘플이 독립적이지 않은 경우가 많다.

## 시퀀스 데이터
소리, 문자열, 주가 등 순차적으로 나열된 데이터. (시간 순서에 따라 나열된 데이터: 시계열 데이터). 앞의 데이터가 뒤의 데이터에 영향을 미칠 수 있다. 그러므로 순서가 매우 중요하다.

시퀀스 데이터는 독립동등분포(i.i.d.) 가정을 잘 위배하므로 순서를 바꾸거나 과거 정보의 손실이 있다면 데이터의 확률분포가 변화함. - 손상된 과거의 정보로 미래를 예측하려는 것이므로 원하는 결과를 얻기 힘들다.

### 조건부 확률의 이용
이전 시퀀스의 정보를 통해 앞으로 발생할 데이터의 확률분포를 다룬다.

$$\begin{matrix}
P(X_1,...,X_t) & = & P(X_t\vert X_{t-1},...,X_1)P(X_{t-1},...,X_1) \\
& = & \prod^t_{s=1}P(X_s\vert X_{s-1},...,X_1)
\end{matrix}$$

$$X_t \sim P(X_t\vert X_{t-1},...,X_1)$$

그렇다고 항상 과거의 모든 정보가 필요하지는 않다. (ex - 주가 예측에 수십년전의 주가 정보까지 사용하지는 않는다.)

### 가변적인 데이터를 다룰 수 있는 방법이 필요
조건부 확률의 조건이 되는 데이터 길이가 가변적

$$X_t \sim P(X_t\vert X_{t-1},...,X_1)$$
$$X_{t+1} \sim P(X_{t+1}\vert X_t,X_{t-1},...,X_1)$$

1. 고정된 길이의 정보만 다루자 - 고정된 길이 $\tau$만큼의 데이터만 이용하자.
    * AR($\tau$) - Autoregressive Model, 자기회귀 모델
    * $X_t \sim P(X_t\vert X_{t-1},...,X_{t-\tau})$ <br>
      $X_{t+1} \sim P(X_{t+1}\vert X_t,X_{t-1},...,X_{t-\tau+1})$
    * $\tau$를 사람이 결정해야한다.
2. 직전 정보를 제외한 과거의 정보를 하나의 변수로 인코딩해 이용하자.
    * 잠재 AR 모델 - 이전 정보를 제외한 나머지 정보들을 잠재변수 $H_t$로 인코딩하여 활용
    * $X_t \sim P(X_t\vert X_{t-1},...,X_1) \rightarrow X_t \sim P(X_t\vert X_{t-1}, H_t)$ <br>
      $X_{t+1} \sim P(X_{t+1}\vert X_{t},...,X_1) \rightarrow X_{t+1} \sim P(X_{t+1}\vert X_{t}, H_{t+1})$ 
    * 가변 길이 데이터 문제를 해결하면서도 과거의 모든 정보를 이용할 수 있다.
 
### RNN
잠재 AR 모델. 잠재변수$H$를 신경망을 통해 반복하여 사용하며 시퀀스 데이터의 패턴을 학습.

과거의 정보를 다루기 위해서 이전의 잠재변수와 현재의 입력을 활용하여 모델링.

**$\boldsymbol{W}^{(2)}, \boldsymbol{W}^{(1)}_H, \boldsymbol{W}^{(1)}_X$는 $t$와 무관하다. 모든 $t$에서 같은 가중치를 사용한다.**

![rnn](/assets/images/post/220121/boostcamp_rnn_first_step/rnn.png){: .align-center}

$$\boldsymbol{H}_t=\sigma\left(\boldsymbol{X}_t\boldsymbol{W}^{(1)}_X+\boldsymbol{H}_{t-1}W^{(1)}_H+\boldsymbol{b}^{(1)}\right)$$

$$\boldsymbol{O}_t=\boldsymbol{H}_t\boldsymbol{W}^{(2)}+\boldsymbol{b}^{(2)}$$

이전의 잠재변수 $\boldsymbol{H}_{t-1}$이 다음 순서의 잠재변수 $\boldsymbol{H}_t$의 인코딩에 사용된다.


#### RNN의 역전파 - BPTT, Backpropagation Through Time

출력으로부터의 그래디언트와 다음 잠재변수로부터의 그래디언트를 고려해야한다.

수식 이해가 안된다.. 주말에 다시
{: .notice--primary}

시퀀스의 길이가 길어질수록 그래디언트 전파가 불안정해진다. 1보다 작은 그래디언트가 계속 누적해서 곱해지면 과거 정보에 대한 그래디언트가 0에 가까워지고 학습이 되지않아 과거 정보가 소실되고 (**gradient vanishing**), 1보다 큰 그래디언트가 계속 누적해서 곱해지면 그래디언트가 매우 커진다.

truncated BPTT: 그래디언트 소실을 막기 위해서 RNN을 몇개 블럭으로 쪼개어 블럭간의 경계에서는 미래 시점에서 잠재변수로 전파되어오는 그래디언트를 막고 오직 출력으로부터만 그래디언트를 전파받는다.
![rnn](/assets/images/post/220121/boostcamp_rnn_first_step/truncated_bptt.png){: .align-center}

truncated BPTT가 완전한 해결책은 아니기에 vanilla RNN은 긴 시퀀스를 처리하는데 적합하지 않고 RNN을 개선한 **LSTM**, **GRU**등이 사용된다.
---
title: "DL Basic 01-02] Historical Review / MLP"
excerpt: "딥러닝 소개 & 역사 / 신경망 & MLP"

categories:
  - boostcamp DL Basic
tags:
  - [boostcamp, boostcamp DL Basic]

toc: true
toc_sticky: true

date: 2022-02-07T17:48:00+09:00
last_modified_at: 2022-02-07T17:48:00+09:00
---
<style>
  .box {
    height: auto;
    border: 2px solid black;
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .box-name {
    font-weight: bold;
  }
</style>

<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">

# Historical Review

## DL 공부
* 구현 실력 - pytorch, tensorflow, ... / 아이디어의 구현, 결과 확인
* 수학 - 선형대수학, 확률론, 통계학, 해석학, ...
* 논문 - 최신 연구 트렌드 파악

## 인공지능, 머신러닝, 딥러닝

<div class="box" style="width: 300px;">
  <div class="box-name">Artificial Inteligence</div>
  <div class="box" style="width: 250px;">
    <div class="box-name">Machine Learning</div>
      <div class="box" style="width: 200px;">
        <div class="box-name">Deep Learning</div>
        <br>
        <div>신경망의 사용</div>
      </div>
    <div>데이터를 통한 학습</div>
  </div>  
  <div>사람의 지능 모방</div>
</div>
<br>

**DL이 AI 전체를 의미하는 것은 아니다.**


## DL의 핵심 요소들

1. data
    * 이미지, 말뭉치, 비디오, ...: 내가 풀고자하는 문제에 따라 다르다.
    * 모델은 데이터로부터 학습한다.
2. model
    * 입력을 내가 원하는 출력으로 변환시키는 역할.
    * 같은 데이터, 같은 task에서도 모델 성질에 따라 결과가 다르다.
3. loss
    * 내가 모델에게 바라는 행동을 근사하며 모델을 어떤 방향으로 학습시킬지의 기준이 된다.
      * loss 값을 감소시키는 방향으로 모델을 학습시킨다.
      * ex) MSE: 모델의 출력과 타겟값의 차이의 제곱을 평가한다.
    * 단순히 loss의 감소가 원하는 목적을 이루는 것은 아니다.
      * 너무 loss의 감소에만 주력하면 일반화성능이 떨어지면서 테스트 데이터에 취약해진다.
    * loss의 종류에 따라서 이 loss를 왜 사용하는지, loss의 감소가 문제에 어떻게 작용하는지 등 loss가 문제와 어떤 관계인지 이해해야함.
      * MSE의 경우 noise가 크다면 제곱 연산이 큰 아웃라이어에 크게 반응하면서 학습에 악영향을 미침. 제곱 대신 1-norm이나 robust regression이 가능한 다른 loss를 사용한다.
4. algorithm
    * loss를 감소시키는 알고리즘
      * optimizer - loss를 parameters로 1차 미분한 그래디언트를 이용해 가중치 갱신: SGD
      * regularizer - 학습을 방해하며 모델이 학습데이터에 완벽히 fitting되는 것을 막는다.
    * dropout, early stopping, weight-decay, BN, ...

위의 네가지 요소를 중심으로 논문을 읽으면 이전 연구들과 비교해 연구의 장점과 contribution을 이해하기 좋다.

## 연도별 DL 패러다임
* 2012
  * AlexNet: DL모델 최초의 이미지넷 1위, 패러다임 변화의 시발점.
* 2013
  * DQN: DeepMind, 강화학습(Q-Learning) + DL
* 2014
  * Encoder / Decoder: Neural Machine Translation / 한 언어의 단어 시퀀스를 벡터로 인코딩, 인코딩된 벡터를 다른 언어의 단어 시퀀스로 디코딩 - seq2seq 모델
  * Adam Optimizer: 까다로운 하이퍼파라미터 탐색이 없더라도 웬만하면 학습을 보장하는 옵티마이저.
* 2015
  * Generative Adverszrial Network: GAN, 적대적 생성 신경망 / 데이터를 생성하는 모델과 자연데이터와 인공적으로 생성한 데이터를 구별하는 모델이 경쟁하며 학습.
  * Residual Networks: ResNet / 일반화 성능의 급격한 하락때문에 레이어를 깊게 쌓지 못하던 모델의 깊이 한계를 크게 확장함. Residual skip connection으로 그래디언트 소실을 완화.
* 2017
  * Transformer: **Attention Is All You Need** / multi head attention / NLP 모델이었으나 현재 모든 것을 대체할 것 같은 기세.
* 2018
  * BERT: NLP 모델 / 거대한 일반 텍스트 데이터셋으로 pre-training한 모델을 적은 수의 특정 종류의 텍스트 데이터셋으로 fine-tuning하여 좋은 성능의 모델 생성.
    * ex) 인터넷에서 텍스트를 잡히는대로 다 데이터로 사용하여 모델 학습. 이후 스포츠 뉴스 기사 관련 데이터 fine tuning.
* 2019
  * BIG Language Models: OpenAI **GPT-3** / 매우매우매우 거대한 모델.
* 2020
  * Self Supervised Learning: **SimCLR** / 학습데이터는 한정적이므로 이미지를 컴퓨터가 잘 이해할 수 있는 벡터로 바꾸는 방식을 학습하기 위해(좋은 representation을 얻기 위해) 라벨이 없는 unsupervised 데이터까지 사용하는 방식에 대한 연구.
  * Self Supervised Data Sampling: 고도의 도메인 지식을 이용해 data를 생성.


# Neural Network & MLP

## neural network

[affine 변환 + <span style="color: red;">비선형 변환</span>]을 반복적으로 쌓아 원하는 함수를 모방, 근사하는 것. function approximator.

## linear neural network

### 단순한 선형회귀 모델

* 데이터: $\mathcal{D}=\{ (x_i, y_i) \}^N_{i=1}$
* 모델: $\hat{y}=wx+b$
  * **목적: $x\rightarrow y$ 맵핑을 선형으로 한정짓고 $w, b$를 찾는다.**
* loss: $\mathcal{L}=\frac{1}{N}\sum^N_{i=1}(y_i-\hat{y}_i)^2$
  * 모델이 목적에 적합할수록 loss가 줄어든다.

데이터가 적고, 모델이 선형이며, loss 함수가 볼록(아래로 볼록)하다면 한 번에 loss 값이 최소인 $w, b$를 찾을 수 있겠지만 일반적으로 신경망은 그런 조건을 갖고있지 않다. 경사하강법(가중치 갱신)과 역전파 알고리즘(가중치 그래디언트 계산)을 사용해야한다.

loss를 $w, b$로 편미분하여 loss를 줄이는 방향으로 $w, b$를 갱신한다. 

$$\frac{\partial \mathcal{L}}{\partial w}=-\frac{1}{N}\sum^N_{i=1}-2(y_i-\hat{y}_i)x$$

$$\frac{\partial \mathcal{L}}{\partial b}=-\frac{1}{N}\sum^N_{i=1}-2(y_i-\hat{y}_i)$$

$$w\leftarrow w - \eta\frac{\partial \mathcal{L}}{\partial w}$$

$$b\leftarrow b - \eta\frac{\partial \mathcal{L}}{\partial b}$$

$\eta$는 학습률, step size로 너무 크다면 학습이 안된다. 그래디언트는 매우 **국소적인 정보**이므로 큰 학습률은 잘못된 방향으로 가중치 갱신을 일으킬 수 있다.

## MLP

### affine 변환

입력에 가중치를 곱하고 편향을 더하는 affine 변환은 두 벡터공간 사이의 맵핑으로 볼 수 있다.

### 비선형 변환

레이어를 여럿 쌓아도 중간중간 비선형 변환이 없다면 레이어를 한 층 쌓는 것과 다를 바 없다. 단 한번의 affine 변환만 적용되어 모델의 표현력이 약하다.

![linear mapping](/assets/images/post/220207/boostcamp-DL-Basic-01-02/linear_mapping.png){: .align-center}

맵핑마다 비선형 변환 $\rho$ 가 적용되면 맵핑의 표현력이 극대화되며 레이어를 쌓는 의미가 생긴다.

$$y=\boldsymbol{W}^\top_2\boldsymbol{h}=\boldsymbol{W}^\top_2\rho(\boldsymbol{W}^\top_1\boldsymbol{x})$$

#### 비선형변환의 예

문제에 따라 다른 함수들을 사용하지만 가장 흔히 사용되는 것은 ReLU

* Rectified Linear Unit (ReLU): $\text{max}(0, x)$

![ReLU](/assets/images/post/220207/boostcamp-DL-Basic-01-02/ReLU.png)

* Sigmoid: $\frac{1}{1+e^{-x}}$

![sigmoid](/assets/images/post/220207/boostcamp-DL-Basic-01-02/sigmoid.png)

* Hyperbolic Tangent (tanh): $\frac{e^x-e^{-x}}{e^x+e^{-x}}$

![tanh](/assets/images/post/220207/boostcamp-DL-Basic-01-02/tanh.png)

### Universal Approximators

[paper: **Multilayer Feedforward Networks are Universal Approximators**](https://www.cs.cmu.edu/~epxing/Class/10715/reading/Kornick_et_al.pdf){: .btn .btn--info}

**이론적으로** 히든 레이어가 하나인 신경망으로도 임의의 연속 함수를 근사할 수 있다.: **universal approximation theorem**

다만 세상 어디엔가 그런 신경망이 존재하는거지 우리의 신경망에 그런 능력은 없다. 아무튼 신경망은 표현력이 매우 좋다.

### MLP - Multi-layer Perceptrons

비선형 변환을 포함하는 히든 레이어를 하나 이상 갖는 신경망.

### loss

* 회귀: MSE
* 분류: CE
  * 분류 모델의 출력은 클래스 수 만큼의 원소를 가진 logit. CE는 target에 해당하는 logit만 크게 키운다. (다른 클래스의 logit값보다 크기만 하면 된다.)
* 확률: MLE
  * 사람 이미지를 입력해 나이를 맞추는 모델처럼 확률적인 정보, uncentainty 정보를 이용하고 싶을 경우.

loss 함수가 항상 우리의 목적을 완성하지는 않음. loss의 성질, loss와 풀고자 하는 문제, 데이터와의 관계를 이해해야한다. (ex - MSE와 노이즈/아웃라이어가 매우 큰 데이터)

숙제: 과연 CE는 분류 문제에 최적인 loss인가?
{: .notice--warning}

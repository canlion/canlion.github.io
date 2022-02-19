---
title: "DL Basic 08] Transformer"
excerpt: "Transformer"

categories:
  - boostcamp DL Basic
tags:
  - [boostcamp, boostcamp DL Basic]

toc: true
toc_sticky: true

date: 2022-02-17T21:51:00+09:00
last_modified_at: 2022-02-17T21:51:00+09:00
---

# 참고
1. [딥 러닝을 이용한 자연어 처리 입문 - 트랜스포머](https://wikidocs.net/31379)
2. [Jay Alammar - The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)
3. [pytorch로 구현하는 Transformer](https://cpm0722.github.io/pytorch-implementation/transformer)

# Transformer

![transformer](/assets/images/post/220218/boostcamp-DL-Basic-08/transformer.png){: .align-center}

transformer는 attention을 이용하는 seq2seq 모델로 NMT 문제에서 좋은 성능을 냈고 특히 인코더 부분은 NMT 문제에만 한정되지 않고 다양한 분야에서 사용된다.

이전까지는 attention이 RNN의 마지막 hidden state에서의 정보 손실을 보조하는 역할이었는데 transformer에서는 RNN 유닛이 사라지고 attention이 핵심 역할을 한다. 그래서 논문 제목이 Attention is all you need인가?

**Attention**:
* 시퀀스 데이터를 다루는 과정에서 과거 시점의 시퀀스 아이템들 중 현재 시퀀스 아이템과 관련된 아이템들을 선별하고 이용하는 기술
* ex - 어떤 문장을 다른 언어로 번역할 때 현재 처리중인 단어와 관련된 다른 단어들을 선별하고 이용해 전체 문맥에 맞는 번역을 찾는다.
  * 나는 고기를 좋아한다. 하지만 '그것'을 다량 섭취하는 것은 건강에 좋지 않다. -> '그것'을 앞문장의 '고기'와 연결시켜야함. attention은 '그것'이라는 단어를 처리하는 시점에서 '고기'의 연관성을 높게 판단한다.

## encoders

![encoder](/assets/images/post/220218/boostcamp-DL-Basic-08/encoder.png){: .align-center}

다수의 인코더가 중첩된 구조. 크게 multi-head self-attention 블럭과 FFN 블럭으로 구성된다. RNN 인코더는 시퀀스 아이템들에 재귀적으로 동작했으나 어텐션을 이용한 인코더는 모든 시퀀스 아이템을 한 번에 인코딩한다.

### multi-head self-attention 블럭

입력된 시퀀스의 아이템들의 어텐션을 계산한다. 

어텐션 계산 예제 - 'I'의 어텐션 계산
1. I, am, a, student의 Query, Key, Value를 생성한다.
2. I의 Query와 I, am, a, student의 Key를 내적해 유사도를 계산한다.
3. 유사도를 softmax한 값을 가중치로하여 모든 단어의 Value를 가중합한다.

**Query와 Key를 통해 단어간의 연관성을 파악하고 중요도가 높은 단어들의 Value에 높은 가중치를 다음 연산에 이용한다(모든 단어끼리 상호작용이 존재).**

![calculate attention value](/assets/images/post/220218/boostcamp-DL-Basic-08/calculate_attention_value.png){: .align-center}

multi-head는 attention value를 한 개가 아닌 여러 개 계산한다는 것으로 병렬적으로 더 풍부한 feature를 얻는 것이 목적이다. 당연히 attention value도 여러 개가 계산되는데 다음 인코더로 넘겨주기위해 입력과 같은 크기로 줄여줘야한다. 모든 attention value를 이어붙이고 dense 연산으로 입력과 같은 shape로 만들어준다.

![encoder output](/assets/images/post/220218/boostcamp-DL-Basic-08/encoder_output.png){: .align-center}

### FFN 블럭
단순한 feed forward network로 두 개의 dense 레이어를 갖고 중간에 ReLU를 activation 함수로 사용한다. 첫 dense 레이어의 출력 수는 변하지만 마지막 레이어의 출력은 입력과 같은 shape를 갖는다.

**FFN 블럭에서는 Attention 블럭과는 다르게 단어간의 상호작용이 존재하지 않는다.**

### residual connection & layer normalization

* residual connection: 그래디언트 소실 완화
* layer normalization: 입력의 마지막 dim에 대해서 정규화. transformer의 경우 각 단어마다 정규화를 수행한다. (CNN의 경우 배치의 각 이미지별로 정규화)

## decoders

![decoder](/assets/images/post/220218/boostcamp-DL-Basic-08/decoder.png){: .align-center}

다수의 디코더가 중첩된 구조. 디코더는 인코더와 같은 구조에 multi-head attention 레이어가 추가됬다. attetion 계산에 인코더의 출력이 관여하므로 self 라는 표현이 빠졌다.

### masked multi-head self-attention 블럭

왜 mask가 붙었는가? 마스킹을 하기 때문이다. 🤗

시퀀스 모델은 이전 출력을 입력으로 사용하여 다음 출력을 계산한다. 그런데 모델이 학습을 시작한 직후라면 제대로된 출력을 기대할 수 없고, 이상한 출력을 다음 출력 계산을 위한 입력으로 사용하는 것도 이상하다. 여기서 Teacher Forcing이라는 개념을 사용한다.

**Teacher Forcing**: label을 디코더의 이전 출력으로 취급하여 학습에 사용한다. 
![teacher forcing](/assets/images/post/220218/boostcamp-DL-Basic-08/teacher_forcing.png){: .align-center}

인코더와 마찬가지로 디코더도 모든 입력을 한 번에 받는다. 그러므로 Teacher Forcing을 이용한 학습 과정에서 attention value 계산시에 미래의 단어를 참고할 수 있다. 오직 과거의 단어만을 참고하도록 미래의 단어에 관련된 attetion score를 매우 작은 값으로 마스킹하여 softmax 계산 후에 0에 가까운 attetion weight를 갖도록 한다.

![decoder masking](/assets/images/post/220218/boostcamp-DL-Basic-08/decoder_masking.png){: .align-center}

그 외의 동작은 인코더의 self-attention 블럭과 같다.

### multi-head attetion / encoder-decoder attention 블럭

attetion 계산에 디코더 외부에서의 입력인 인코더의 출력을 함께 사용하기 때문에 이름에서 self가 빠졌다.

디코더의 self-attention value에서 Query를 생성하고 인코더의 출력에서 Key, Value를 생성하여 attention value를 계산한다.

![encoder-decoder attention](/assets/images/post/220218/boostcamp-DL-Basic-08/encoder_decoder_attention.png){: .align-center}

### 출력

classifier 모델처럼 Dense 레이어 - softmax 레이어를 통해 총 단어수만큼의 길이를 갖는 벡터를 출력하여 각 단어의 확률값으로 사용한다.

## positional encoding

RNN의 경우 시퀀스 아이템의 순서에 따라 순차적으로 연산이 진행되지만 attention 계산 과정은 어순을 나타낼 수 있는 요소가 없다. (그나마 attention weight까지는 열의 순서가 단어의 순서였으나 Value의 가중합을 하면서 순서가 사라진다.)

그래서 시퀀스 아이템의 위치를 나타낼 수 있는 값을 워드 임베딩에 더해준다.

![positional encoding](/assets/images/post/220218/boostcamp-DL-Basic-08/transformer_positional_encoding_large_example_jalammar.png){: .align-center}

출처: [The Illustrated Transformer - Jay Alammar](https://jalammar.github.io/illustrated-transformer/)
{: .text-center}

입력 시퀀스의 길이가 길어지면 보간해서 사용한다.

Q: 더하면 워드 임베딩의 의미가 달라지지 않을까? - 포지셔널 인코딩을 더하니 다른 단어의 임베딩과 같아진다던가
{: .notice}

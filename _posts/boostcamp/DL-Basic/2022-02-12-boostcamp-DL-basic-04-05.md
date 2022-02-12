---
title: "DL Basic 04-05] CNN"
excerpt: "CNN"

categories:
  - boostcamp DL Basic
tags:
  - [boostcamp, boostcamp DL Basic]

toc: true
toc_sticky: true

date: 2022-02-12T23:01:00+09:00
last_modified_at: 2022-02-12T23:01:00+09:00
---

# Convolutional Neural Networks

대부분의 내용은 이전 수업과 겹치니 생략.

## CNN

일반적인 CNN은 convolution 레이어, 풀링 레이어, fully-connected(dense) 레이어로 구성.

* conv. layer + pooling layer: feature extraction 역할
* dense layer: decision making
  * CNN 가중치 수의 대부분을 차지하며 일반화 성능을 떨어트리는 주된 원인.

초기의 CNN은 다수의 dense 레이어를 포함했으나 점차 그 수를 줄여 가중치의 수는 줄이고, conv 레이어의 수를 늘려 층을 깊게 쌓는 방향으로 발전한다. (conv 레이어는 filter의 가중치를 입력의 모든 영역에 적용하는 shared parameters 특성으로 가중치의 수가 적다.)

모델 아키텍쳐를 보고 가중치 수를 대충 파악하는 감을 쌓는 것이 좋다. 

## 1 x 1 convolution

![1x1 ch reduction](/assets/images/post/220212/boostcamp-DL-Basic-04-05/1x1_ch_reduction.png){: .align-center}

1x1 크기의 필터를 통해 이미지의 영역을 보지 않고 채널만 줄이는 conv.

* 차원 축소 역할
  * 모델의 레이어를 많이 쌓을 때 가중치의 수를 감소시키기 위해서 사용한다.
  * ex) bottleneck 아키텍쳐

# Modern CNN

## 요약

* VGG: 3x3 필터 사용 - receptive field를 유지하면서 가중치 수 감소
* GoogLeNet: 1x1 필터 사용 - 차원 축소를 통한 가중치 수 감소
* ResNet: 스킵 커넥션을 통한 그래디언트 소실 문제 완화, 모델의 깊이 제한을 크게 확장
* DenseNet: 이전 레이어의 activation들을 concaternation하여 다음 레이어에 입력

## AlexNet

ILSVRC에서 우승한 첫 딥러닝 모델. 머신러닝의 패러다임을 바꿨다. 굉장히 큰 필터를 사용하여 가중치가 많다.

* ReLU 사용 - 비선형이면서도 연산이 쉽고 빠름. 그래디언트 소실 문제 해결.
* GPU의 사용
* data augmentation
* dropout

지금보면 당연한 아이디어들이다. 즉, 표준을 세운 모델.

## VGGNet

3x3 필터를 사용하여 레이어 수를 늘리며서도 가중치의 수가 적다. dense 레이어를 1x1 conv로 대체함. 

### 3x3 필터와 5x5 필터의 비교

![3x3 vs 5x5](/assets/images/post/220212/boostcamp-DL-Basic-04-05/3x3_5x5.png){: .align-center}

receptive field는 하나의 conv 값을 얻을 때 고려되는 입력의 spatial dimension으로 크게 키울수록 global한 context를 파악하는데 도움이 된다. 3x3 conv 레이어를 반복적으로 쌓으면 큰 필터를 갖는 conv 레이어와 같은 수준의 receptive field 크기를 얻으면서도 가중치 수를 적게 유지할 수 있다.

### GoogLeNet

![inception module](/assets/images/post/220212/boostcamp-DL-Basic-04-05/inception_module.png){: .align-center}

출처: [arXiv:1409.4842](https://arxiv.org/pdf/1409.4842.pdf)
{: .text-center}

네트워크 안에 작은 네트워크(inception 모듈)를 포함하는 Network-in-network (NiN) 구조. 인셉션 모듈에서 하나의 feature에 여러 방식의 연산을 적용해 다양한 feature를 얻어 concat. 특히 1x1 conv를 적극 이용하여 레이어 수가 많음에도 가중치의 수를 적게 유지한다. 

질문: 채널을 줄였다가 다시 확장하면서 생기는 손실은 없는지 / 가중치를 줄이면서 생기는 문제는 없는지
{: .notice}

### ResNet

레이어가 깊으면 깊을수록 학습이 힘들다.

가중치가 매우 많은 모델은 오버피팅이 일어나기 쉽다. 그런데 레이어의 수가 아주 많은 모델의 경우 학습 loss와 테스트 loss가 수렴한 상태에서 레이어가 상대적으로 적은 모델보다 loss가 큰 현상이 발생한다. 즉, 깊은 모델이 학습이 덜 되었다.

![deeper net higher error](/assets/images/post/220212/boostcamp-DL-Basic-04-05/deeper_net_higher_error.png){: .align-center}

출처: [arXiv:1512.03385](https://arxiv.org/pdf/1512.03385.pdf)
{: .text-center}

ResNet에서는 이런 문제를 **skip connection**을 도입해 완화한다.

![skip connection](/assets/images/post/220212/boostcamp-DL-Basic-04-05/skip_connection.png){: .align-center}

`Residual Block` / 출처: [arXiv:1512.03385](https://arxiv.org/pdf/1512.03385.pdf)
{: .text-center}

이상적인 출력을 $\mathcal{H}(\boldsymbol{x})$라 하면, 기존의 conv 레이어의 연속은 $\mathcal{H}(\boldsymbol{x})$를 학습하는 것이 목표이고, conv 레이어에 skip connection을 추가한 residual block은 $\mathcal{F}(\boldsymbol{x}):=\mathcal{H}(\boldsymbol{x})-\boldsymbol{x}$를 학습하는 것이 목표.

논문에서는 residual block은 $\boldsymbol{x}$에 더해줄 정보 $\mathcal{F}(\boldsymbol{x})$만 학습하므로 $\mathcal{H}(\boldsymbol{x})$를 학습하는 것보다 쉽다고 가정한다. (특히 identity mapping을 생각하면 conv 레이어의 가중치를 0으로 밀어버리면 간단하게 학습이 가능하다.)

그래디언트가 skip connection을 통해 그래디언트가 흐르며 최소한의 값을 보장해 그래디언트 소실 문제를 해결한다.


### DenseNet

ResNet에서 skip connection을 통해 feature들을 합했지만 DenseNet에서는 feature들을 concat한다. 웬만하면 좋은 성능을 얻을 수 있다.

#### DenseBlock

![dense block](/assets/images/post/220212/boostcamp-DL-Basic-04-05/dense_block.png){: .align-center}

이전 레이어의 feature를 모두 concaternate. 채널이 빠르게 증가한다.

#### Transition Block

`BatchNorm`$\rightarrow$`conv`$\rightarrow$`2x2 AvgPooling`

채널이 빠르게 증가하기 때문에 1x1 conv로 채널 축소를 수행한다.
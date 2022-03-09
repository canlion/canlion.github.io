---
title: "CV] image classification"
excerpt: "CV, image classification"

categories:
  - boostcamp CV
tags:
  - [boostcamp, boostcamp CV]

toc: true
toc_sticky: true

date: 2022-03-09T14:32:00+09:00
last_modified_at: 2022-03-09T14:32:00+09:00
---

\# CV, image classification

# CV

## 지각 능력의 중요성

AI 개발의 레퍼런스는 사람의 지능. 어린 아이의 발달을 생각하면 지능의 발달은 지각 능력의 획득이 첫 걸음이다. 지각 능력을 통해 세상과 상호작용하며 데이터를 얻고(관찰), 그 데이터들을 통해 세상을 이해(인과관계의 이해)한다. 그리고 인간은 특히 시지각에 매우 의존한다.

시각, 청각, 촉각, 미각, 후각 등의 오감 외에도 그런 감각들이 섞여 복합적인 감각, 표정-말투 등을 느끼는 사회적 지각으로부터도 데이터를 얻는다.

AI 개발에서 데이터는 지각을 통해 얻어지므로 지각 능력의 구현은 매우 중요하다.

## Computer Vision?

사람이 눈으로 세상을 보고 사고를 통해 해석하는 것처럼 기계는 카메라를 통해 이미지를 얻고 HW, 알고리즘을 통해 이미지를 해석한다. 여기서 기계가 해석한 결과를 **representation**이라 한다.

![representation](/assets/images/post/220309/boostcamp-CV-1/representation.jpg){: .align-center}

* Computer Graphics (Rendering): representation를 통해 이미지를 생성
* **Computer Vision** (Inverse Rendering): 이미지에서 representation 추출

cv는 이미지나 영상을 입력받아 색상이나 움직임, 공간감, 인간의 감정 등 다양한 representation을 추출하는 것과 더불어 사람의 시지각 능력을 연구하고 이해하는 것까지 포함한다.

사람의 시지각은 완전하지 않다. 자라면서 주로 보는 것에 익숙해지며 바이어스가 생긴다 (우리는 ). 그러므로 기계의 시지각을 개발하려면 사람의 시지각을 연구하며 장단점을 파악해 불완전함을 개선해야한다.

## CV와 deep learning

![ML vs DL](/assets/images/post/220309/boostcamp-CV-1/MLvsDL.jpg){: .align-center}

과거의 CV는 사람이 직접 데이터를 분석하여 feature를 추출하는 알고리즘을 작성하였고, 그 알고리즘을 통해 추출한 feature를 이용했다. DL이 발달한 이후에는 모델이 데이터로부터 feature를 추출하는 능력을 학습한다. 사람은 편견을 갖고 있고, 데이터에는 사람이 발견하지 못하는 패턴들이 존재할 수 있다. 그러므로 편견없이 데이터로부터 직접 배우는 DL이 매우 강력한 성능을 보여준다.


# 이미지 분류, image classification

이미지를 특정 카테고리로 분류하는 task

**classifier**: 이미지를 입력받아 특정 카테고리를 출력하는 맵핑

만약 세상의, 가능한 모든 데이터를 보유한다면 K-Nearest Neighbors(KNN) 메소드로 분류 문제는 간단히 풀린다. 모든 데이터를 뒤져서 입력된 이미지와 비슷한 이미지들을 찾아서 카테고리를 내면 된다.

다만 그 정도의 데이터를 얻는 것은 불가능하다. 데이터를 모았다고해도 무한대에 가까운 양의 데이터를 처리하는 시간 복잡도, 공간 복잡도도 마찬가지로 무한대이므로 사용이 불가능하다.

결국 가능하면 많은 양의 데이터를 모아서 보유한 데이터에서 잘 해봐야한다. 여기서 딥러닝이 매우 강력하다. 딥러닝은 보유한 데이터를 신경망에 압축시킨다.

## Convolutional Neural Networks, CNN

### fc 레이어로 이미지를 처리한다면?
![average images](/assets/images/post/220309/boostcamp-CV-1/avg_imgs.png){: .align-center}

출처: [CS 231n lecture5 slide](http://cs231n.stanford.edu/slides/2021/lecture_5.pdf)
{: .text-center}

위 이미지는 fc 레이어로 이루어진 간단한 분류기의 가중치 $w$에 대응되는 각 픽셀이 각 클래스에서 어떤 영향을 갖는지를 시각화한 이미지(가중치가 학습한 각 클래스에 대한 템플릿 정도로 이해). 각 클래스의 대표적인 형태로 이해한다면 이것들과 다른 형태의 이미지가 입력되면 제대로 대응하지 못하게된다.

### Covolution 레이어

conv 레이어는 
* local feature learning: 국소적으로 작은 영역만을 입력으로 받아 출력을 계산한다. 작은 영역만 때서 계산하므로 입력은 공간적 특성을 유지한다.
* sliding window & parameter sharing: 이미지의 모든 영역에 위의 연산을 똑같이 반복하여 가중치 수가 훨씬 적다.

fc 레이어는 이미지 내의 오브젝트의 위치가 바뀐다면 전혀 새로운 데이터로 인식하지만 conv 레이어는 국소 영역에 대한 연산을 반복하는 방식이므로 오브젝트 위치가 바뀌더라도 유연하게 대응이 가능하다.

### CNN

conv 레이어로 구성된 신경망인 CNN은 여러 CV task의 backbone으로 활용된다. 이미지에서 feature를 추출하는 성능이 탁월하다.

# CNN architecture for image classification 1

* AlexNet - 2012 / game changer
* VGGNet - 2014 / 더 깊으면서도 간단해진 구조
* GoogLeNet - 2015 / 더더 깊으면서도 연산 효율을 높인 구조
* ResNet - 2016 / residual connection을 통한 매우 깊은 네트워크의 문제 완화
* DenseNet, SENet, EfficientNet, ...

## AlexNet

이미지넷에서 우승한 첫 CNN이자 딥러닝의 시대를 연 모델

* ReLU
* Dropout
* 당시 GPU 스펙이 낮아 두 장의 GPU에서 분산학습
* Local Response Normalization이 이용되었으나 이제는 사용되지 않음
* conv의 커널 크기가 상당히 큰 편

## VGGNet

간단한 구조로 지금까지도 baseline, backbone으로 사용된다.

* 더 깊어진 구조 - 16, 19층
* 간단하다. 3x3 conv와 2x2 max pooling만 쌓았다.
* fine-tuning 없이도 feature extractor로 잘 작동한다. 좋은 일반화 성능.

### 3x3 conv만 쌓았다.
* 커널 크기를 줄이더라도 많이 쌓으면 **가중치 수를 줄이면서도** receptive field를 동일하게 확보할 수 있다.
* 더 많이 쌓는 만큼 더 많은 activation function을 적용 - **비선형성이 증가**하며 더 복잡한 함수를 근사할 수 있다.


# 더 깊게 깊게

일반적으로 CNN은 깊어질수록 성능이 향상된다. 큰 receptive field로 넓은 범위를 context로 취할 수 있으며 비선형성이 커지며 더 복잡한 관계를 학습할 수 있다.

다만 문제가 발생한다.

## 최적화가 어렵다.

* gradient vanishing / exploding
  * 역전파를 통해 낮은 층의 레이어에 전달된 그래디언트가 너무 작거나 너무 크면 모델이 학습이 안되거나 망가져버린다.
* 연산량이 많고, 오버피팅 위험도 높아진다.

### degradation problem

![degradation problem](/assets/images/post/220212/boostcamp-DL-Basic-04-05/deeper_net_higher_error.png){: .align-center}

출처: [arXiv:1512.03385](https://arxiv.org/pdf/1512.03385.pdf)
{: .text-center}

일반적으로 큰 모델이 작은 모델보다 성능이 높다. 그런데 위 그래프에서는 56층의 모델이 20층의 모델보다 train, test 에러가 모두 높다. 이렇게 레이어가 매우 많은 모델이 상대적으로 레이어가 적은 모델보다 학습조차 제대로 못하는 현상이 degradation 문제다.

gradient vanishing이 주요 원인으로 꼽힌다.

# CNN architecture for image classification 2

## GoogLeNet

### inception module

![inception module](/assets/images/post/220309/boostcamp-CV-1/inception_module.jpg){: .align-center}

동일한 입력에 대해 여러 사이즈의 커널로 conv 연산을 수행하여 다양한 feature를 얻는다.

bottleneck 구조: conv 연산이 많다는 점에서 연산량과 메모리 필요량이 매우 커지므로 **1x1 conv를 이용해 채널 차원을 줄여 연산량을 조절한다.**

**auxiliary classifiers**: gradient vanishing을 해결하기 위해 네트워크 중간의 activation을 이용해 출력을 계산하고 역전파를 진행해 gradient를 주입해준다. (학습 과정에서만 이용된다.)

![googlenet](/assets/images/post/220309/boostcamp-CV-1/googlenet.png){: .align-center}

출처: [arXiv:1409.4842](https://arxiv.org/pdf/1409.4842.pdf)
{: .text-center}

## ResNet

아직도 baseline으로 많이 사용되는 영향력 높은 모델. residual skip connection을 도입해 degradation 문제를 완화하며 네트워크의 레이어 수 한계를 크게 확장했다.

### Residual block

![skip connection](/assets/images/post/220212/boostcamp-DL-Basic-04-05/skip_connection.png){: .align-center}

출처: [arXiv:1512.03385](https://arxiv.org/pdf/1512.03385.pdf)
{: .text-center}

이상적인 출력을 $\mathcal{H}(\boldsymbol{x})$라 하면, 기존의 conv 레이어의 연속은 $\mathcal{H}(\boldsymbol{x})$를 학습하는 것이 목표이고, conv 레이어에 skip connection을 추가한 residual block은 $\mathcal{F}(\boldsymbol{x}):=\mathcal{H}(\boldsymbol{x})-\boldsymbol{x}$를 학습하는 것이 목표.

$\mathcal{H}(\boldsymbol{x})=\boldsymbol{x}+\mathcal{F}(\boldsymbol{x})$이라 가정하고 직접 $\mathcal{H}(\boldsymbol{x})$를 학습하기보다는 $\boldsymbol{x}$를 보존하려는 노력은 residual skip connection으로 대신하고 $\mathcal{F}(\boldsymbol{x})$만 학습하는 것이 쉽다는 전제.

### skip connection

![skip connection 2](/assets/images/post/220309/boostcamp-CV-1/skip_connection.jpg){: .align-center}

gradient가 skip connection을 통해 conv 레이어를 건너뛰고 전달될 수 있어 gradient vanishing을 해결할 수 있다.

#### 가설 - 나중에 읽어보자

![residual block unraveled view](/assets/images/post/220309/boostcamp-CV-1/residual_block_unraveled_view.png){: .align-center}

출처: [arXiv:1605.06431](https://arxiv.org/pdf/1605.06431.pdf)
{: .text-center}

residual skip이 gradient vanishing을 해결해서 성능이 좋은 것이 아니라 다양한 경로를 만들면서 여러 경로가 앙상블되는 효과덕에 성능이 좋다는 논문.

### 구조

* residual skip을 통해 입력이 conv 출력과 더해지므로 다음 conv 레이어의 입력값의 크기가 훨씬 커질 수 있다. 그 점을 고려한 He 가중치 초기화를 이용.
* input->Conv-BN-ReLU-Conv-BN-add input-ReLU->output
* 스테이지를 거치며 채널 수가 두 배, feature 맵은 절반이 되므로 각 스테이지의 첫 residual block에서는 입력을 downsampling하고 채널을 맞춰주는 과정이 필요.
  1. pooling + ch 방향 0 padding 또는
  2. stride 2의 1x1 conv
*

## Resnet 이후...

### DenseNet

#### dense block

![dense block](/assets/images/post/220212/boostcamp-DL-Basic-04-05/dense_block.png){: .align-center}

입력과 모든 conv 출력을 다음 conv 출력에 concat (더하지 않고 concat하므로 feature가 보존된다).

#### transition block

`BatchNorm`$\rightarrow$`1x1 conv`$\rightarrow$`2x2 AvgPooling`

dense block에서 채널수가 빠르게 증가하므로 1x1 conv로 채널 축소를 수행.

### SENet

![se block](/assets/images/post/220309/boostcamp-CV-1/se_block.png){: .align-center}

출처: [arXiv:1709.01507](https://arxiv.org/pdf/1709.01507.pdf)
{: .text-center}

channel attention. 채널간의 관계를 모델링하고 중요성을 평가하여 채널에 가중치(attention score)를 부여한다.

* squeeze path: average pooling을 통해 채널 분포 계산
* excitation path: fc 레이어를 통해 채널마다 가중치를 계산

### EfficientNet

#### compound scaling method

![efficientnet performance](/assets/images/post/220309/boostcamp-CV-1/efficientnet_performance.png){: .align-center}

출처: [arXiv:1905.11946](https://arxiv.org/pdf/1905.11946.pdf)
{: .text-center}

보통 모델 아키텍쳐를 스케일링하면 깊이(레이어 수), 너비(채널 수), 입력 사이즈 중 하나만 조절한다. 이 세 가지 요소는 성능 증가가 수렴하는 한계가 다르다. 그러므로 적절한 비율로 세 가지 요소를 동시에 스케일링하면 동일한 연산량 제한에서 더 좋은 성능을 낼 수 있다. 

### deformable convolution

![deformable conv](/assets/images/post/220309/boostcamp-CV-1/deformable_conv.png){: .align-center}

출처: [arXiv:1703.06211v3](https://arxiv.org/pdf/1703.06211v3.pdf)
{: .text-center}

자동차같이 고정된 형태가 있는 오브젝트외의, 사람의 팔다리처럼 다양한 형태 변화가 가능한 오브젝트를 포착하기위한 convolution 연산. 일반적인 사각형 영역 대신 offset을 통해 conv 연산 영역을 지정한다.

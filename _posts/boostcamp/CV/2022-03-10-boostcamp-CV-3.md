---
title: "CV] Semantic Segmentation"
excerpt: "픽셀단위로 분류하기"

categories:
  - boostcamp CV
tags:
  - [boostcamp, boostcamp CV]

toc: true
toc_sticky: true

date: 2022-03-10T20:10:00+09:00
last_modified_at: 2022-03-10T20:10:00+09:00
---

# Semantic segmentation

![semantic segmentation](/assets/images/post/220310/boostcamp-CV-3/semantic_segmentation.png){: .align-center}


출처: [arXiv:1706.05587](https://arxiv.org/pdf/1706.05587.pdf)
{: .text-center}

이미지를 픽셀 단위로 분류하는 task. 인스턴스 단위는 고려하지 않고 클래스, 카테고리 단위만 고려하므로 같은 카테고리의 오브젝트는 한 덩어리로 구분된다.

영상 편집, 의료 영상 처리, 자율주행 시스템 등에 이용된다.


# Semantic segmentation architectures

## Fully Convolutional Networks, FCN

![fcn architecture](/assets/images/post/220310/boostcamp-CV-3/fcn_architecture.png){: .align-center}


출처: [arXiv:1411.4038](https://arxiv.org/pdf/1411.4038.pdf)
{: .text-center}

첫 end-to-end semantic segmentation 모델. classification 모델 구조에서 마지막 flatten + fc 레이어를 1x1 conv로 대체하여 입력 사이즈에 제한이 없으며 입력의 사이즈에 비례하는 사이즈의 heatmap, segmentation 결과를 출력할 수 있다.

backbone에서 수차례 다운샘플링이 일어나 출력이 작고 거칠기 때문에 업샘플링이 필요하다. (backbone에서의 다운샘플링은 큰 receptive field를 통한 context 획득과 연산량 조절 역할)

### fully connected vs fully convolutional

![compare fc](/assets/images/post/220310/boostcamp-CV-3/compare_two_fc.png){: .align-center}


출처: [arXiv:1411.4038](https://arxiv.org/pdf/1411.4038.pdf)
{: .text-center}

![compare fc](/assets/images/post/220310/boostcamp-CV-3/compare_two_fc_2.jpg){: .align-center}

* fully connected layer:
  * 고정된 크기의 벡터 입력, 고정된 크기의 출력
  * 텐서를 벡터로 flatten하여 입력하므로 공간적인 정보가 제거된다. 
* fully **convolutional** layer:
  * 텐서의 각 위치에서 채널방향으로 flatten하여 fc 연산
  * 입력과 출력 크기 제한이 없다.
  * 공간정보를 유지하며 1x1 conv 연산과 같다.
  * 출력이 activation map - segmentation이 가능

### upsampling

작은 activation map을 확대하기 위한 방법

* unpooling
* **transposed convolution**
* **upsample and comvolution**

#### Transposed convolution

![transposed conv](/assets/images/post/220310/boostcamp-CV-3/transposed_conv.jpg){: .align-center}

conv 연산을 거꾸로 하듯이 하나의 값을 커널에 곱해 일정한 간격으로 큰 영역에 찍어내는 연산이다.

**overlap issue**: 커널 크기와 stride가 맞지않아 겹쳐지는 부분이 있다면 규칙적으로 큰 값이 생기며 이미지에 체스판처럼 규칙적인 패턴이 생긴다.

#### interpolation(upsampling) + conv

upsampling에 보간을 사용한 후 conv 연산을 진행하는 방식. 오버랩 이슈가 없이 잘 작동하므로 주로 사용된다.


### low-level feature map의 활용

FCN의 입력은 backbone을 거치며 수차례 다운샘플링된다. 최종 출력의 한 픽셀은 receptive field 만큼의 영역을 대표하여 세그멘테이션이 디테일한 물체 경계를 잡아낼 수가 없다.

이 문제를 해결하기위해 skip connection을 이용한다.

![fcn fuse features](/assets/images/post/220310/boostcamp-CV-3/fcn_fuse_features.png){: .align-center}

![fcn fuse features2](/assets/images/post/220310/boostcamp-CV-3/fcn_fuse_features_2.png){: .align-center}


출처: [arXiv:1411.4038](https://arxiv.org/pdf/1411.4038.pdf)
{: .text-center}

|  저층 activation | 고층 activation   |
|---|---|
| 고해상도(디테일함) | 저해상도(뭉뚱그림) |
| 좁은 범위에 대한 정보 | 넓은 범위에 대한 정보 |
| low-level | high-level (더 추상적) |

저층의 activation map은 low-level feature를 담고 receptive field가 작아 '의미'에 대한 정보는 적지만 고해상도이므로 물체 경계를 잡아내는 것에 유리하고, 고층의 activation map은 high-level feature를 담아 물체의 클래스를 구분하는 것에는 유리하다.

그러므로 저층의 activation map을 활용하여 디테일을 살린다. 위 그림에서처럼 저층 activation map에 1x1 conv를 적용해서 segmentation 출력에 더하고 업샘플링을 적용한다.


## U-Net

![u-net architecture](/assets/images/post/220310/boostcamp-CV-3/unet_architecture.png){: .align-center}


출처: [arXiv:1505.04597](https://arxiv.org/pdf/1505.04597.pdf)
{: .text-center}


의료 영상 분석같은 디테일이 중요한 연구분야에서 주로 이용된다. FCN과 비슷한 구조를 가지고 있고 low-level feature와 high-level feature를 더 효과적으로 융합하여 매우 정교한 세그멘테이션 결과를 얻을 수 있다.

### Contracting path

encoding path로 각 스테이지마다 x1/2 다운샘플링 & 채널 x2

### expanding path

* decoding path로 각 스테이지마다 x2 업샘플링 & 채널 x1/2
* 각 스테이지의 입력에 contracting path의 low-level feature를 concat한다.
  * low-level feature는 고해상도, 작은 receptive field로 입력의 변화에 민감하여 오브젝트간 경계처럼 공간적으로 중요한 정보를 전달해준다.
  * FCN은 low-level feature를 합치되 따로 conv연산 후 더했고 u-net에서는 concat하여 conv연산


## DeepLab

### Conditional Random Fields, CRFs

따로 공부해보자.
{: .notice--info}

### Dilated(Atrous) convolution

![dilated conv](/assets/images/post/220310/boostcamp-CV-3/dilated_conv.jpg){: .align-center}

conv 커널의 각 값 사이에 공간을 만들어 가중치의 수를 유지하면서 더 넓은 receptive field를 확보할 수 있는 conv 연산.

### Atrous depthwise separable convolution


![separable conv](/assets/images/post/220310/boostcamp-CV-3/separable_conv.jpg){: .align-center}


deeplabV3는 입력 사이즈가 커서 연산량, 연산속도를 고려하여 atrous conv와 depthwise separable conv를 결합하여 사용한다.

### 구조

![deeplab v3+](/assets/images/post/220310/boostcamp-CV-3/deeplabv3p.jpg){: .align-center}

출처: [arXiv:1802.02611](https://arxiv.org/pdf/1802.02611.pdf)
{: .text-center}


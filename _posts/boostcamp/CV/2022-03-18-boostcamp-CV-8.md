---
title: "CV] Conditional Generative Model"
excerpt: "유저가 결과물에 관여할 수 있는 생성 모델"

categories:
  - boostcamp CV
tags:
  - [boostcamp, boostcamp CV]

toc: true
toc_sticky: true

date: 2022-03-18T19:08:00+09:00
last_modified_at: 2022-03-18T19:08:00+09:00
---
# 1. Conditional generative model

## 1.1. Conditional generative model

![conditional generative model](/assets/images/post/220318/boostcamp-CV-8/conditional_generative_model.jpg){: .align-center}

* 생성모델에 직접적으로 "조건"을 주고 "조건"을 만족시키는 결과물을 얻는다.
  * 기존의 생성모델은 분포를 모델링하고 샘플링만 가능
  * 조건부 생성모델은 조건을 통해 어떤 분포에서 샘플링할지를 선택할 수 있다.
* 응용
  * 저해상도 이미지를 고해상도로, 저음질의 사운드를 고음질로
  * 기계번역
  * 주어진 주제에 대한 문장 생성

### GAN vs. Conditional GAN

![gan vs. cgan](/assets/images/post/220318/boostcamp-CV-8/gan_vs_cgan.jpg){: .align-center}

출처: boostcamp ai tech - [CV]08. Conditional Generative Model
{: .text-center}

* 입력에 조건을 추가

## 1.2. Conditional GAN and image translation

### Image-to-Image translation

![style transfer](/assets/images/post/220318/boostcamp-CV-8/style_transfer.jpg){: .align-center}

출처: [arXiv:1703.10593](https://arxiv.org/abs/1703.10593)
{: .text-center}

* 이미지를 다른 이미지로 변환
  * style transfer: 사진을 유명 화가의 그림처럼 변환
  * super resolution: 저해상도 이미지를 고해상도 이미지로 변환
  * colorization: 흑백 이미지를 컬러 이미지로 변환

## 1.3. example: Super resolution - SRGAN

![SRGAN](/assets/images/post/220318/boostcamp-CV-8/SRGAN.jpg){: .align-center}

출처: boostcamp ai tech - [CV]08. Conditional Generative Model
{: .text-center}

* 저해상도 이미지를 고해상도 이미지로 변환

### regression model vs. SRGAN

![regression vs. SRGAN](/assets/images/post/220318/boostcamp-CV-8/regression_vs_srgan.jpg){: .align-center}

출처: boostcamp ai tech - [CV]08. Conditional Generative Model
{: .text-center}

![loss별 출력](/assets/images/post/220318/boostcamp-CV-8/outputs_by_loss.jpg){: .align-center}

출처: [arXiv:1609.04802](https://arxiv.org/abs/1609.04802)
{: .text-center}

* 이전의 방식은 MAE, MSE loss를 통해 입력과 출력의 픽셀값을 비교
  * **blurry**한 결과
    * 타겟만큼 현실적인 결과물을 만들어도 타겟과 픽셀값이 다르다면 loss가 크다.
    * 어떤 타겟에 대해서도 적당히 높지않은 loss를 얻을 수 있는 방향으로 학습 진행
      * 타겟들의 평균적인, 흐릿한 이미지를 출력하게된다.
* adversarial loss는 결과물이 실제인지, 가짜인지만 따지므로 출력이 현실적이고 디테일하다.
* toy example: 흰색? 검은색?
  * ![toy example](/assets/images/post/220318/boostcamp-CV-8/SRGAN_toy_example.jpg){: .align-center}

  출처: boostcamp ai tech - [CV]08. Conditional Generative Model
  {: .text-center}
* SRGAN이 더 디테일하고 현실적인 이미지를 생성
  * ![출력 비교](/assets/images/post/220318/boostcamp-CV-8/SRGAN_compare.jpg){: .align-center}

  출처: [arXiv:1609.04802](https://arxiv.org/abs/1609.04802)
  {: .text-center}


# 2. Image translation GANs
## 2.1. Pix2Pix

![pix2pix 예시](/assets/images/post/220318/boostcamp-CV-8/pix2pix_example.jpg){: .align-center}

출처: [arXiv:1611.07004](https://arxiv.org/abs/1611.07004)
{: .text-center}

* 입력 이미지의 내용은 유지한채로 다른 도메인의 이미지로 변환
  * 오브젝트 형태, 종류 등은 유지한채로 질감이나 색상 등을 변환한다.

### loss

![pix2pix loss별 출력](/assets/images/post/220318/boostcamp-CV-8/pix2pix_outputs_by_loss.jpg){: .align-center}

출처: [arXiv:1611.07004](https://arxiv.org/abs/1611.07004)
{: .text-center}

$$G^* = \text{arg}\underset{G}{\text{ min}}\underset{D}{\text{ max }}\mathcal{L}_{cGAN}(G, D) + \lambda\mathcal{L}_{L1}(G)$$

* $\mathcal{L}_{cGAN}(G, D)$
  * 현실적인 결과를 만들기 위한 loss
  * real, fake만 판단하므로 타겟과 같은 내용을 만들 수는 없다.
* $\lambda\mathcal{L}_{L1}(G)$
  * 타겟과 같은 내용을 만들기 위한 loss
  * GAN 학습이 불안정하므로 가이드 역할도 한다.
* 위의 예시 이미지에서 cGAN loss 단독으로 사용하는 경우보다 L1 loss를 함께 사용하는 경우에 물체의 형태, 색감, 조명 등의 요소가 GT와 더 가깝다.

## 2.2. CycleGAN

![cyclegan 예시](/assets/images/post/220318/boostcamp-CV-8/cyclegan_zebras_horses.jpg){: .align-center}

출처: [arXiv:1703.10593](https://arxiv.org/abs/1703.10593)
{: .text-center}


* 직접적인 대응관계가 없는 "unpaired" 데이터셋간의 도메인(스타일) 변환
  * pix2pix는 질감, 색상같은 스타일만 다르고 내용은 같은 "paired" 데이터셋이 필요하다.
  * ![paired vs unpaired](/assets/images/post/220318/boostcamp-CV-8/cyclegan_paired_unpaired.jpg){: .align-center}

  출처: [arXiv:1703.10593](https://arxiv.org/abs/1703.10593)
  {: .text-center}

### loss

* GAN loss + Cycle-consistency loss

#### GAN loss

![cyclegan gan loss](/assets/images/post/220318/boostcamp-CV-8/cyclegan_gan-loss.jpg){: .align-center}

출처: [arXiv:1703.10593](https://arxiv.org/abs/1703.10593)
{: .text-center}

* $L_{GAN}(X\rightarrow Y) + L_{GAN}(Y\rightarrow X)$
  * $D_Y, D_X$: discriminator
  * $G, F$: generator
* X에서 Y, Y에서 X 양방향으로 도메인(스타일) 변환 평가

#### mode collapse

![mode collapse](/assets/images/post/220318/boostcamp-CV-8/cyclegan_mode-collapse.jpg){: .align-center}

* 타겟 도메인을 잘 표현한 이미지가 매번 입력과 무관하게 출력되는 현상
  * unpaired 데이터셋을 사용하므로 직접적으로 출력과 비교할 타겟이 없다.
  * generator 출력이 타겟 도메인을 잘 표현하기만 하면 discriminator는 real로 판단하고 generator는 학습되지 않는다.

#### Cycle-consistency loss

![cyclegan cycle-consistency loss](/assets/images/post/220318/boostcamp-CV-8/cyclegan_cycle-consistency-loss.jpg){: .align-center}

출처: [arXiv:1703.10593](https://arxiv.org/abs/1703.10593)
{: .text-center}

![cyclegan cycle-consistency loss](/assets/images/post/220318/boostcamp-CV-8/cyclegan_cycle-consistency-loss_2.jpg){: .align-center}

* mode collapse 해결을 위한 loss
* G(X)와 F(G(X)), F(Y)와 G(F(Y))의 L1-norm
  * 스타일의 변환만을 원하므로 변환 결과를 다시 역변환하면 입력과 같아야한다.
  * 최대한 내용은 같게 유지하며 스타일 변환만 일어나도록 한다.
* self-supervision


## 2.3. Perceptual loss

![perceptual loss 예시](/assets/images/post/220318/boostcamp-CV-8/perceptual-loss_example.jpg){: .align-center}

출처: [arXiv:1603.08155](https://arxiv.org/abs/1603.08155)
{: .text-center}

* GAN처럼 고퀄리티의 결과를 내면서도 학습은 쉬운 방식
* **pretrained 네트워크가 필요**

* pretrained 네트워크의 저층 레이어들은 인간의 시지각과 비슷하게 이미지를 인식한다.
  * 저층 레이어들의 필터를 시각화하면 방향성이나 색상 등 간단한 요소들을 인식함
  * 저층 레이어들이 이미지를 인간의 지각 공간과 비슷한 공간의 원소로 변환한다는 가정
    * 인간의 눈처럼 중요한 것과 중요하지 않은 것을 걸러서 연관시키는 것을 기대
  
### 구조

![perceptual loss overview](/assets/images/post/220318/boostcamp-CV-8/perceptual-loss_overview.jpg){: .align-center}

출처: [arXiv:1603.08155](https://arxiv.org/abs/1603.08155)
{: .text-center}
  
* 스타일 변환 네트워크 뒤에 pretrain된 loss 네트워크 연결
  * loss 네트워크: pretrained VGG-16 (프리징)

### Perceptual loss

![perceptual loss](/assets/images/post/220318/boostcamp-CV-8/perceptual-loss_loss.jpg){: .align-center}

#### Style reconstruction loss

* 입력 이미지와 스타일이 변환된 이미지의 Gram matrix의 L2-norm
* Gram matrix: 이미지의 스타일을 함축한 tensor
  * 계산
    1. loss 네트워크의 특정 레이어에서 activation 수집
    2. [C, H, W] shape의 activation $\phi$를 [C, H*W] shape의 $\phi'$로 reshape
    3. $\phi'\phi'^\top$ 계산 / shape: [C, C]
  * activation은 채널마다 다른 정보, 패턴을 포함하므로 채널간의 연관성을 통해 스타일을 표현
    * ex) 일정한 간격으로 수평무늬와 파란색이 동시에 나타난다, 체크무늬가 있는 위치에는 붉은색이 나타난다, ...
  * 공간 정보 제거: 스타일은 이미지 전체에 걸쳐 나타나는 경향성

#### Feature reconstruction loss

* 입력 이미지와 스타일이 변환된 출력 이미지의 feature간 L2-norm
  * pretrained 네트워크르 통해 content를 인식하여 비교
  * 스타일이 다르더라도 물체의 형태, 종류 등의 semantic content는 같아야함
* super resolution 등에서 단독으로 사용되어 perceptual quality를 높이는 역할을 할 수도 있다.


# 3. Various GAN applications

* Deepfake, Face de-identification
  * 공통적으로 사람의 얼굴을 조작하는 어플리케이션
  * 전자는 개인 정보 도용으로 사회적 이슈가 된 상태이고, 후자는 개인 정보 보호에 이용된다.
  * 비슷한 task를 다루더라도 방향성이 크게 다를 수 있다.
* Face anonymization with passcode, vidie translation, ...
  
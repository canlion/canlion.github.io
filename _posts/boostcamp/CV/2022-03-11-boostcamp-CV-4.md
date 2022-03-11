---
title: "CV] Object detection"
excerpt: "물체의 바운딩박스와 클래스"

categories:
  - boostcamp CV
tags:
  - [boostcamp, boostcamp CV]

toc: true
toc_sticky: true

date: 2022-03-11T11:45:00+09:00
last_modified_at: 2022-03-11T11:45:00+09:00
---

# Object detection

![object detection](/assets/images/post/220311/boostcamp-CV-4/object_detection.png){: .align-center}

출처: [arXiv:1506.02640](https://arxiv.org/pdf/1506.02640.pdf)
{: .text-center}

* classification + box localization:
  * 물체의 바운딩박스와 클래스
* 자율 주행 시스템, OCR 등에 이용
* 과거에는 전통적인 머신러닝과 마찬가지로 사람에 의한 feature 추출 알고리즘과 단순한 분류기를 사용

**selective search**

![selective search](/assets/images/post/220311/boostcamp-CV-4/selective_search.png){: .align-center}

출처: [selective search for object detection](https://staff.fnwi.uva.nl/th.gevers/pub/GeversIJCV2013.pdf)
{: .text-center}

* 과거에 물체의 위치를 찾기 위해 사용한 알고리즘
* 색상, gradient 등 어떤 기준에 따라 비슷한 영역들을 반복하여 클러스터링하며 물체가 있을법한 영역을 찾아낸다.

# Two-stage detector

* 2단계로 작동하는 detector
  1. 물체가 있을법한 영역 제안
  2. 영역에 어떤 클래스의 물체가 있는지 판단
* 높은 성능, 느린 처리 속도

## R-CNN

![R-CNN](/assets/images/post/220311/boostcamp-CV-4/r-cnn_overview.jpg){: .align-center}

출처: [arXiv:1311.2524](https://arxiv.org/pdf/1311.2524.pdf)
{: .text-center}

* 학습된 분류 모델을 이용한 디텍터
1. selective search로 2000개의 영역(RoI: Region of Interest) 추출
2. 종횡비 무관하게 고정 사이즈로 warping
3. pre-trained CNN으로 각 patch의 feature 추출
      * fc 레이어가 포함된 CNN을 이용하므로 고정 사이즈 입력이 필요
4. SVM을 이용하여 분류
* 특징
  * end-to-end 모델이 아님: CNN, SVM은 따로 학습됨
  * selective search는 사람이 설계하므로 영역 추출 성능이 낮음
  * 2000번의 CNN 연산: 각 patch마다 CNN 연산 - 매우 느림

## Fast R-CNN

![Fast R-CNN](/assets/images/post/220311/boostcamp-CV-4/fast_r-cnn_overview.jpg){: .align-center}

출처: [arXiv:1504.08083](https://arxiv.org/pdf/1504.08083.pdf)
{: .text-center}

* 추출된 feature를 재사용하여 처리 속도 향상
1. selective search로 영역 추출
2. CNN을 통해 이미지의 feature 계산
3. 이미지의 feature에서 selective search로 제안된 영역에 대응되는 부분의 feature 추출
      * RoI pooling layer
        * **제안된 영역의 크기에 관계없이 고정된 크기의 feature를 추출**
        * 입력된 feature를 일정 비율로 분할하여 각 분할영역마다 pooling
4. 각 RoI의 feature로 classification & bbox regression
* 특징
  * CNN 연산을 한 번만 수행하므로 R-CNN 대비 처리속도 대폭 향상 (x8.8 ~ x18.3)

## Faster R-CNN

![Faster R-CNN](/assets/images/post/220311/boostcamp-CV-4/faster_r-cnn_overview.jpg){: .align-center}

출처: [arXiv:1506.01497](https://arxiv.org/pdf/1506.01497.pdf)
{: .text-center}

* 영역 제안 알고리즘을 신경망으로 대체한 end-to-end detector
  * 영역 추출 성능의 향상
  * 모든 파트가 하나로 통합되어 학습이 용이해짐

**IoU, Intersection over Union**

![IoU](/assets/images/post/220311/boostcamp-CV-4/IoU.jpg){: .align-center}

* 오브젝트 디텍션에서 바운딩박스를 잘 찾았는지를 평가하는 지표

### Region Proposal Network & Anchor box

![Faster R-CNN RPN](/assets/images/post/220311/boostcamp-CV-4/faster_r-cnn_rpn.jpg){: .align-center}

출처: [arXiv:1506.01497](https://arxiv.org/pdf/1506.01497.pdf)
{: .text-center}

* Anchor box
  * 미리 지정한 여러 개의 bbox 템플릿 - 다양한 크기와 비율
  * ground-truth bbox와의 IoU를 기준으로 matching
    * IoU > .7인 앵커 박스는 positive, 나머지는 negative
* Region Proposal Network
  * selective search를 신경망으로 대체
  * 이미지로부터 얻은 feature에서 오브젝트가 있을법한 영역을 제안
    * $k$: 앵커 박스의 수
    * 입력된 feature의 각 위치마다 영역 제안
      * $2*k$개의 score: 각 앵커마다 [오브젝트 있다/없다] 분류
      * $4*k$개의 coordinates: 각 앵커와 오브젝트의 bbox의 차이($x, y, w, h$)
    * 학습
      * positive 앵커: 앵커와 gt bbox와의 차이 & 앵커에 오브젝트가 있음을 학습
      * negative 앵커: 앵커에 오브젝트가 없음을 학습

# Single-stage detector

![one vs two](/assets/images/post/220311/boostcamp-CV-4/one_vs_two.jpg){: .align-center}

* Region proposal이 없는 디텍터
* two-stage detector와 비교하여 간단한 구조로 빠르지만 성능은 낮다.

## YOLO, You only look once

![yolo](/assets/images/post/220311/boostcamp-CV-4/yolo_overview.jpg){: .align-center}

![yolo architecture](/assets/images/post/220311/boostcamp-CV-4/yolo_architecture.jpg){: .align-center}

출처: [arXiv:1506.02640](https://arxiv.org/pdf/1506.02640.pdf)
{: .text-center}

* region proposal 없이 바로 디텍션 결과를 출력하는 구조
* 모델 출력은 $S\times S\times (B\times (1+4)+C)$
  * 이미지를 가로, 세로 $S$등분하여 총 $S \times S$개의 영역을 정의하고 각 영역은 영역에 중심점이 포함된 오브젝트를 탐지한다.
  * 각 영역은 $B$개의 앵커를 갖고 있고 앵커마다 ($x, y, w, h, \text{obj score}$)를 계산한다.
  * 각 영역은 $C$개의 클래스 스코어를 갖는다.
  * inference과정에서 각 영역은 $\text{obj score}$가 가장 높은 앵커를 선택한다. (영역마다 하나의 오브젝트만 탐지)
* 특징
  * 매우 빠르다.
  * 당시 real-time detector 중에서는 압도적인 성능
  * regression을 한 번만 하므로 localization 성능이 아쉽다.


## SSD, Single Shot MultiBox Detector

![ssd architecture](/assets/images/post/220311/boostcamp-CV-4/ssd_architecture.jpg){: .align-center}

![ssd multi scale](/assets/images/post/220311/boostcamp-CV-4/ssd_multi_scale_anchor.jpg){: .align-center}

출처: [arXiv:1512.02325](https://arxiv.org/pdf/1512.02325.pdf)
{: .text-center}

* multi-scale detection: 여러 해상도의 feature에서 디텍션을 수행하여 좋은 성능
  * 다양한 스케일의 물체를 디텍션
* 여러 feature에 다양한 앵커 박스를 정의하여 디텍션이 많지만 구조가 단순하여 빠르다.

## RetinaNet

![retinanet architecture](/assets/images/post/220311/boostcamp-CV-4/retinanet_architecture.jpg){: .align-center}

출처: [arXiv:1708.02002](https://arxiv.org/pdf/1708.02002.pdf)
{: .text-center}

* U-Net처럼 low & high level feature를 융합
* multi-scale 디텍션
* 빠르고 좋은 성능
* 구조
  * $K$: 클래스 수 / $A$: 앵커 박스 수
  * 작은 high-level feature를 업샘플링하여 low-level feature와 합한다.

### focal loss

* 앵커박스를 이용하면 feature의 모든 위치에서 앵커박스를 정의하므로 positive 앵커박스는 매우 적고 negative 앵커박스는 매우 많아 class imbalance 문제를 겪는다.

$$FL(p_t) = -(1-p_t)^\gamma log(p_t)$$

* focal loss는 cross entropy loss에 각 샘플의 prediction에 따라 가중치를 부여한다.
  * 잘 맞추면 작은 가중치를, 잘 못맞추면 높은 가중치를 준다.
  * 예시 ($\gamma=1.$)
    * label: 1, $p_t$ = 0.8
      * 가중치: (1-$p_t$) = 0.2
    * label: 1, $p_t$ = 0.2
      * 가중치: (1-$p_t$) = 0.8
* 전체적인 loss의 값은 줄어들지만 예측이 잘 되는 샘플은 약하게 학습시키고, 예측이 잘 안되는 샘플은 강하게 학습시키며 class imbalance 문제를 완화한다.

![focal loss](/assets/images/post/220311/boostcamp-CV-4/focal_loss.jpg){: .align-center}

출처: [arXiv:1708.02002](https://arxiv.org/pdf/1708.02002.pdf)
{: .text-center}

# Detection with Transformer

* 트랜스포머는 자연어처리 문제에서 뛰어난 성능을 보이며 판도를 바꿈.
* 비전 분야에서도 트랜스포머를 적용하면서 성과를 보이고 있음.
  * google: ViT
  * facebook: DeiT, DETR

## DETR, DEtection TRansformer

![DETR architecture](/assets/images/post/220311/boostcamp-CV-4/DETR.jpg){: .align-center}

출처: [arXiv:2005.12872](https://arxiv.org/pdf/2005.12872.pdf)
{: .text-center}

# NMS, Non-Maximum Supression

![NMS](https://cdn.analyticsvidhya.com/wp-content/uploads/2020/07/Screenshot-from-2020-07-31-18-29-32.png){: .align-center}

출처: [AnalyticsVidhya: Selecting the Right Bounding Box Using Non-Max Suppression (with implementation)](https://www.analyticsvidhya.com/blog/2020/08/selecting-the-right-bounding-box-using-non-max-suppression-with-implementation/)
{: .text-center}

* 디텍터는 같은 오브젝트에 대해 여러개의 bbox를 예측하기도 한다.
* NMS는 같은 오브젝트를 포착한 bbox를 필터링하여 하나의 오브젝트에는 하나의 bbox가 연결되도록 하는 작업이다.
  1. objectness score 순으로 bbox들을 정렬한다.
  2. 가장 높은 objectess score를 가진 bbox를 선택한다.
  3. 앞서 선택한 bbox와의 IoU가 0.5 이상인 bbox들을 모두 제거한다.
  4. 2~3 과정을 반복한다.

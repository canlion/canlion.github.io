---
title: "CV] Instance/panoptic segmentation and landmark localization"
excerpt: "인스턴스들과 배경의 구분 / 랜드마크 디텍션"

categories:
  - boostcamp CV
tags:
  - [boostcamp, boostcamp CV]

toc: true
toc_sticky: true

date: 2022-03-17T22:03:00+09:00
last_modified_at: 2022-03-17T22:03:00+09:00
---
# 0. Semantic segmentation & Instance segmentation & Panoptic segmentation

![segmentation task 비교](/assets/images/post/220317/boostcamp-CV-7/compare_segmentation_tasks.jpg){: .align-center}

출처: [Panoptic Segmentation](https://openaccess.thecvf.com/content_CVPR_2019/papers/Kirillov_Panoptic_Segmentation_CVPR_2019_paper.pdf)
{: .text-center}

* Semantic segmentation: 배경과 클래스 구분 - 같은 클래스는 한 덩어리
* Instance segmentation: 클래스와 인스턴스 구분
* Panoptic segmentation: 배경과 클래스, 인스턴스 구분

# 1. Instance segmentation

## Mask R-CNN

![mask r-cnn result](/assets/images/post/220317/boostcamp-CV-7/mask-r-cnn_result.jpg){: .align-center}

![mask r-cnn mask branch](/assets/images/post/220317/boostcamp-CV-7/mask-r-cnn_mask-branch.jpg){: .align-center}

출처: [arXiv:1703.06870](https://arxiv.org/abs/1703.06870)
{: .text-center}

* Faster R-CNN의 개선판
  * 기존의 정수 좌표만 지원하던 RoI pooling을 RoIAlign으로 대체하여 정수 좌표가 아니더라도 interpolation을 통해 더 정교한 region proposal 수행
* Mask branch를 추가하여 세그멘테이션 수행
  1. 디텍션마다 클래스 수만큼의 세그멘테이션 마스크 생성
  2. 디텍션 분류 결과로 세그멘테이션 마스크를 선택

## YOLACT

![yolact overview](/assets/images/post/220317/boostcamp-CV-7/yolact_overview.jpg){: .align-center}

출처: [arXiv:1904.02689](https://arxiv.org/abs/1904.02689)
{: .text-center}

* single-stage segmenter
* prototypes의 선형 결합으로 세그멘테이션 생성
  * prototypes
    * 세그멘테이션 마스크는 아니지만 마스크를 만들어낼 수 있는 기본적인 soft segmentation components
    * 세그멘테이션 마스크를 span할 수 있는 basis
  * 과정
    * protonet에서 prototypes 생성
    * prediction head에서 디텍션과 동시에 각 prototype의 가중치 생성
    * prototypes를 가중합하여 세그멘테이션 생성
* **key:** 클래스 수 만큼 세그멘테이션 마스크를 만드는 대신 적당한 수의 prototype을 만들고 조합하는 방식으로 속도 개선

## YolactEdge

![yolactedge overview](/assets/images/post/220317/boostcamp-CV-7/yolactedge_overview.jpg){: .align-center}

출처: [arXiv:2012.12259](https://arxiv.org/abs/2012.12259)
{: .text-center}

* 비디오 세그멘테이션을 위한 YOLACT
* YOLACT를 edge device에서 사용하기 위해 속도 개선
  * 일정 간격으로 key frame을 설정하여 key frame 처리시에 얻은 feature를 non-key frame에서 사용하여 연산을 줄인다.
  * ex) 7번째 이미지를 처리할 때 5번째 이미지를 처리할 때 얻은 feature를 재활용하여 연산량을 줄인다.
  * 빠르면서도 성능을 비슷하게 유지
* 다른 프레임의 feature를 이용하므로 세그멘테이션 마스크가 깜빡이거나 떨리는 증상 존재

# 2. Panoptic segmentation

* 배경까지 구분하는 점에서 instance segmentation보다 유용

## UPSNet

![upsnet overview](/assets/images/post/220317/boostcamp-CV-7/upsnet_overview.jpg){: .align-center}

![upsnet panoptic head](/assets/images/post/220317/boostcamp-CV-7/upsnet_panoptic_head.jpg){: .align-center}

출처: [arXiv:1901.03784](https://arxiv.org/abs/1901.03784)
{: .text-center}

* semantic segmentation 결과와 instance segmentation 결과를 이용하여 panoptic segmentation 결과 생성
* panoptic head
  * $Y_i$: $i$번째 instance segmentation mask
  * $X$: semantic segmentation mask
    * $X_{stuff}$: 배경
    * $X_{thing}$: 오브젝트
  * 과정
    * $N_{inst}$: instance mask
      * $X_{things}$에서 $i$번째 인스턴스의 bbox 밖을 0-마스킹하여 $X_{mask}$ 생성
      * $Y_i$를 스케일과 크기가 맞도록 resize, pad하여 $X_{mask}$와 더한다.
    * $N_{stuff}$: $X_stuff$ 그대로 사용
    * unknown: 배경도 아니고 찾고자하는 카테고리의 오브젝트도 아니다. $X_{things}$에서 $X_{mask}$를 뺀다.


## VPSNet (for video)

![vpsnet overview](/assets/images/post/220317/boostcamp-CV-7/vpsnet_overview.jpg){: .align-center}

출처: [arXiv:2006.11339](https://arxiv.org/abs/2006.11339)
{: .text-center}

* 비디오 세그멘테이션을 위한 UPSNet
* motion map $\phi$: $t-\tau$ 시점의 이미지에서의 위치를 $t$ 시점의 이미지에서의 위치로 맵핑
  * $t$시점까지의 오브젝트의 위치 이동을 반영하여 $t-\tau$ 시점의 feature를 수정한다.
* 과거 시점의 feature를 모션맵을 통해 수정한 후 현재 시점의 feature와 함께 이용한다.
  * $t-\tau$ 시점까지는 보이다가 $t$시점에는 가려져서 보이지 않는 오브젝트처럼 $t$ 시점에서는 알 수 없는 정보를 줄 수 있다.
  * 과거 시점의 feature를 함께 사용하므로 더 연속적인 세그멘테이션 결과를 얻을 수 있다.
* Track head: $t-\tau$의 RoI와 $t$의 RoI를 매칭시켜 트랙킹

# 3. Landmark localization = Keypoint estimation

![pose estimation](/assets/images/post/220317/boostcamp-CV-7/pose_estimation.jpg){: .align-center}

출처: [arXiv:1611.08050](https://arxiv.org/abs/1611.08050)
{: .text-center}

* 키포인트의 좌표를 찾는다.

## Coordinate regression vs. heatmap classification

* coordinate regression
  * dense 레이어를 통해 키포인트의 좌표값을 직접적으로 regression
  * 부정확하고 일반화 성능이 떨어진다.
* heatmap classification
  * 세그멘테이션처럼 이미지상에서 키포인트 부분을 heatmap으로 표현한다.
  * 성능이 좋지만 연산량이 많다.

### landmark location to Gaussian heatmap

* heatmap 방식으로 키포인트를 찾을때 키포인트의 위치를 어떻게 표시할까?
  * 점 하나로 띡 표시하기엔 inference가 너무 어렵다.
  * 보통 키포인트 위치를 중심으로 가우시안 분포를 그려준다.

![gaussian keypoint](/assets/images/post/220317/boostcamp-CV-7/gaussian_keypoint.jpg){: .align-center}

출처: boostcamp ai tech - [CV]07. Landmark Localization
{: .text-center}

$$G_\sigma(x, y)=\text{exp}\left(-\frac{(x-x_c)^2+(y-y_c)^2}{2\sigma^2}\right)$$

* $x_c, y_c$: 키포인트 위치

## Hourglass network

![hourglass overview](/assets/images/post/220317/boostcamp-CV-7/hourglass_overview.jpg){: .align-center}

출처: [arXiv:1603.06937](https://arxiv.org/abs/1603.06937)
{: .text-center}

* UNet과 비슷한 구조의 반복
  * skip connection에 conv 레이어 추가, concat 대신 add
  * 결과를 반복하여 정제한다.

## Extensions

### DensePose

![densepose overview](/assets/images/post/220317/boostcamp-CV-7/densepose_overview.jpg){: .align-center}

출처: [arXiv:1802.00434](https://arxiv.org/abs/1802.00434)
{: .text-center}

* 이미지에서 UV 맵 좌표를 예측
  * UV 맵 좌표는 3D mesh의 포인트와 1대1 맵핑되므로 3D mesh를 예측하는 것이나 다름없다.
  * **데이터와 출력의 표현을 바꿔 2D를 다루는 네트워크로 3D mesh를 예측**
* mask R-CNN과 유사
  * mask branch를 patch와 U, V 좌표를 예측하는 3D surface regression branch로 교체
  * patch: 각 바디 파트의 세그멘테이션 맵

### RetinaFace

![retinaface overview](/assets/images/post/220317/boostcamp-CV-7/retinaface_overview.jpg){: .align-center}

출처: [arXiv:1905.00641](https://arxiv.org/abs/1905.00641)
{: .text-center}

* FPN + multi-task branches
  * 얼굴과 연관된 다양한 task를 동시에 수행한다
  * 학습 과정에서 여러 task로부터 공통된 대상을 조금씩 다른 측면에서 본 정보가 생성되므로 backbone이 좋은 성능을 가지게된다. (데이터양에 비해 높은 성능 향상)


# 4. Detecting objects as keypoints

## CornerNet

![cornernet overview](/assets/images/post/220317/boostcamp-CV-7/cornernet_overview.jpg){: .align-center}

출처: [arXiv:1808.01244](https://arxiv.org/abs/1808.01244)
{: .text-center}

* top-left, bottom-right 두 지점을 찾아 오브젝트 디텍션을 수행
* heatmap을 통해 TL, BR 두 지점을 찾고 embedding을 통해 같은 오브젝트의 TL, BR을 매칭시킨다.
* single-stage. 빠르다.

## CenterNet

### bbox = {Top-left, Bottom-right, Center}

![centernet 0](/assets/images/post/220317/boostcamp-CV-7/centernet_0.jpg){: .align-center}

출처: [arXiv:1904.08189](https://arxiv.org/abs/1904.08189)
{: .text-center}

* 중앙점이 최종 결정에 도움이 된다!
* 당시에 single-stage 디텍터 중에서는 압도적이고, 2-stages 디텍터 중에서도 상당히 좋은 성능
* 그런데 TL, BR을 찾는데 중앙점까지 찾는건 겹치는 정보를 찾는게 아닐까?

### bbox = {Width, Height, Center}

![centernet 2](/assets/images/post/220317/boostcamp-CV-7/centernet_2.jpg){: .align-center}

![centernet 2 performance](/assets/images/post/220317/boostcamp-CV-7/centernet_2_performance.jpg){: .align-center}

출처: [arXiv:1904.07850](https://arxiv.org/abs/1904.07850)
{: .text-center}

* 중앙점을 이용할 때 가로, 세로 크기만 찾으면 최소의 정보로 bbox를 구성할 수 있다.
* 빠른데 성능까지 좋다.

# 강조

* 새로운 task를 위해 완전히 새로운 구조를 설계하기보다는 기존의 디자인 패턴을 따라 설계하는 것부터 시작하자.
  * FPN + task에 적당한 head
* 데이터와 출력의 표현을 바꿔 속도, 성능을 향상시킬 수 있는 방법을 찾자.
  * CenterNet
  * DensePose
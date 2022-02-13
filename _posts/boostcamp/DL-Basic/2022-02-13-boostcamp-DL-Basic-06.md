---
title: "DL Basic 06] Computer Vision Applications"
excerpt: "CNN의 이용 분야 - segmentation / object detection"

categories:
  - boostcamp DL Basic
tags:
  - [boostcamp, boostcamp DL Basic]

toc: true
toc_sticky: true

date: 2022-02-13T14:44:00+09:00
last_modified_at: 2022-02-13T14:44:00+09:00
---

# Semantic Segmentation

픽셀단위의 분류 - dense classification, per-pixel classification이라고도 함. 자율주행, 운전 보조 등에 주로 사용됨.

## Fully Convolutional Network

![convolutionalization](/assets/images/post/220213/boostcamp-DL-Basic-06/convolutionalization.png){: .align-center}

flatten-dense 레이어를 입력과 동일한 크기의 필터를 사용하는 conv 레이어로 대체함. 가중치의 수는 동일함.

![convolutionalization_1](/assets/images/post/220213/boostcamp-DL-Basic-06/convolutionalization_1.png){: .align-center}

![fcn heatmap](/assets/images/post/220213/boostcamp-DL-Basic-06/fcn_heatmap.png){: .align-center}
출처: [arXiv:1411.4038](https://arxiv.org/pdf/1411.4038.pdf)
{: .text-center}

dense 레이어는 입력의 크기가 항상 동일해야하지만 conv로 대체하면 shared parameters 특성 덕분에 입력의 shape에서 자유로워짐. 입력의 작은 영역을 출력의 작은 영역으로 연결할 수 있게되어 segmentation이나 heatmap을 출력할 수 있게 된다.

## Deconvolution (conv transpose)

![deconvolution](/assets/images/post/220213/boostcamp-DL-Basic-06/deconv.png){: .align-center}

CNN은 pooling, conv 레이어의 반복으로 출력의 크기가 입력에 비해 작아진다. heapmap이나 segmentation을 출력하면 입력에 비해 해상도가 낮아진다. 그래서 해상도를 높이기 위해 작은 feature를 upsampling 하는 방식을 도입한다.

부르기를 deconvolution이지만 conv 레이어의 출력은 입력과 필터 곱을 다 합하므로 역연산은 불가. 그냥 feature로부터 크기가 큰 feature를 만든다고 생각하자. (변수가 아주 많지만 식이 하나인 연립방정식같은 느낌?)

# Detection

이미지에서 특정 카테고리에 속하는 물체의 바운딩박스를 찾는 테스크.

## R-CNN

![r-cnn](/assets/images/post/220213/boostcamp-DL-Basic-06/rcnn.png){: .align-center}

출처: [arXiv:1311.2524](https://arxiv.org/pdf/1311.2524.pdf)
{: .text-center}

1. 이미지 입력
2. 이미지에서 selective search를 통해 오브젝트가 있을 것 같은 영역을 2천개 선정
3. 2천개 영역을 동일한 사이즈로 리사이즈하여 AlexNet 연산, feature 추출 (2천번의 연산)
4. feature를 SVM으로 분류, regressor를 통해 바운딩 박스 regression

AlexNet을 2천번을 돌리려니 느리다. 이미지를 크기, 종횡비 무관하게 같은 사이즈로 리사이즈하다보니 오브젝트의 특성을 망가트림. (dense 레이어가 존재)

feature extractor, SVM, regressor 모두 따로 학습시켜야하고 느리다.

## SPPNet

R-CNN의 속도를 개선하기위해 이미지에서 영역을 뜯어 CNN 연산을 하는 대신 전체 이미지를 CNN에 넣어 얻은 feature에서 영역에 해당하는 부분을 뜯는 방식을 사용. 2천번의 CNN 연산이 단 한 번으로 줄어든다. 

Spatial Pyramid Pooling 레이어의 활용 - 입력의 크기, 종횡비에 무관하게 고정된 크기의 벡터를 추출함. 무리한 이미지 리사이징이 필요하지 않음.

마찬가지로 feature extractor, SVM, regressor 모두 따로 학습시켜야하고 느리다.

## Fast R-CNN

![fast r-cnn](/assets/images/post/220213/boostcamp-DL-Basic-06/fast-rcnn.png){: .align-center}

출처: [arXiv:1504.08083](https://arxiv.org/pdf/1504.08083.pdf)
{: .text-center}

SPPNet과 거의 유사하지만 CNN ~ classifier, regressor 까지를 신경망으로 구성하여 end-to-end 학습이 가능해졌다.

## Faster R-CNN

![faster r-cnn architecture](/assets/images/post/220213/boostcamp-DL-Basic-06/faster-rcnn.png){: .align-center}

출처: [arXiv:1506.01497](https://arxiv.org/pdf/1506.01497.pdf)
{: .text-center}

Fast R-CNN에서 selective search를 신경망인 Region Proposal Network로 대체했다. 

![faster r-cnn anchors](/assets/images/post/220213/boostcamp-DL-Basic-06/faster-rcnn-anchors.png){: .align-center}

출처: [arXiv:1506.01497](https://arxiv.org/pdf/1506.01497.pdf)
{: .text-center}

Region Proposal Network는 미리 $k$개의 바운딩박스 템플릿, anchor를 정의한다. 입력된 feature의 각 값마다 $k$개의 anchor에 적절하게 들어맞는 오브젝트가 존재하는지 확인한다. 

feature의 각 위치마다 2개의 1x1 conv를 이용하여 다음 값들을 계산한다.
* $2k$개의 값: $k$개의 앵커마다 적절한 오브젝트가 [존재한다/존재하지않는다]를 의미하는 2개의 값
* $4k$개의 값: $k$개의 앵커마다 오브젝트 바운딩 박스의 $x, y, w, h$를 구할 수 있는 4개의 값

위의 값들을 이용해 오브젝트가 존재할만한 RoI(Region of Interest, 관심 영역)를 구해서 RPN에 입력된 feature를 뜯어 분류한다.

간추리면
1. 이미지를 입력
2. CNN으로부터 feature 획득
3. RPN에 2의 feature를 입력하여 오브젝트가 있을법한 영역의 좌표를 구함
4. 2의 feature에서 3의 영역 좌표에 해당하는 부분을 뜯어 클래스 분류

## YOLO

매우 빠른 detection 알고리즘.

이전 R-CNN 계열의 알고리즘은 바운딩 박스를 찾은 후에 feature에서 바운딩 박스 영역만큼을 뜯어 분류를 했다면 YOLO 계열은 바운딩 박스 regression과 분류를 동시 처리한다. Region Proposal 유닛이 없다.

![YOLO](/assets/images/post/220213/boostcamp-DL-Basic-06/YOLO.png){: .align-center}

출처: [arXiv:1506.02640](https://arxiv.org/pdf/1506.02640.pdf)
{: .text-center}

구조 자체는 앞서 R-CNN 계열과 비교해 매우 간단하다. 이미지를 CNN을 통과시켜 나온 feature에 바로 바운딩박스와 클래스 정보를 담겨있다.

YOLO 네트워크는 `S x S x (B x 5 + C)` 크기의 feature를 출력한다.
* `S`: 이미지를 가로, 세로 `S`등분하여 총 `S x S` 셀로 분할
* `B`: 각 셀에서 찾을 바운딩 박스 수
  * `B x 5`: 박스마다 `x, y, w, h`를 구할 수 있는 값과 박스의 오브젝트 포함 여부에 대한 점수 `confidence score`
* `C`: 클래스의 수, 셀에 포함된 오브젝트 분류

임의의 오브젝트의 중심을 포함한 셀은 그 오브젝트를 탐지해야한다.

각 셀의 `B`개의 박스 중 `confidence score`가 가장 높은 박스 하나와 `C`개의 클래스 중 점수가 가장 높은 클래스를 오브젝트의 바운딩박스와 클래스로 선택한다. (각 셀마다 오직 하나의 오브젝트를 찾을 수 있다.)
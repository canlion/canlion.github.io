---
title: "CV] CNN Visualization"
excerpt: "CNN의 내부 동작을 확인한다."

categories:
  - boostcamp CV
tags:
  - [boostcamp, boostcamp CV]

toc: true
toc_sticky: true

date: 2022-03-17T09:29:00+09:00
last_modified_at: 2022-03-17T09:29:00+09:00
---

# 1. Visualizing CNN

## CNN visualization

* CNN은 여러 레이어에 걸쳐 가중치들의 조합으로 복잡하게 연결되어있어 사실상 해석이 불가능한 블랙박스.
* visualization으로 내부를 들여다보자. (디버깅 툴)
  * 내부는 어떻게 되어있을까?
  * 왜 성능이 좋을까?
  * 어떻게 개선할 수 있을까?

## ZFNet

![zfnet deconv visualization](/assets/images/post/220317/boostcamp-CV-5/zfnet_deconv_visualization.jpg){: .align-center}

출처: [arXiv:1311.2901](https://arxiv.org/abs/1311.2901)
{: .text-center}

* CNN이 어떤 지식을 배웠는지 deconvolution을 이용해 시각화
* 고층 레이어가 어떤 feature를 학습하는지 시각화하여 튜닝에 참고
  * 저층 레이어는 방향성, blob을 찾는 영상처리 필터 학습
  * 고층 레이어는 구체적인 의미가 있는 feature 학습
* 2013 imagenet 우승 - 시각화가 실용적인 기술이 될 수 있음을 보여줌

## filter visualization [low level feature]

![filter weight visualization](/assets/images/post/220317/boostcamp-CV-5/filter_weight_visualization.jpg){: .align-center}

출처: boostcamp ai tech - [CV]06. CNN Visualization
{: .text-center}

* 일반적으로 첫 레이어를 시각화
  * 두번째 레이어부터는 채널 수가 많아 직관적인 이해를 얻기 힘들다. (사람이 CNN을 이해하는 것이 시각화의 목적)
  * 뒤에 있는 레이어일수록 앞의 레이어들의 필터들이 합성된 필터를 가지므로 매우 추상적, 해석 불가
* filter visualization: 방향성, blob, color 등을 파악하는 기본적인 operation을 학습
* activation visualization: filter에 따라 다른 결과. 각 filter가 무엇을 보고있는지 어느정도 느낌이 온다.

## types of neural network visualization

![types of visualization](/assets/images/post/220317/boostcamp-CV-5/type_of_visualization.jpg){: .align-center}

출처: boostcamp ai tech - [CV]06. CNN Visualization
{: .text-center}

* Analysis of model behaviors: 모델의 행동, 특성 분석
* Model decision explanation: 주어진 입력에 대해 모델이 왜 그런 출력을 냈는지 분석

# 2. Analysis of model behaviors

## Embedding feature analysis [high level feature]

### NN in a feature space
![nn in feature space](/assets/images/post/220317/boostcamp-CV-5/embedding_feature_analysis.jpg){: .align-center}

출처: [ImageNet Classification with Deep Convolutional Neural Networks](https://papers.nips.cc/paper/2012/file/c399862d3b9d6b76c8436e924a68c45b-Paper.pdf)
{: .text-center}

![](/assets/images/post/220317/boostcamp-CV-5/embedding_feature_analysis_2.jpg){: .align-center}

* 네트워크의 feature 벡터의 유클리드 거리가 가까운 이미지들을 출력하는 방식 (Nearest neighbors)
  * 의미적으로 같은 이미지들이 클러스터링된다. -> 모델은 각 클래스의 컨셉에 대해 이해하는 것 같다.
  * 단순한 픽셀 비교와는 달리 오브젝트 위치나 포즈 등의 variance에 강하다.

### Dimensionality reduction

![t-SNE](/assets/images/post/220317/boostcamp-CV-5/t-sne.jpg){: .align-center}

출처: [Visualizing Data using t-SNE](https://www.jmlr.org/papers/volume9/vandermaaten08a/vandermaaten08a.pdf)
{: .text-center}

* backbone을 통해 뽑은 feature는 너무 고차원적이고 추상적이라 해석이 힘들다.
* 이해하기 쉽도록 차원을 줄인다.
* 대표적인 예: **t-SNE**
  * 근접한 클러스터간의 경계에 위치한 샘플들은 구분하기 힘든, 유사한 샘플이구나.

## Activation investigation [mid, high level feature]

### Layer activation

![layer activation](/assets/images/post/220317/boostcamp-CV-5/layer_activation.jpg){: .align-center}

출처: [arXiv:1704.05796v1](https://arxiv.org/abs/1704.05796v1.pdf)
{: .text-center}

* activation의 한 채널을 threshold하여 mask 생성 후 이미지에 overlay한다.
* 노드가 어떤 부분에 집중하는지 알 수 있다.
* 각 노드들이 오브젝트의 간단한 부분들을 파악하고 조합해 물체를 인식하는구나.

### Maximally activating patch [mid level feature]

![maximally activating patch](/assets/images/post/220317/boostcamp-CV-5/maximally_activating_patches.jpg){: .align-center}

출처: [arXiv:1412.6806](https://arxiv.org/abs/1412.6806.pdf)
{: .text-center}



* activation의 한 채널에서 가장 큰 값의 위치를 중심으로 이미지에서 patch를 뜯어본다. (receptive field 고려)
* 마찬가지로 노드가 어떤 부분에 집중하는지 알 수 있다.
* 작은 영역만 확인하므로 mid level feature 분석에 적합하다.

### Class visualization [score of target class]

![class visualization](/assets/images/post/220317/boostcamp-CV-5/class_visualization.jpg){: .align-center}

resnet18 이용
{: .text-center}

* 데이터없이 모델이 학습한 feature만 이용하는 방법
* 분류 모델의 특정 클래스에 대한 스코어를 최대화하는 입력을 찾는다.
  * 모델 입력을 파라미터화하여 입력 자체를 학습시킨다.
  * 학습된 입력은 클래스에 대한 템플릿으로 이해할 수 있다.

학습:

$$I^*=\text{argmax}_{I}f(I)-\lambda\Vert I\Vert_2^2$$

* $I$: 입력
* $\text{argmax}_{I}f(I)$: 특정 클래스에 대한 스코어를 최대화하는 $I$
* $\lambda\Vert I\Vert_2^2$: 정규화항. 영상은 [0, 1] 또는 [0, 255]처럼 일정 범위를 가지므로 너무 범위를 벗어난 영상이 나오지않도록 regulaization을 적용.
* 업데이트된 입력을 다시 입력하여 계속 최적화.
* 중간중간 gaussian blur를 해주면 더 나은 결과가 나온다.
* $-1$을 곱해 경사하강법을 사용하자.

# 3. Model decision explanation

* 모델이 특정 입력에 대해 어떻게 반응하는지 해석

## Saliency test

### Occlusion map

![occlusion map](/assets/images/post/220317/boostcamp-CV-5/occlusion_map.jpg){: .align-center}

출처: boostcamp ai tech - [CV]06. CNN Visualization
{: .text-center}

* 이미지의 일부 영역을 마스킹하여 입력하고 스코어의 변화를 분석한다.
  * 중요한 영역을 마스킹해버리면 스코어가 낮을 것.
* 이미지의 모든 영역을 차례로 마스킹해보면서 중요한 영역이 어딘지를 파악한다.

### via backpropagation

* Simonyan et al., CoRR 2013
* 이미지를 입력하고 이미지 클래스의 스코어에 대해서 역전파하고 입력단에서 gradient의 크기를 시각화한다.
  * gradient 크기가 큰 부분이 바뀌면 스코어에 큰 영향을 미치므로 중요한 영역이라는 결론

## Backpropagate features

* backpropagate에 집중한 방법

### guided backpropagation

잘 이해안되므로 논문 공부 후 보충 예정..
{: .notice--primary}

![guided backpropagation](/assets/images/post/220317/boostcamp-CV-5/guided_backpropagation.jpg){: .align-center}

![guided backpropagation 2](/assets/images/post/220317/boostcamp-CV-5/guided_backpropagation_2.jpg){: .align-center}

출처: [arXiv:1412.6806](https://arxiv.org/abs/1412.6806.pdf)
{: .text-center}

* ReLU activation function
* Backward pass
  * backpropagation
    * forward pass때의 ReLU의 마스킹 영역을 역전파때에 그대로 적용
  * deconvnet
    * 역전파되는 gradient에 relu를 적용한다.
  * guided backpropagation
    * forward pass의 ReLU 마스킹 영역 + 역전파되는 gradient에 relu 적용
    * 더 이해하기 쉬운 결과를 보여준다.
    * gradient에 relu를 적용하는 것은 영향력을 더 키워야하는 부분만을 반영하려는 의도가 아닐까

## Class activation mapping

### Class activation mapping, CAM

![CAM](/assets/images/post/220317/boostcamp-CV-5/CAM.jpg){: .align-center}

출처: [arXiv:1512.04150](https://arxiv.org/abs/1512.04150.pdf)
{: .text-center}

* activation의 각 채널에 가중치를 부여하여 시각화하는 방식

$$
\begin{matrix}
S_c & = & \sum_k w^c_kF_k \\
& = & \sum_k w^c_k \sum_{(x, y)}f_k(x, y) \\
& = & \sum_{(x, y)}\sum w^c_k f_k(x, y)
\end{matrix}
$$

* $S_c$: $c$ 클래스의 스코어
* $F_k$: GAP activation의 $k$번째 값
* $f_k(x, y)$: GAP에 입력되는 feature의 $(x,y)$ 지점의 값 (feature의 크기로 나눈 값일 듯 - 평균된 값을 평균하기 전으로 돌린 값이므로?)
* **CAM**
  * $\sum_{(x, y)}\sum w^c_k f_k(x, y)$
  * 공간에 대한 정보가 남아있는 상태로 시각화 가능

특징
* 네트워크에 GAP 레이어가 필요
* 모델이 어디를 보고 이미지를 분류했는지 해석할 수 있다.
* CAM 결과를 threshold하여 오브젝트 디텍터로 사용할 수 있다.
  * weakly supervised learning: 정교한 task를 상대적으로 덜 정교한 task를 통해 간접적으로 해결

### Grad-CAM

![Grad-CAM](/assets/images/post/220317/boostcamp-CV-5/grad_cam.jpg){: .align-center}

![Grad-CAM](/assets/images/post/220317/boostcamp-CV-5/grad_cam_2.jpg){: .align-center}

출처: [arXiv:1610.02391](https://arxiv.org/abs/1610.02391)
{: .text-center}

* $A$: backbone으로 추출한 feature
* feature의 가중치 $\alpha^c_k$
  * 각 채널 gradient의 평균

$$\alpha^c_k=\frac{1}{Z}\sum_i\sum_j\frac{\partial y^c}{\partial A^k_{ij}}$$

* Grad-CAM
  * feature의 각 채널에 가중치를 곱해 더한 후 ReLU

$$L^c_{Grad-CAM} = \text{ReLU}(\sum_k \alpha^c_k A^k)$$
    

특징
* 네트워크 구조 변경 불필요
* backbone이 CNN이면 어떤 task에도 적용 가능
* guided backprop.와 결합
  * guided backprop.의 결과는 오브젝트 형태를 샤프하게 잡아내지만 클래스에 대해 민감하지 않음.
  * grad-CAM은 클래스에 대한 정보를 잘 잡아내지만 형태는 디테일하지 않음.
  * 두 결과를 곱하면 guided backprop.의 샤프한 결과에서 특정 클래스에 대한 영역만을 강조하여 시각화 가능

### 기타...

* **SCOUTER** [Li et al., arXiv 2020]
  * 모델이 입력을 특정 카테고리로 분류했는지뿐만 아니라 왜 특정카테고리로 분류하지 않았는지까지 시각화하는 방식
* **GAN dissection** [Bau et al., ICLR 2019]
  * GAN에서 어떤 feature가 무엇을 담당하는지를 추적하여 해당 feature를 조작해 GAN 결과를 조작하는 내용
  * 해석법을 찾아냈다면 해석에서 끝나지 않고 응용

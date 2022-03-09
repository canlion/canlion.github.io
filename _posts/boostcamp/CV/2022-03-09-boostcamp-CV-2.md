---
title: "CV] Annotation data efficient learning"
excerpt: "갖고있는 데이터로 잘해보기."

categories:
  - boostcamp CV
tags:
  - [boostcamp, boostcamp CV]

toc: true
toc_sticky: true

date: 2022-03-09T20:44:00+09:00
last_modified_at: 2022-03-09T20:44:00+09:00
---

딥러닝은 많은 데이터를 필요로 하지만 데이터셋을 구축하는 것은 쉽지 않다.

# Data augmentation

데이터셋은 항상 편향되어있어 실제 데이터와 다르다. 사람이 데이터를 생성하고, 사람이 수집하므로 사람의 편향이 첨가된다. 예를 들어 사진으로 데이터셋을 구축할때 각 사진은 찍힌 시간의 유행이나 촬영자의 습관이 반영된다.

데이터셋에 담을 수 있는 데이터들은 실제를 반영한 데이터 중 극히 일부에 불과하고 편향되어있기까지 하다. 결국 데이터셋은 실제 데이터 분포를 제대로 표현할 수 없다.

![augmentation fill space](/assets/images/post/220309/boostcamp-CV-2/augmentation_fill_space.jpg){: .align-center}

보유한 데이터들을 다양하게 변형시켜 실제 데이터 분포와 유사하게 만드는 것이 목표.

![cutmix](/assets/images/post/220309/boostcamp-CV-2/cutmix.png){: .align-center}

출처: [cutmix / arXiv:1905.04899](https://arxiv.org/pdf/1905.04899.pdf)
{: .text-center}

* crop: 간단하면서도 강력한 기법
* cutmix:
  * 한 이미지의 일부를 다른 이미지 위에 붙여넣는 방식으로 두 이미지가 차지하는 영역에 맞춰 라벨도 섞어준다.
  * 성능 향상과 더불어 이미지 내의 오브젝트의 위치를 정교하게 파악하게된다.
    * 어떤 오브젝트가 얼마나 섞였는지를 잘 파악해야 타겟값을 맞출 수 있으므로?
* RandAugment - augmentation policy(sequence) search
  * 여러 augmentation 기법 중 어떤 기법들을, 어떤 강도로, 어떤 순서로 사용할지를 정하려면 많은 노력이 필요
  * 무작위로 [여러 개의 기법]과 [각 기법을 적용할 강도]를 샘플링하며 실험을 반복하여 최적의 augmentation policy를 찾는다.


# Leveraging pre-trained information

어떤 데이터셋으로 학습시킨 모델을 다른 데이터셋을 이용하는 task에 활용한다.

## transfer learning

지도학습을 데이터가 많이 필요하다. 그러나 데이터셋을 구축하는 비용은 매우 비싸고 그 퀄리티도 보장되지 않는다. - 여러 사람에 의해 annotation되면 각자의 편향이 반영되기 마련.

transfer learning은 큰 데이터셋에서 학습된 모델을 이용하여 작은 데이터셋이나 다른 task에서 경제적으로 합리적인 성능을 끌어낼 수 있다.

### 비슷한 데이터셋은 비슷한 패턴을 공유한다.

비슷한 데이터셋끼리는 비슷한 패턴을 공유하므로 한 데이터셋에서 학습한 지식을 다른 데이터셋에도 적용할 수 있다.

### 1. pre-trained 모델의 지식을 그대로 사용한다.

![approach 1](/assets/images/post/220309/boostcamp-CV-2/transfer_learning_approach_1.jpg){: .align-center}

데이터셋이 비슷한 경우에 head만 학습시키므로 적은 데이터셋으로도 괜찮은 성능을 달성할 수 있다.

### 2. pre-trained 모델의 지식을 현재 데이터셋에 적응하도록 학습시킨다.

![approach 2](/assets/images/post/220309/boostcamp-CV-2/transfer_learning_approach_2.jpg){: .align-center}

현재 데이터셋이 적지 않다면 pre-trained 모델을 현재 데이터셋에 적응하도록 학습시킬 수 있다. 이때 pre-trained 모델 부분은 학습률을 낮게 설정하여 이전 데이터셋으로부터의 지식을 크게 변형하지 않으면서 현재 데이터셋에 적응하도록 한다.

## Knowledge distillation

Teacher-student learning. 큰 모델의 지식을 작은 모델에게 전달한다. 주로 모델 경량화에 이용하고 수도라벨링에서 사용한다.

### 1. unlabeled 데이터로 Teacher의 행동을 Student에 가르친다.

![teacher and student - unlabeled](/assets/images/post/220309/boostcamp-CV-2/teacher_student_unlabeled.jpg){: .align-center}

unlabeled 데이터를 사용한다. 입력된 데이터에 대해서 모델이 실제로 어떤 출력을 내야하는지는 전혀 상관하지 않는다. 그저 Student 모델의 출력이 Teacher 모델의 출력과 같아지도록, Student 모델이 Teacher 모델의 행동을 모방하도록 한다.

### 2. labeled 데이터로 Teacher의 행동을 Student에 가르친다.

![teacher and student - labeled](/assets/images/post/220309/boostcamp-CV-2/teacher_student_labeled.jpg){: .align-center}

라벨이 있는 데이터가 있다면 Student 모델이 Teacher 모델의 출력을 모방하도록 하면서도 실제 라벨도 맞출 수 있도록 학습시킨다.

distilation loss(KL div. loss)에서는 주어진 데이터의 실제 라벨은 신경쓰지 않는다. Teacher 모델이 전혀 접해보지 못한 도메인의 데이터이더라도 distilation loss의 목적은 Teacher의 행동을 Student가 모방하도록 하는 것이므로 정답을 맞추던 못 맞추던 중요하지 않다.

#### hard label & soft label

* Hard label (one-hot vector)
  * 주로 데이터셋의 타겟을 표현하는 방식. 각 클래스가 정답인지 아닌지 표현.
* Soft label (`softmax(logit)`)
  * 주로 모델의 출력
  * 각 클래스에 대응되는 값들의 경향성이 모델이 주어진 데이터를 어떻게 생각하는지를 나타내므로 모델이 가진 "지식"이라고 생각할 수 있다.

$$
\begin{matrix}
\begin{bmatrix}
\text{Bear} \\ \text{Cat} \\ \text{Dog}
\end{bmatrix} =
\begin{bmatrix}
0 \\ 1 \\ 0
\end{bmatrix} &
\begin{bmatrix}
\text{Bear} \\ \text{Cat} \\ \text{Dog}
\end{bmatrix} =
\begin{bmatrix}
0.14 \\ 0.8 \\ 0.06
\end{bmatrix} \\
\text{Hard Label} & \text{Soft Label} 
\end{matrix}
$$

#### softmax with temperature ($T$)

소프트맥스 연산은 모델의 출력에서 큰 값과 작은 값의 차이를 크게 만드는 연산이다 (합은 1). temperature 항 $T$는 큰 값과 작은 값의 차이를 조절한다.

* Normal softmax (= Hard Prediction)

$$
\frac{\text{exp}(z_i)}{\sum_j\text{exp}(z_j)}
$$

$$\text{softmax}(5, 10) = (0.0067, 0.9933)$$

출력값들의 차이를 크게 만든다. 그 과정에서 모델의 "지식"이 뭉게진다. (출력값의 차이도 모델이 입력을 어떻게 생각하는지를 나타내는 요소가 아닐까?)

* Softmax with temperature $T$ (= Soft Prediction)

$$
\frac{\text{exp}(z_i+T)}{\sum_j\text{exp}(z_j+T)}
$$

$$\text{softmax}_{(T=100)}(5, 10) = (0.4875, 0.5125)$$

일반 softmax보다 출력값들의 차이가 작다. 하나의 답을 찾기보다는 출력간의 미묘한 차이에 집중하기 위해 사용한다. "지식"이 크게 상하지않아 Student가 Teacher의 분포를 재현하는데 유리하다.

무슨 느낌인지는 알겠는데 정확히는 모르겠다.
{: .notice}

# Leveraging unlabeled dataset for training

## Semi-supervised learning

**semi-supervised learning = unsupervised(unlabeled data) + supervised(labeled data)**

label이 있는 데이터는 적지만 없는 데이터는 넘쳐난다. 그럼 이 데이터들을 유용하게 사용할 수 없을까-

### pseudo-labeling

![pseudo-labeling](/assets/images/post/220309/boostcamp-CV-2/pseudo_labeling.jpg){: .align-center}

1. 적은 수의 labeled 데이터셋을 통해 모델을 학습시킨다.
2. 학습된 모델에 unlabeled 데이터셋을 입력하여 나온 출력을 unlabeled 데이터셋의 label로 삼는다. = **pseudo-labeling**
3. labeled 데이터셋과 pseudo-labeled 데이터셋을 모두 사용해 새로운 모델을 학습시킨다.

## Self-training

![noisy student](/assets/images/post/220309/boostcamp-CV-2/noisy_student.jpg){: .align-center}

### Noisy Student
* 2019 ImageNet SOTA
* Augmentation + Teacher-Student + semi-supervised learning
1. 이미지넷 데이터셋으로 Teacher 모델 학습
2. 학습된 Teacher 모델을 통해 pseudo-labeled 데이터셋 생성
3. 이미지넷 데이터셋과 pseudo-labeled 데이터셋으로 Student 모델 학습
4. 학습된 Student 모델을 Teacher 모델로 삼고 새로운 Student 모델로 조금 더 큰 모델을 생성.
5. 2~4 과정을 반복한다.


슬라이드가 더 있는데 수업에서는 다루지 않으셨음. 나중에 공부해보자.
{: .notice}
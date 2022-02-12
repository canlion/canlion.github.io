---
title: "DL Basic 03.01] 주요한 용어"
excerpt: "주요한 용어"

categories:
  - boostcamp DL Basic
tags:
  - [boostcamp, boostcamp DL Basic]

toc: true
toc_sticky: true

date: 2022-02-08T21:11:00+09:00
last_modified_at: 2022-02-08T21:11:00+09:00
---

# 주요 용어들

## Generalization

![generalization gap](/assets/images/post/220208/boostcamp-DL-Basic-03.01.01/generalization_gap.png){: .align-center}

일반화 성능. 학습된 모델이 처음 본 데이터에 얼마나 잘 대처하는지를 의미. 단순히 학습 loss의 감소가 좋은 모델을 의미하지 않는다. 아무리 학습 loss가 낮아 학습 데이터를 잘 처리한다해도 처음 보는 데이터에 제대로 작동하지않으면 곤란함.

**generalization 성능이 좋다.** - 처음 본 데이터에도 학습 데이터와 비슷한 수준의 성능을 기대할 수 있겠다.

하지만 모델이 학습 데이터조차도 제대로 학습하지 못한다면 generalization 성능은 무의미함.

## Underfitting vs. Overfitting

![underfitting vs overfitting](/assets/images/post/220208/boostcamp-DL-Basic-03.01/underfitting_overfitting.png){: .align-center}

* Underfitting: 모델이 너무 간단하거나 학습이 부족해서 학습데이터에 대한 성능이 나쁨.
* Overfitting: 학습데이터에 점차 메모라이징하며 완벽히 fitting되기 시작함. 테스트 데이터에 대한 성능이 나빠지며 테스트 loss가 점차 커짐.

## Cross-validation

학습 데이터를 $k$개로 분할 후 분할된 데이터셋 중 하나를 validation 셋으로, 나머지를 학습 데이터셋으로 사용한 모델 학습을 $k$번 반복하여 결과들을 종합해 최적의 하이퍼파라미터, 모델 아키텍쳐를 찾는다. (k-fold cross validation). 고정된 validation 셋을 사용하는 것보다 더 다양한 조합의 validation 셋을 경험하여 더 일반화 성능이 좋은 모델을 구성할 수 있다.

## Bias and Variance

![variance vs bias](/assets/images/post/220208/boostcamp-DL-Basic-03.01/variance_bias.png){: .align-center}

* Bias: 출력의 평균과 타겟의 차이
  * 바이어스가 작다면 평균적인 출력은 타겟과 근접
* Variance: 비슷한 입력에 대한 출력의 일관성
  * 분산이 작다면 비슷한 입력은 비슷한 출력을 갖는다.

### Bias and Variance Tradeoff

![variance](/assets/images/post/220208/boostcamp-DL-Basic-03.01/variance.png){: .align-center}

$\mathcal{D}=\{(x_i, t_i)\}^N_{i=1}, \text{where } t=f(x)+\epsilon \text{ and } \epsilon ~ \mathcal{N}(0, \sigma^2)$, 노이즈 $\epsilon$가 섞인 데이터가 있다. MSE를 loss로 삼아 최소화한다고 하면 MSE는 세 개의 항으로 볼 수 있다.

$$
\begin{matrix}
\mathbb{E}\left[(t-\hat{f})^2\right] & = & \mathbb{E}\left[(t-f+f-\hat{f})^2\right]&&&&\\
& = & \mathbb{E}\left[(f-\mathbb{E}\left[\hat{f}\right]^2)^2\right] & + & \mathbb{E}\left[(\mathbb{E}\left[\hat{f}\right]-\hat{f})^2\right] & + & \mathbb{E}\left[\epsilon\right] \\
& & \text{Bias}^2 & & \text{variance} & & \text{noise}
\end{matrix}
$$

$\hat{f}$가 커지냐 작아지냐에 따른 문제인가 싶었는데 위키를 보니 $\hat{f}$의 복잡도에 대한 해석을 해야한다.

[wiki: 편향-분산 트레이드오프](https://ko.wikipedia.org/wiki/%ED%8E%B8%ED%96%A5-%EB%B6%84%EC%82%B0_%ED%8A%B8%EB%A0%88%EC%9D%B4%EB%93%9C%EC%98%A4%ED%94%84){: .btn .btn--info}

* 노이즈: 데이터에 내제된 줄일 수 없는 오류
* 편향이 높은 모델: 단순하고 표현력이 작은 모델
  * 학습 데이터의 패턴을 충분히 잡아내지 못해 언더피팅 문제 발생
* 분산이 높은 모델: 복잡하고 표현력이 높은 모델
  * 학습 데이터의 노이즈, 잘못된 데이터까지 데이터의 패턴으로 잡아내어 오버피팅 문제 발생

복잡한 모델을 사용하면 분산항은 커지되 편향항은 작아지고, 단순한 모델을 사용하면 분산항은 작아지되 편향항은 증가한다. 적절한 복잡도의 모델을 선택하는 것이 중요하다.


## bootstrapping

학습 데이터셋의 일부를 샘플링하여 구성한 데이터셋으로 모델 생성을 반복하여 다수의 모델을 함께 사용하는 방식. (ex. 학습 데이터셋의 80%를 무작위 샘플링)

### bagging

![bagging](/assets/images/post/220208/boostcamp-DL-Basic-03.01/bagging.png){: .align-center}

앙상블. 부트스트래핑으로 학습된 다수의 모델을 병렬적으로 사용하는 방식. 하나의 입력에 대한 다수 모델의 출력을 종합하는 방식으로 동작. 일반적으로 단일모델보다 좋은 성능을 보인다.

### boosting

![boosting](/assets/images/post/220208/boostcamp-DL-Basic-03.01/boosting.png){: .align-center}

데이터 샘플링, 모델 학습 과정을 반복하되 이전 모델 학습 과정에서 제대로 학습하지 못한 데이터에 가중치를 부여하여 다음 모델 학습에 반영한다. 모델 학습이 반복되면서 앞의 모델들이 제대로 학습하지 못한 데이터를 점차 뒤의 모델들은 제대로 학습한다. 이 과정에서 생성된 모델들을 weak learner라 하고, 최종적으로 각 weak learner에 가중치를 부여하여 하나의 모델, strong learner로 동작하도록 한다.
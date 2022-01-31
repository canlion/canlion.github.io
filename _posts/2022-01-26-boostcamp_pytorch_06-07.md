---
title: "pytorch_06-07. 모델 불러오기 / 모니터링 도구"
excerpt: "transfer learning, state_dict / tensorboard, w&b"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-26 10:19
last_modified_at: 2022-01-26 10:19
---

# 모델 불러오기
학습된 백본 모델을 다른 데이터에 적응시키는 파인튜닝이 주류.

## 학습 결과의 공유

* `torch.save()`
  * 학습 결과를 저장
    * (모델 아키텍쳐 & 파라미터) 또는 (파라미터) 저장
    * 타인과 모델 공유 가능
  * `state_dict`: 모델의 파라미터와 버퍼를 담은 딕셔너리
    ```python
    torch.save(model.state_dict(), 'model_params.pt')  # 파라미터만 저장
    new_model_0 = Model()
    new_model_0.load_state_dict('model_params.pt')  # 새로 생성한 모델에 파라미터 로드

    torch.save(model, 'model.pt')  # 아키텍쳐와 파라미터 저장
    new_model_1 = torch.load('model.pt')  # 모델 아키텍쳐와 파라미터 로드
    ```
* checkpoints
  * 모델 학습 중간중간 모델을 저장하여 최고 성능의 모델 선택
  * loss, metrice을 지속적으로 확인
  * ex)
    ```python
    ckp_path = f'checkpoint_{epoch}_{epoch_loss:.4f}_{metric:.4f}.pt'
    # 한눈에 체크포인트 결과를 알 수 있는 파일명
    
    torch.save({
      'epoch': epoch,
      'model_state_dict': model.state_dict(),
      'optimizer_state_dict': optimizer.state_dict(),
      'loss': epoch_loss,
    }, ckp_path)

    ckp = torch.load(ckp_path)
    model.load_state_dict(ckp['model_state_dict'])
    optimizer.load_state_dict(ckp['optimizer_state_dict'])
    ...
    ```

## transfer learning
학습된 모델(pretrained model)을 새로운 타입의 데이터셋에 적응시킨다. 대용량 데이터셋으로 학습시킨 모델을 사용하면 모델의 성능을 향상시킬 수 있음. 학습된 모델의 백본에 필요한 레이어를 덧붙여 새로운 데이터셋으로 학습시킨다.

* pretrained models
  * CV
    * torchvision
    * [repository: pytorch-image-models](https://github.com/rwightman/pytorch-image-models){: .btn .btn--info}
    * [repository: segmentation_models](https://github.com/qubvel/segmentation_models.pytorch){: .btn .btn--info}
  * NLP
    * [HuggingFace](https://huggingface.co/models){: .btn .btn--info}


### freezing
pretrained model 활용시에 모델 일부분의 가중치를 갱신하지않고 고정시킨다.

<br>

# 모니터링 도구

## TensorBoard
* Tensorflow의 시각화 도구.
* 그래프, metric, 학습 결과 시각화 지원.
* pytorch에서도 사용할 수 있음.
* 스칼라, 모델 그래프, 가중치 히스토그램, 이미지(groundtruth & prediction), mesh(3D 데이터 표현)
* 플롯이나 PR 커브 등을 그릴 수도 있다. 임베딩도 가능.

```python
from torch.utils.tensorboard import SummaryWriter

writer = SummaryWriter(log_dir)
...
writer.add_scalar('Loss/train', train_loss, epoch)  # Loss 카테고리의 train 항목
...
writer.flush()
...
writer.close()

# jupyter에서 tensorboard 실행
%load_ext tensorboard
%tensorboard --logdir {log_dir}
```

```python
# 같은 카테고리, 항목에 여러 값을 한번에 입력
writer.add_scalar('Loss', {'loss_0': {loss_0}, 'loss_1': {loss_1}})
```

```python
# histogram
# 가중치를 추적할때 주로 사용
for i in range(10):
    x = np.random.normal(size=1000)
    writer.add_histogram('distribution', x, i)
```

```python
# 이미지
# 배치 단위로 표시할 수 있다.
imgs = np.random.randint(0, 256, (16, 3, 100, 100), np.uint8)  # ch first??
writer.add_images('batch_imgs', imgs, epoch)
```

```python
# 하이퍼파라미터와 메트릭 등록 - 테이블로 보여줌
# 학습 종료 후 기록위해 등록하면 될 듯?
writer.add_hparams(
  {'lr':1e-3, 'batch_size': 32},
  {'hparam/acc': acc, 'hparam/loss': loss}
)
```

## Weight & Biases
* 머신러닝 실험을 지원하는 상용 도구
* MLOps의 대표적인 툴
* 협업, 코드 버전 관리, 실험 결과 기록 등

아주 좋아보인다. 개인은 무료.

<br>

# 메모

* torch-summary: keras 스타일로 모델 정보를 출력.
* torchvision.utils.make_grid(images): 이미지 배치를 그리드로 출력해줌.
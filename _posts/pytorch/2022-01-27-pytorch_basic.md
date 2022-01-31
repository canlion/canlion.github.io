---
title: "pytorch 튜토리얼"
excerpt: "간단한 모델 생성과 학습, 저장"

categories:
  - pytorch
tags:
  - [pytorch]

toc: true
toc_sticky: true

date: 2022-01-27 17:43
last_modified_at: 2022-01-27 17:43
---
[link:파이토치 튜토리얼](https://pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html){: .btn .btn--success}

# 간단한 학습의 구성 요소
* 데이터셋, 데이터로더 - 데이터 로드 및 전처리, 미니 배치 생성
* 모델
* 학습 & 테스트 과정 - 모델 업데이트, 모델 평가 지표 계산

```python
import pickle  # cifar10 로드
from pathlib import Path

import numpy as np  # 데이터 수정
import torch
from torch import nn
import torchvision
import torchvision.transforms.functional as F
from torchvision.transforms import ToTensor
from torchsummary import summary  # 모델의 형태, 파라미터 수 등의 상세 출력
```

## 데이터셋, 데이터로더
* 데이터 순회를 위한 툴
  * `torch.utils.data.Dataset`
    * 리스트처럼 인덱스를 입력받아 인덱스에 대응되는 데이터 샘플을 반환하는 객체.
  * `torch.utils.data.DataLoader`
    * `torch.utils.data.Dataset`를 랩핑하는 객체로 데이터셋의 샘플을 셔플하거나 배치화하고, 그런 작업들을 병렬 처리하는 역할을 담당.
  * `torchvision.transforms` - 이미지 전처리
    * 전처리 관련 함수와 모듈들

**CIFAR10 데이터**
* [link:CIFAR10](http://www.cs.toronto.edu/~kriz/cifar.html){: .btn .btn--success}
* (32, 32, 3) RGB 이미지

```python
def unpickle(file):
    """CIFAR10 데이터 로드 함수

    자세한 내용은 사이트 참조
    
    Args:
        file (str): 배치 파일 경로
      
    Return:
        dict: key=(b'data', b'label')를 갖는 딕셔너리
    """
    import pickle
    with open(file, 'rb') as fo:
        dict = pickle.load(fo, encoding='bytes')
    return dict


class CifarDatasets(torch.utils.data.Dataset):
    """dataset 객체
    
    __len__, __getitem__ 메소드가 포함되어야한다.

    * __len__: 데이터 샘플의 수 반환
    * __getitem__: 인덱스를 입력받아 인덱스에 해당하는 데이터 샘플 반환
    """
    def __init__(self, path_list, meta_path=None, transforms=None):
        # cifar 데이터 load
        batch_list = [unpickle(str(p)) for p in path_list]
        imgs_shape = (-1, 3, 32, 32)
        self.X = np.concatenate([batch[b'data'].reshape(imgs_shape) for batch in batch_list], axis=0)
        self.X = self.X.transpose(0, 2, 3, 1)  # (N, C, H, W) -> (N, H, W, C)
        self.y = np.concatenate([batch[b'labels'] for batch in batch_list], axis=0)
        # X와 y를 보관하고 __getitem__에서 입력받은 인덱스로 인덱싱하여 반환
        
        # 데이터 전처리 유닛
        self.transforms = transforms
        
        # 라벨에 대응하는 클래스 이름
        self.classes = None
        if meta_path:
            self.classes = unpickle(meta_path)[b'label_names']
        
    def __len__(self):
        """데이터 길이 반환"""
        return len(self.X)
    
    def __getitem__(self, idx):
        """입력된 인덱스에 해당되는 샘플 반환. 전처리 유닛이 있다면 전처리 수행"""
        X, y = self.X[idx], self.y[idx]
        if self.transforms:
            X = self.transforms(X)
        return X, y


# 데이터 경로 설정
cifar_path = Path('/mnt/hdd/datasets/cifar-10-python/cifar-10-batches-py')
train_batch_paths = cifar_path.glob('data_batch_*')
test_batch_paths = cifar_path.glob('test_batch')
meta_path = str(cifar_path / 'batches.meta')

# 데이터 전처리 유닛 설정
transforms = torchvision.transforms.Compose([
    ToTensor(),  # (H,W,C) [0, 255] -> (C, H, W) [0, 1.]
])  

# 데이터셋 생성 - 단순히 데이터를 읽어오는 역할
train_dataset = CifarDatasets(train_batch_paths, meta_path, transforms)
test_dataset = CifarDatasets(test_batch_paths, meta_path, transforms)

# 데이터로더 생성 - 읽어온 데이터를 배치처리하거나 섞어주는 역할
train_dataloader = torch.utils.data.DataLoader(
    dataset=train_dataset,  # 데이터셋 전달
    batch_size=64,
    shuffle=True,
    num_workers=4,  # 데이터 로드를 병렬처리할 서브프로세스의 수
    drop_last=True,  # 마지막 미니배치의 샘플이 배치사이즈보다 작다면 버린다.
)

test_dataloader = torch.utils.data.DataLoader(
    dataset=test_dataset,
    batch_size=128,
    shuffle=False,
    num_workers=4,
    drop_last=False,
)
```


## 모델

`torch.nn.Module`을 상속하는 클래스를 통해 레이어를 정의하거나, 레이어들을 속성으로 선언해 모델을 정의할 수 있다.

```python
# 연산 장치 선택. GPU 이용이 가능하다면 'cuda'.
device = 'cuda' if torch.cuda.is_available() else 'cpu'


class CRB(nn.Module):
    """간단한 3x3Conv-ReLU-BN 레이어
    
    torch.nn.Module를 상속하여 레이어를 정의한다.
    forward 메소드가 필요하다.

    * forward: 레이어의 연산을 정의한다.
    """
    def __init__(self, in_ch, out_ch):
        super().__init__()
        
        # Sequential: 포함된 레이어, 모듈 연산을 순서대로 수행한다.
        # 앞의 레이어의 출력은 다음 레이어의 입력이 된다.
        self.seq = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, 3, padding='same'),
            nn.ReLU(),
            nn.BatchNorm2d(out_ch),
        )
    
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        """레이어의 연산 정의"""
        return self.seq(x)


class SimpleCNN(nn.Module):
    """레이어를 모아 모델 정의"""
    def __init__(self):
        super().__init__()
        self.seq = nn.Sequential(
            CRB(3, 32),
            nn.MaxPool2d(2, 2),  # 16
            CRB(32, 64),
            nn.MaxPool2d(2, 2),  # 8
            CRB(64, 128),
            nn.MaxPool2d(2, 2),  # 4
            CRB(128, 256),
            nn.Conv2d(256, 10, 4),  # 1
        )
        
    def forward(self, x):
        logits = self.seq(x).squeeze(2).squeeze(2)
        return logits
    

model = SimpleCNN().to(device)  # 모델을 GPU로 보낸다.
summary(model, (3, 32, 32));  # 모델 상세 / (3, 32, 32)는 입력의 쉐이프 (배치 차원 제외)
```

## 학습 & 테스트 과정

### 학습 요소
#### loss, optimizer
* loss: 모델 학습 지표. 가중치를 어떤 방향으로 갱신할지를 알려주는 함수.
* optimizer: 모델 가중치를 갱신하는 알고리즘을 수행하는 객체.

```python
loss_fn = nn.CrossEntropyLoss()  # 분류 문제
optimizer = torch.optim.SGD(model.parameters(), lr=1e-2)  # 확률적경사하강법 적용
# 학습률을 점차 감소시키기위한 스케쥴러
scheduler = torch.optim.lr_scheduler.ExponentialLR(optimizer, gamma=0.9)
```

#### 한 epoch의 학습 과정
```python
def train(dataloader, model, loss_fn, optimizer):
    size = len(dataloader.dataset)
    model.train()  # 모델을 학습 모드로 설정 - BN 레이어등에 영향을 미친다.
    for batch_idx, (X, y) in enumerate(dataloader):
        X, y = X.to(device), y.to(device)  # 데이터를 모델이 있는 GPU로 보낸다.
        
        pred = model(X)  # 모델 출력
        loss = loss_fn(pred, y)  # 모델 출력과 라벨로 손실함수 계산
        
        optimizer.zero_grad()  # 이전 데이터 샘플의 그래디언트를 초기화시킴
        loss.backward()  # 그래디언트 계산
        optimizer.step()  # 가중치 계산
        
        if (batch_idx+1) % 100 == 0:
            loss, current = loss.item(), batch_idx * len(X)
            print(f'loss: {loss:>7f} [{current:>5d}]/{size:>5d}]')
```
#### 한 epoch의 테스트 과정
```python
def test(dataloader, model, loss_fn):
    size = len(dataloader.dataset)
    num_batches = len(dataloader)
    model.eval()  # 모델을 evaluation 모드로 설정. 
    test_loss, correct = 0, 0
    with torch.no_grad():  # 그래디언트 계산을 하지 않도록 설정 - 메모리 사용량 감소
        for X, y in dataloader:
            X, y = X.to(device), y.to(device)
            pred = model(X)
            test_loss += loss_fn(pred, y).item() * X.shape[0]
            # 예측된 클래스와 라벨이 일치하는 수를 누적한다.
            correct += (pred.argmax(1) == y).type(torch.float).sum().item()
    test_loss /= size  # 샘플당 loss
    correct /= size  # 정확도
    print(f'Test Error: \n Accuracy: {(100*correct):>0.1f} %, Avg loss: {test_loss:>8f}')
```

#### 학습 & 테스트 루프
```python
epochs = 20  # 데이터셋 순회 횟수
for epoch in range(epochs):
    print(f'Epoch {epoch+1:02d}\n--------------------------------')
    train(train_dataloader, model, loss_fn, optimizer)  # 학습
    test(test_dataloader, model, loss_fn)  # 테스트
    scheduler.step()  # 학습률 조정 - epoch이 지날때마다 감소시킴
    print(f' lr: {scheduler.get_lr()}\n')
print('Done')

# 01 epoch - Accuracy: 58.4 %, Avg loss: 1.204955
# 20 epoch - Accuracy: 75.8 %, Avg loss: 0.872336
```

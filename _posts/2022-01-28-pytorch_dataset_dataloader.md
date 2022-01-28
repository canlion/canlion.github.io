---
title: "pytorch 튜토리얼 - Dataset & Dataloader"
excerpt: "dataset & dataloader"

categories:
  - pytorch
tags:
  - [pytorch]

toc: true
toc_sticky: true

date: 2022-01-28 18:54
last_modified_at: 2021-01-28 18:54
---

# Dataset과 Dataloader

* 파이토치의 데이터 순회
  * `Dataset`: 데이터 샘플과 라벨을 보관
  * `DataLoader`: `Dataset`을 이용해 이터러블 객체 생성

## 파이토치에서 제공하는 데이터셋
파이토치는 퍼블릭 이미지, 텍스트, 오디오 데이터셋을 미리 정의하여 제공하여 쉽게 가져다 쓸 수 있음.

ex) Fashion-MNIST
```python
from torchvision import datasets
from torchvision.transforms import ToTensor

train_data = datasets.FashionMNIST(
    root='/mnt/hdd/datasets/',  # 데이터셋 저장 위치. 해당 경로에 FashionMNIST 디렉토리 생성
    train=True,  # train data or test data
    download=True,  # root 경로에 데이터셋이 없다면 데이터셋 다운로드
    transform=ToTensor(),  # 전처리 유닛
)

print(train_data[0][0].shape, train_data[0][0].dtype)  # 샘플 shape, dtype
print(train_data[0][1])  # 라벨
```
출력
```output
torch.Size([1, 28, 28]) torch.float32
9
```

## custom 데이터셋 클래스 정의
`Dataset`을 상속하고 `__init__`, `__len__`, `__getitem__` 메소드를 정의해야한다.

예를 들어 몇 장의 이미지와 이미지 파일명과 라벨을 담은 csv가 있다. (출처: 이미지넷 데이터셋)
![dataset_example](/assets/images/post/220128/pytorch_dataset_dataloader/dataset_example.png){: .align-center}

```python
import os
import math
from pathlib import Path

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from PIL import Image

import torch
from torch.utils.data import Dataset, DataLoader
from torchvision.transforms import Compose, ToTensor, RandomResizedCrop

class CustomDataset(Dataset):
    def __init__(self, data_dir, csv_path, transforms):
        data_csv = pd.read_csv(str(csv_path), header=None)
        
        self.data_dir = data_dir
        self.img_names = data_csv.iloc[:, 0].values
        self.label = data_csv.iloc[:, 1].values.astype(np.int32)
        self.transforms = transforms
        
    def __len__(self):
        """전체 데이터 샘플 수 반환"""
        return len(self.label)
    
    def __getitem__(self, idx):
        """주어진 인덱스에 해당하는 데이터 샘플, 라벨 반환"""
        img_path = os.path.join(self.data_dir, self.img_names[idx])
        img = Image.open(img_path)
        label = self.label[idx]
        
        if self.transforms:  # 전처리
            img = self.transforms(img)
            
        return img, label

# 전처리 과정 정의
transforms = Compose([
    RandomResizedCrop(224, [1., 1.], [1., 1.]),
    ToTensor(),  # 텐서로 변환
])

dataset = CustomDataset(data_dir, csv_path, transforms)

# 이미지 & 라벨 출력
def draw_grid(data, cols):
    len_ds = len(data)
    fig, axes = plt.subplots(math.ceil(len_ds / cols), cols)
    for i, (img, label) in enumerate(dataset):
        ax = axes[i//cols][i%cols]
        ax.imshow(img.numpy().transpose(1, 2, 0))
        ax.set_title(f'label: {label}')
        ax.axis('off')
    plt.show()

  
draw_grid(dataset, 4)
```
![image_grid](/assets/images/post/220128/pytorch_dataset_dataloader/image_grid.png){: .align-center}


## DataLoader를 이용한 데이터셋 처리
데이터셋은 데이터 샘플 하나하나를 로드한다. 모델을 트레이닝할때는 개별 데이터 샘플의 로딩에서 끝나지않고 데이터 샘플 로딩을 병렬처리하고, 다수의 데이터 샘플을 미니배치로 구성하고, 데이터셋을 한 번 순회한 후에 데이터셋의 순서를 섞어주어야한다. `DataLoader`는 바로 그런 역할을 담당한다.

```python
dataloader = DataLoader(
    dataset=dataset,  # Dataset 객체
    batch_size=4,  # 미니배치 크기
    shuffle=True,  # 한 번 순회를 마치면 셔플
    num_workers=4,  # 병렬처리에 사용할 서브프로세스 수
)

dataloader_iter = iter(dataloader)
imgs, labels = next(dataloader_iter)
print(imgs.shape, labels.shape, label)
```
```output
# 3차원의 이미지를 stack하여 4차원이 된다.
torch.Size([4, 3, 224, 224]) torch.Size([4]), tensor([4, 6, 0, 5], dtype=torch.int32)
```

`DataLoader`는 데이터셋을 받아 데이터셋의 데이터 샘플 로드를 병렬처리하고, 로딩된 샘플을 쌓아 미니배치를 생성한다. `iter`를 통해 이터레이터를 생성할때마다 데이터가 셔플된 이터레이터를 반환된다.
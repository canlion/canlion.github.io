---
title: "pytorch 튜토리얼 - Tensor"
excerpt: "Tensor"

categories:
  - pytorch
tags:
  - [pytorch]

toc: true
toc_sticky: true

date: 2022-01-28 13:45
last_modified_at: 2021-01-28 13:45
---

# Tensor
[링크: Tensors](https://pytorch.org/tutorials/beginner/basics/tensorqs_tutorial.html){: .btn .btn--info}

파이토치에서 모델의 입출력, 파라미터를 인코딩하는 객체. 넘파이 ndarray와 유사하지만 하드웨어 가속기에서 연산이 가능하다.

## tensor 생성
* `torch.tensor(data)`
* `torch.from_numpy(numpy.ndarray)` & `torch.tensor.numpy()`:
  * `numpy.ndarray`와 메모리를 공유한다. - 데이터 복사 없이 텐서 생성
  * ```python
    ndarray_0 = np.arange(4)
    tensor_0 = torch.tensor(ndarray_0)
    ndarray_0[0] = 5
    print(ndarray_0)  # [5 1 2 3]
    print(tensor_0)   # tensor([0, 1, 2, 3]) 

    ndarray_1 = np.arange(4)
    tensor_1 = torch.from_numpy(ndarray_1)
    ndarray_1[0] = 10
    print(ndarray_1)  # [10  1  2  3]
    print(tensor_1)   # tensor([10,  1,  2,  3]) - 메모리 공유
    ```
  * ```python
    tensor = torch.tensor([1, 2, 3])
    ndarray = tensor.numpy()

    tensor[0] = 5
    print(tensor)  # tensor([5, 2, 3])
    print(ndarray) # [5, 2, 3]  - 메모리 공유
    ```

## 속성
```python
print(f"""shape: {tensor_0.shape}
data type: {tensor_0.dtype}
device: {tensor_0.device}""")
# device: 현재 텐서가 할당된 장치
```
```output
출력
shape: torch.Size([4])
data type: torch.int64
device: cpu
```

## 텐서 할당
기본적으로 텐서는 cpu에 생성되는데(시스템 메모리) gpu로 옮기 수 있다. 다만 장치간 텐서 복사는 시간과 메모리가 많이 필요하다.
```python
if torch.cuda.is_available():  # cuda 장치가 사용가능하면 (ex - gpu)
    tensor = tensor.to('cuda') # cuda 장치로 텐서를 옮긴다.
```
```python
# mem: 5.10G/31.3G
# gpu mem: 637 / 11016 MB

# 텐서 생성
a = torch.randint(0, 255, size=(100, 1000, 1000))
# mem: 5.82G/31.3G  - 약 0.7G 증가
# gpu mem: 666 / 11016 MB

# GPU로 복사
a.to('cuda')
# mem: 7.18G/31.3G  - gpu를 컨트롤하기위한 뭔가가 실행?
# gpu mem: 2169 / 11016MB

# a 참조 제거
del(a)
# mem: 6.43G/31.3G  - 약 0.7G 감소: cpu상의 텐서는 제거
# gpu mem: 2160 / 11016 MB  - gpu 상의 텐서는 유지된다.
```

## 텐서 연산

### in-place op: +=, /=, ...
torch.tensor는 mutable한 객체이므로 in-place 연산을 수행하면 tensor 값이 변경된다.
```python
print('*mutable - list')
a = [1, 2, 3]
b = [4, 5, 6]
print(id(a), a)
a += b
print(id(a), a)

print('\n*immutable - int')
x = 5
print(id(x), x)
x += 5
print(id(x), x)

print('\n*mutable - torch.tensor')
c = torch.tensor([1, 2, 3])
print(id(c), c)
c += 5
print(id(c), c)
```
```output
*mutable - list
140342474660992 [1, 2, 3]
140342474660992 [1, 2, 3, 4, 5, 6]

*immutable - int
9789120 5
9789280 10  # 객체 새로 생성

*mutable - torch.tensor
140342474040128 tensor([1, 2, 3])
140342474040128 tensor([6, 7, 8])
```

in-place 연산자 외에도 torch 자체의 in-place 연산이 존재. 메소드 끝에 언더바가 붙는다.
```python
a = torch.tensor([5, 6, 7])
a.add_(5)
print(a)
```
```output
tensor([10, 11, 12])
```

### torch.tensor().item()
`torch.tensor` 원소가 하나인 경우 `item` 메소드는 파이썬 객체를 반환한다.
```python
item = torch.tensor([[[5]]]).item()
print(item, type(item))
```
```output
5 <class 'int'>
```
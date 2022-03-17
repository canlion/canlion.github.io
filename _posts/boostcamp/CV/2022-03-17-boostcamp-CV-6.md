---
title: "CV] AutoGrad"
excerpt: ""

categories:
  - boostcamp CV
tags:
  - [boostcamp, boostcamp CV]

toc: true
toc_sticky: true

date: 2022-03-17T13:49:00+09:00
last_modified_at: 2022-03-17T13:49:00+09:00
---

# AutoGrad

* **Auto**matic **Grad**ient calculating API
* forward pass에서 계산 그래프를 통해 연산을 기록하고 backward pass에서 계산 그래프를 거슬러올라가며 gradient 계산

## pytorch에서는...

```python
...
y = x * 3

grads = torch.tensor([100., .1])
y.backward(grads)

print(x.grad)
>>> tensor([300.0000, 0.3000])
```

* `backward` 호출시에 상위 레이어로부터의 gradient를 인자로 넣을 수 있다. 
  * 체인 룰
* 만일 `x`의 `requires_grad`가 `False`라면 `backward` 호출시에 에러 발생
  * gradient를 가질 수 없는 요소가 끼어있으면 그 요소 너머로는 역전파가 불가능하다.
* `backward`를 두 번 연속으로 호출하면 에러 발생
  * `backward` 호출 후에는 자원 절약을 위해 계산 그래프와 기타 `backward` 호출시에 필요한 요소들을 지워버림
  *  `y.backward(retain_graph=True)`: `backward`를 위한 요소들을 유지하므로 연속으로 `backward` 호출 가능

```python
print(y)
>>> tensor([...], grad_fn=<MulBackward0>),
```

* 텐서는 `requires_grad=True`인 경우 `backward`를 위한 함수를 포함하고 있다.

### hook

* `torch.nn.Module`에 등록되어 forward pass에서 입력과 출력, backward pass에서 그래디언트를 조작하는 함수
* `register_forward_hook`
  * forward pass에서 작동하는 hook 등록
  * hook parameter (self 생략)
    * `input`: 레이어 입력
    * `output`: 레이어 출력
* `register_forward_pre_hook`
  * forward pass에서 레이어 연산 전에 작동하는 hook 등록
  * hook parameter (self 생략)
    * `input`: 레이어 입력
* `register_backward_hook`
  * backward pass에서 작동하는 hook 등록
  * hook parameter (self 생략)
    * `grad_input`: 입력에 대한 gradient - 레이어에서 다음 레이어로 전달되는 값
    * `grad_output`: 출력에 대한 gradient
  * return이 있다면 `grad_input`을 대체한다.

* hook 제거
  ```python
  net = CNN()  # nn.Module / network
  handle = net.conv1.register_forward_hook(hook_func)  # hook 등록
  ...
  handle.remove()  # hook 제거
  ```
* 예제 - 레이어의 activation을 빼오고싶다.
  ```python
  act_list = []
  def hook_get_activation(self, input, output):
      global act_list
      act_list.append(output)
      ...

  # activation을 훔칠거니깐 forward_hook으로 등록
  module.register_forward_hook(hook_get_activation)
  ...
  ```
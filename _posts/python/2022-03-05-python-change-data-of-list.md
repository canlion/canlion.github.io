---
title: "s[:] = s[::-1]"
# folder: "blog"
excerpt: "파이썬 리스트 조작 - s[:]"

categories:
  - python
tags:
  - [python]

toc: true
toc_sticky: true

date: 2022-03-05T23:27:00+09:00
last_modified_at: 2022-03-05T23:27:00+09:00
---
두 변수 `a`와 `b`가 리스트 객체를 참조하고 있는 상황에서 `b[:] = a`는 `b`가 참조하는 리스트에 `a`가 참조하는 리스트의 데이터를 집어넣는다.

```python
>>> a = [random.randint(0, 10) for _ in range(5)]
>>> a
[5, 4, 8, 4, 0]
>>> id(a)
140477818657152
>>> b = a  # b는 a가 참조하는 리스트를 참조한다.
>>> id(a) == id(b)
True
```

```python
>>> b = []
>>> b[:] = a  # b 리스트에 a 리스트의 데이터를 넣는다.
>>> id(a) == id(b)
False
>>> b[-1] = 10
>>> b
[5, 4, 8, 4, 10]
>>> a
[5, 4, 8, 4, 0]
```

이때 데이터를 복사하지는 않고 같은 객체를 가리킨다.

```python
>>> a = [0.134679]
>>> b = []
>>> b[:] = a
>>> id(b[0]) == id(a[0])
True
```

출처: 파이썬 알고리즘 인터뷰
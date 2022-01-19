---
title: "파이썬 - 3.1. 자료구조 / 3.2. pythonic code"
excerpt: "자료구조 & map, reduce, comprehension, ..."

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-19 13:04
last_modified_at: 2021-01-19 13:04
---
# 자료구조
데이터를 효율적으로 관리하고 다루기 위한 데이터의 구조화 방식

* 종류
  * 스택, 큐
  * 튜플, 집합
  * 딕셔너리
  * collection 모듈

## 스택 stack
* LIFO: Last In First Out / ex) 바구니. 위에서부터 꺼낸다.
* push & pop
* 리스트를 스택으로 활용 가능: append & pop
  ```python
  stack = []
  stack.append(1)
  stack.append(6)  # stack: [1, 6]
  elem = stack.pop()  # elem: 6 / stack: [1,]
  ```

## 큐 queue
* FIFO: First In First Out / ex) 대기열. 들어온 순서대로 처리.
* enqueue & dequeue
* 스택과 반대 개념
  ```python
  queue = []
  queue.append(1)
  queue.append(6)  # queue: [1, 6]
  elem = queue.pop(0)  # elem: 1 / queue: [6,]
  ```

## 튜플 tuple
* 값 변경 불가한 리스트 / immutable - 보호되어야하는 데이터의 시퀀스

## 집합 set
* 값을 순서없이, 중복없이 저장하는 시퀀스 - 중복되는 원소는 삭제, 하나만 유지.
* 집합연산 - (합\|차\|교)집합
  * (union, \|)
  * (intersection, &)
  * (difference, -)

## 딕셔너리 dict
* 데이터와 데이터에 접근하기위한 identifier(key)를 함께 보관
  * key는 immutable한, 구분되는 값이어야함.
    * ex) key: 주민등록번호, value: 이름
* Hash Table

## collections 모듈
* 리스트, 튜플, 딕셔너리에 대한 확장 자료 구조 모듈
* 주요 모듈
  * deque
    * 큐 + 스택
    * 기존 리스트 기능 + 리스트보다 빠른 저장 방식 (효율적인 메모리 구조)
    * append & appendleft, extend & extendleft(원소 순서도 역순으로 붙음)
    * rotate, reverse 등 linked list 특성 지원
  * Counter
    * 시퀀스 타입의 원소들의 수를 세서 딕셔너리로 반환
      ```python
      l = [1, 2, 3, 1, 1]

      Counter(l)  # Counter({1: 3, 2: 1, 3: 1})
      ```
    * 카운터끼리의 집합 연산이 가능.
  * OrderedDict
    * 파이썬 3.6부터는 딕셔너리도 순서를 보장. 이제는 의미 없어짐.
  * defaultdict
    * 딕셔너리는 key를 최초 한번 등록 해야하는데 (`d[key]=data`) defaultdict는 생성시에 기본 객체를 지정하여 새로운 key에 지정한 객체를 value로 부여할 수 있다.
      ```python
      d = dict()
      print(d['ho'])  # error

      dd = defaultdict(lambda: 0)
      print(dd['ho'])  # 출력: 0
      ```
  * namedtuple
    * 튜플 형태로 데이터 구조체 정의
    * 원소에 이름을 붙일 수 있다. $\rightarrow$ key: value 형태
      ```python
      Point = namedtuple('Point', ['x', 'y'])
      p = Point(x=1, y=10)
      print(p.x, p.y)  # 출력: 1 10
      ```

<br><br><br>

# Pythonic Code
* 파이썬 스타일의 코딩 기법
* 파이썬 특유의 문법을 활용해 효율적인 코드 표현
* 고급 코드 작성의 기초

## 예시

### split & join
  * 문자열 조작

### list comprehension
  ```python
  l = [(i, j) for i in list_0 for j in list_1]
  # 코드를 매우 간소화
  l = []
  for i in list_0:
    for j in list_1:
      l.append((i, j))
  ```

### enumerate & zip

### lambda & map & reduce

* lambda: 익명 함수, 3부터 권장하지 않으나 여전히 많이 쓰임. (def 구문이 권장됨)
  * 이유
    * 어려운 문법
    * 테스트의 어려움
    * docstring 없음
    * 해석 어려움
    * 이름이 없다
* map: 함수를 시퀀스의 각 원소에 적용함. 함수의 파라미터가 여러개면 파라미터 수만큼의 시퀀스가 필요. 이제는 list comprehension이 권장됨.
  ```python
  map(lambda x, y: x+y, list_0, list_1)
  ```
* reduce: `functools.reduce` / 시퀀스의 원소에 함수를 적용하며 통합해나감. 권장하지 않음.
  ```python
  list_0 = [1, 2, 3, 4, 5]
  reduce(lambda x, y: x+y, list_0)  # 1+2+3+4+5 = 15
  ```

### asterisk *: 가변인자의 사용

* 가변인자: 갯수가 정해지지 않은 인자
  * 함수 정의 파라미터의 *, `def func(*args):`: 가변인자 수용
  * 시퀀스 앞의 *, `print(*[1, 2, 3])`: unpacking
* unpacking a container: 리스트, 딕셔너리를 언팩킹하여 포지션인자, 키워드인자 전달
  ```python
  # 가변인자
  def asterisk_print(a, b, *args, **kwargs):
      return print(a, b, args, kwargs)

  asterisk_print(1, 2, 3, 4, 5, haha='haha', hoho='hoho')
  # 출력: 1 2 (3, 4, 5) {'haha': 'haha', 'hoho': 'hoho'}

  # unpacking container: 시퀀스, 딕셔너리를 언팩킹하여 인자를 넘겨줄 수 있다.
  def asterisk_print_(a, b, c):
      print(a, b, c)
  
  asterisk_print_(*[1, 2, 3])  # 1 2 3
  asterisk_print_(**{'a': 1, 'b': 2, 'c': 3})  # 1 2 3
  ```

### iterable object: 시퀀스형 자료형에서 데이터를 순서대로 추출하는 오브젝트

* `__iter__`, `__next__` 메소드 사용
* `iter()`, `next()` 함수로 iterable 객체를 iterator 객체로 사용
  ```python
  import sys

  list_0 = ['haha', 'hoho', 'hehe']
  iter_list_0 = iter(list_0)  # 시퀀스 자료형의 데이터 주소를 보관

  print(sys.getsizeof(list_0))  # 80
  print(sys.getsizeof(iter_list_0))  # 48
  
  # next 함수를 통해 값을 요청할때 데이터 주소를 통해 값을 반환한다.
  print(id(list_0[0]))  # 139782265343728 
  print(id(next(iter_list_0)))  # 139782265343728 - 같은 객체를 참조함
  ```
* generator: iterable 객체를 특수한 형태로 사용하는 함수. 원소가 사용되는 시점에 값을 메모리에 반환(yield를 통해 한번에 하나의 원소 반환). **대용량 데이터를 다룰때 사용한다.**
  ```python
  # list
  print(sys.getsizeof([i for i in range(50)]))  # 520

  def generator_int(x):
      for i in range(x):
          yield i
  
  # generator - 순회 가능하면서도 값을 호출하는 시점에서 메모리에 반환.
  gen_0 = (i for i in range(50))  # generator comprehension 활용
  gen_1 = generator_int(50)  # yield 활용
  print(sys.getsizeof(gen_0))  # 112
  print(sys.getsizeof(gen_1))  # 112
  ```
  * 사용
    * 리스트, 큰 데이터, 파일 처리에 사용
      

**iterable object & generator 어떤 방식인지 감이 안온다.**
{: .notice--danger} 
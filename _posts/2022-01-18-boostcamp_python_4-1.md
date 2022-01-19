---
title: "파이썬 - 4.1. OOP"
excerpt: "파이썬 객체지향프로그래밍"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-19 15:28
last_modified_at: 2021-01-19 15:28
---
# 객체지향 프로그래밍

어떤 기능을 담당하는 주체들을 선정하고 주체들마다의 행동과 데이터 구조를 프로그래밍하는 방식

### 개요
* Object-Oriented Programming, OOP
* 객체: 속성attribute와 행동action을 가진 것
  * 객체: 축구선수
    * 속성: 이름, 소속팀, 포지션
    * 행동: 패스, 슛, 태클
* OOP는 객체 개념을 프로그램으로 표현하여 속성은 변수, 행동은 메소드로 표현
  * 설계도에 해당하는 class -> 설계도대로 만들어낸 구현체 instance
  
```python
# 설계도
class SoccerPlayer:  # CamelCase
    # 객체 초기화 예약함수
    def __init__(self, name: str, position: str, number: int):
        self.name = name
        self.position = position
        self.number = number
        
    def run(self):
        print('뛴다!')
        
    # __메소드명__: 특수한 예약 함수 또는 변수, 함수명 변경(맨글링)으로 사용
    def __str__(self):  # __str__: print를 통해 인스턴스 출력시에 출력할 문자열을 반환
        return f'{self.name} / {self.position} / {self.number}'

# 설계도대로 만든 인스턴스
murloc_0 = SoccerPlayer('Murloc', 'GK', 0)
murloc_1 = SoccerPlayer('Murloc', 'DF', 1)
```

### 특징
* 상속 Inheritance
* 다형성 Polymorphism
* 가시성 visibility

#### 상속
* 부모클래스로의 속성과 메소드를 이어받은 자식 클래스를 생성

```python
class Pen:
    def __init__(self, make):
        self.make = make
        
    def about_make(self):
        print(f'make: {self.make}')
        
class FX153(Pen):
    def __init__(self, make, point_size, color):
        super().__init__(make)  # super(): 부모 클래스를 지칭
        self.point_size = point_size
        self.color = color
    
    def pen_info(self):
        super().about_make()
        print(f'point_size: {self.point_size} mm, color: {self.color}')

fx_153 = FX153('monami', '0.7', 'black')
fx_153.pen_info()
```

#### 다형성
* 같은 이름의 메소드의 내부 로직을 다르게 작성(개념적으로 같은 행동을 하되 다른 로직)

```python
class Boxer:
    def __init__(self, name):
        self.name = name
        
    def hook(self):
        raise NotImplementedError('자식클래스는 반드시 hook 메소드를 구현해야함.')
        # 자식 클래스에서 hook 메소드를 구현하지않으면 부모 클래스의 hook이 실행되고 에러 레이즈
        
class Southpaw(Boxer):
    """왼손잡이 복서"""
    def __init__(self, name):
        super().__init__(name)
        
    def hook(self):
        print('left hook-!')

        
class Orthodox(Boxer):
    """오른손잡이 복서"""
    def __init__(self, name):
        super().__init__(name)
        
    def hook(self):
        print('right hook-!')

        
s_boxer = Southpaw('A')
o_boxer = Orthodox('B')

s_boxer.hook()
o_boxer.hook()
```

#### 가시성
* 객체의 일부 변수를 객체 외부에서 직접 접근할 수 없도록 제한함
  * 사용자에 의한 객체 변수 임의 조작
  * 불필요한 접근
  * 소스의 보호 (?) 잘 되나?
* 캡슐화 Encapsulation, 정보 은닉 Information Hiding
  * 클래스 설계할때 클래스 간 간섭/정보공유 최소화: 얽히고 설킬수록 수정, 관리가 힘들어진다.
    * 클래스간에도 불필요한 정보 접근은 최소화하자 
    * 인터페이스만으로 객체를 사용할 수 있도록 하자

* 객체의 변수 접근 제한

```python
class AngryPerson:
    def __init__(self, name, reason_angry):
        self.name = name
        self.reason_angry = reason_angry
        
        
class Person:
    def __init__(self, name):
        self.name = name
        

class BambooGrove:
    def __init__(self):
        self.__angry_people = []  # __, 변수앞의 언더바 2개: private 변수로 선언, 타객체가 접근 불가
        
    def entering_grove(self, angry_person):
        if type(angry_person) == AngryPerson:
            self.__angry_people.append(angry_person)
            print('풀고 가십쇼')
        else:
            print('화가 없는 사람은 돌아가십쇼.')

person_0 = AngryPerson('A', '내일이 월요일')
person_1 = Person('B')

bg = BambooGrove()
bg.entering_grove(person_0)
bg.entering_grove(person_1)

print(bg.__angry_person)  # AttributeError
```

* 객체의 변수 접근은 가능하되 변수값을 바꿔치지는 못하게한다.

```python
class Person:
    def __init__(self, belongings):
        self.__belongings = belongings
        
    @property  # 메소드를 속성처럼 쓰게한다.
    def belongings(self):
        return self.__belongings
    

person = Person('phone')
print(person.belongings)  # 저 사람이 뭘 들고있는지 볼 수는 있다.

person.belongings = ''  # AttributeError: can't set attribute / 강제로 물건을 뺏거나 바꿔치지는 못한다.
```

### 데코레이터 decorator

#### 먼저
* first-class objects
* inner function
* decorator

#### first-class objects
* 일등 함수, 일급 객체
* 변수나 데이터 구조에 할당이 가능한 객체
* 파라메터로 전달이 가능, 리턴 값으로 반환 가능
* 파이썬 함수는 일등 함수

```python
def brushing_teeth():
    print('양치질...')
    

def sleep(before_sleep):
    before_sleep()
    print('취침...')


sleep(brushing_teeth)
```

#### inner function
🔹 함수 안의 함수

```python
def sleep(action):
    def brushing_teeth():
        print('양치질...')
        print(action)
    
    brushing_teeth()
    print('취침...')


sleep('알람맞추기')
```

🔹 closures: inner function 자체를 반환할 수 있고 반환된 inner function을 closure라 한다.
받은 인자나 내부 변수를 유지하여 반환된 클로저를 호출할때 사용된다.

```python
def sleep(action):
    def brushing_teeth():
        print('양치질...')
        print(action)
        print(on_the_bed)
        print('취침...')
    on_the_bed = 'youtube 보기'
    return brushing_teeth


sleep_process = sleep('물마시기')
sleep_process()
```

#### 데코레이터는 closure를 간단하게 생성

```python
def sleep_deco(before_sleep):
    def inner(*args, **kwargs):
        print('이닦기..')
        print(f'{args[0]} 유튜브 올라왔나..')
        before_sleep(*args, **kwargs)
        print('취침...')
    return inner
    
@sleep_deco  # 함수를 인자로 받아 closure를 반환한다
def watch_youtube_on_bed(title):
    print(f'유튜브시청 - {title}')
    

watch_youtube_on_bed('옥냥이')
```

🔹 2중  inner function을 통한 인자값을 받는 데코레이터 정의

```python
def dessert(d_name):  # 데코레이터를 반환한다
    def wrapper(f):  # 반환되는 데코레이터. 변수 d_name을 가지고 간다.
        def inner(*args, **kwargs):
            f(*args, **kwargs)
            print(f'올 때 {d_name}')
        return inner
    return wrapper


@dessert('메로나') # closure를 반환하는 closure를 반환한다
def see_you_later():
    print('잘가')

    
see_you_later()
```
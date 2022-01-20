---
title: "파이썬 - 4.2 모듈 & 프로젝트 / 4.3 파일 & 예외 & 로그"
excerpt: "모듈, 패키지, 프로젝트의 개념 / 파일다루기, 예외처리, 로그남기기"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-20 15:31
last_modified_at: 2021-01-20 15:31
---
# 모듈과 패키지

## 모듈
부분, 조각. 프로그램을 구성하는 작은 프로그램 조각들. 모아서 하나의 큰 프로그램 개발. 잘 모듈화시킬수록 다시 사용하기 좋음.

*.py 파일 의미. 모듈 내부에 작성된 함수, 객체 등을 가져와 사용한다. `import`

**\_\_pycache\_\_ \*.pyc**: 컴파일된 파일. 기계어로 번역된 코드. 다음에 사용할때 더 빠르게 로딩한다.

## 패키지 (프로젝트)
하나의 대형 프로젝트를 만드는 코드의 묶음. 다양한 모듈들의 모음. 

* \_\_init\_\_.py:
  * 현재 디렉토리가 패키지임을 알리는 초기화 스크립트. 이제는 없어도 되지만..
  * 현재 디렉토리에서 사용할 모듈, 패키지를 명시한다.
  * 하위 디렉토리와 모듈을 포함한다.
    ```python
    __all__ = ["hoho", "haha", "hehe"]  # 패키지명(디렉토리) 또는 .py 모듈 파일명

    from . import hoho
    from . import haha
    from . import hehe
    ```
* \_\_main\_\_.py
  * ```python
    if __name__ == '__main__':
        ...
    ```
  * 패키지 디렉토리를 실행할때의 동작 정의.
    * `python {패키지디렉토리명}`


### 패키지 내에서 다른 폴더의 모듈을 부를때
* 절대참조: `from a.b.c import d`
* 상대참조:
  * 현재 디렉토리 기준: `from .b import d`
  * 부모 디렉토리 기준: `from ..b.c import d`

### 가상환경
프로젝트별로 파이썬 환경을 만들어줄 수 있음. 가상환경 내에서 패키지를 설치하면 가상환경 외부에는 영향을 주지 않음. (ex - 동일한 패키지를 가상환경별로 다른 버전으로 유지할 수 있음.)

* 대표적인 가상환경 도구
  * virtualenv + pip
    * 대표적인 가상환경 관리 도구
    * 장점: 레퍼런스 + 패키지 개수
  * conda
    * 상용 가상환경 도구
    * 장점: 설치의 용이성


# Exception / File / Log Handling

## Exception

* 예외
  * 예상 가능한 예외 - 개발자가 명시적으로 예외 처리
    * ex) 주로 사용자의 조작과 관련된 예외들
  * 예상 불가능한 예외 - 인터프리터가 예외 호출
    * ex) 인터프리터 과정에서 발생 - 인덱스 에러, 0으로 나누기, 개발자의 실수, ...

### Exception Handling, 예외 처리

예외가 발생하면 처리할 방법이 필요하다.

**try ~ except 문법**
```python
try:
    예외가 발생할 수 있는 코드
except ValueError as e:  # 구체적인 예외를 지정
    발생한 예외에 대응하는 코드
except Exception as e:  # 모든 예외를 캐치
    대응 코드
else:  # 생략 가능
    예외가 발생되지않은 경우 실행되는 코드
finally:  # 생략 가능
    예외 발생과 무관하게 항상 실행되는 코드
```

예외 발생하면 해당 예외의 종류를 명시한 `except` 구문이 발생한 예외를 캐치하여 대응하는 코드가 실행되고 프로그램은 종료되지않고 계속 진행된다. 구체적인 예외를 지정하여 구체적인 대응 방안을 마련하자.

**raise 구문**
```python
if a = 0:
    raise ZeroDivisionError
```

강제로 `Exception`을 발생시킨다. 불완전하게 프로그램 실행을 유지하는 것보다 프로그램을 종료시키는게 차라리 나은 상황인 경우. (무의미하게 많은 리소스를 차지하는 경우 등등)

**assert 구문**
```python
def func(a, b):
    assert a != 0, 'a는 0이어서는 안된다.'
    return b / a
```

`assert`는 조건이 `False`인 경우 `AssertionError`를 발생시킨다.

## File Handling

### 파일 종류
* text 파일
  * 인간이 이해하는 문자열 형식으로 저장된 파일
  * 메모장으로 열면 사람이 이해할 수 있다. - *.txt, *.py, ...
* binary 파일 
  * 컴퓨터만 알 수 있는 이진(법)형식으로 저장된 파일
  * 메모장으로 열면 사람이 읽을 수 없는 출력을 보인다.

### python I/O

#### read

```python
f = open('file_path', 'r')
txt = f.read()
f.close()

with open('file_path', 'r') as f:
    txt = f.read()
```

* read 메소드
  * file.read(): 파일 내용 통채로 반환.
  * file.readlines(): `\n`기준으로 잘라 리스트로 반환.
  * file.readline(): 실행때마다 한줄씩 읽어 메모리에 올린다.
    * 파일 용량이 너무 큰 경우 사용.
    * 파일 끝에 도달하면 빈 문자열 반환

#### write

```python
f = open('file_path', 'w', encoding='utf8')
f.write(data)
f.close()

with open('file_path', 'w', encoding='utf8') as f:
    f.write(data)
```

encoding: 데이터를 컴퓨터가 이용하는 코드로 변환하는 방식. (utf-8, euc-kr, cp949, ...)

* mode
  * w: 새로 파일을 생성하거나 파일이 존재하면 덮어쓰기.
  * a: 기존 파일 뒤에 데이터를 쓴다.

### pathlib

경로를 객체로 다룬다. 다양한 기능 제공.

### pickle

파이썬 객체를 영속화하는 빌트인 객체. 데이터나 객체 등을 상태 그대로 저장하고 불러올 수 있다.
```python
import pickle

f = open('ho.pickle', 'wb')
l_0 = (0, 5, 10)
pickle.dump(l_0, f)
f.close()

f = open('ho.pickle', 'rb')
l_1 = pickle.load(f)
f.close()
```

## Logging

프로그램 실행되는 동안의 동작을 기록을 남김. (유저의 접근, exception, 함수의 사용 등등). 로그를 분석해서 유의미한 결과를 얻고 프로그램을 유지보수하는 것이 목적. 

### logging 모듈

#### 로그 종류
* debug
  * 개발시 처리 기록
  * ex) A를 호출함, 변수 B를 변경함, ...
* info
  * 처리가 진행되는 동안의 정보
  * ex) 서버 시작 / 서버 종료 / 사용자 A 접속 / ...
* warning - 예외 발생. 조심.
  * 개발시에 의도치않은 동작 발생에 대한 정보
  * 동작 + 예외처리
    * ex) numpy array 대신 리스트 입력됨 -> array로 변환 / deprecated 알림 / ...
* error - 예외 발생
  * 에러 발생했으나 프로그램 동작 가능
  * ex) 파일 기록 불가 - 예외처리 후 사용자에게 알림 / 서버 접속 불가 / ... 
* critical - 프로그램이 완전 종료되는 수준의 문제
  * 잘못된 처리로 데이터 손실, 더 이상 프로그램 동작 불가
  * ex) 사용자의 강제 종료 / 잘못된 처리로 중요 파일 삭제 / ...

`debug`는 개발측면에서, `info`는 운영측면에서 유의미한 내용을 담기위해 사용하고 `warning, error, critical`은 사용자에게 어떤 경고를 보내기위한 메세지를 담을 수도 있다.

#### logging 모듈

기본 로깅 레벨은 `warning` (`warning` ~ `critical`)

* logging 옵션 설정 방식
  * configparser - 파일에 설정값을 담아 관리.
    * 설정을 file에 저장 - key:value 쌍으로 설정-값 저장.
  * argparser - 콘솔에서 프로그램 실행시에 인자값으로 넘겨준다.
    * 커맨드 라인 옵션 - 콘솔에서 프로그램 실행시에 인자를 넘겨준다.

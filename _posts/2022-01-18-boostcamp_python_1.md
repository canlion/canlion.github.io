---
title: "파이썬 - 2-2 ~ 2-4"
excerpt: "함수, 조건문, 문자열"

categories:
  - boostcamp
tags:
  - [boostcamp]

toc: true
toc_sticky: true

date: 2022-01-18 11:36
last_modified_at: 2021-01-18 11:36
---
## 함수
* 캡슐화: 내부 로직을 몰라도 인터페이스만으로 타인의 코드 사용 가능
* parameter: 함수의 입력 인터페이스. 인자값을 받는 변수.
* argument: 인자값, 파라미터에 대입되는 값.

## 문자열 formatting
* % <br>
  * |type|설명|
  |----|---|
  |%s|문자열|
  |%c|문자한개|
  |%d|정수|
  |%f|부동소수점|
  |%o|8진수|
  |%x|16진수|
  |%%|%자체|
* format이나 f-string이 보기 편하다.
* 참고: <a href="https://pyformat.info/" class="btn btn--info">link: pyformat</a>

## 조건문
### 비교연산자
* is, is not: 객체의 메모리 주소를 비교

## 메모
* 자주 사용되는 -5 ~ 256 정수를 정적 메모리에 보관


## loop
* for ... else, while ... else:
  * for문, while문이 **break의 방해 없이** 정상적으로 loop를 탈출하면 else에 포함된 코드가 실행된다.


## 문자열
* 영문자 한글자: 1 byte
  * 1 byte = 8 bit (0~255)
  * [2진수 $\leftrightarrows$ 문자] 규칙이 존재. ex) UTF-8
* 시퀀스형 데이터
* raw string: `r"raw string\n"` - \n도 문자로 인식

## 함수

* 파라미터 전달 방식
  * call by value: 함수에 인자를 넘길때 값만 넘김. 함수내의 활동이 함수 밖의 인자에 영향을 미치지 않음.
  * call by reference: 함수에 인자를 넘길때 메모리 주소를 넘김. 함수 내에서 메모리 주소를 통해 메모리에 영향을 줄 수 있음.
  * call by object reference: 파이썬은 함수에 인자를 넘길때 객체의 주소를 넘김. list 같은 mutable한 객체를 넘기면 함수 내의 작업이 객체에 영향을 미칠 수 있음.
    ```python
    list_obj = []
    
    
    def append_elem(l, elem):
        l.append(elem)  # 변수 l과 list_obj는 동일한 리스트를 참조
    
    append_elem(list_obj, 3)
    print(list_obj)  # [3]
    ```

* 가이드라인
  * 가능하면 짧게 여러개. 기본적인 기능단위로 작성.
  * 함수명에 의도, 역할 표현
  * 함수 기능을 일관성있게 작성. (함수에 뜬금없는 기능 넣지 않기)
  * 남이 이해할 수 있게 짜야한다.
    * **코딩 컨벤션**
      * flake8, black의 도움을 받자.
        * black: pep8에 근접한 수준으로 코드를 **수정**

---

잠깐 복습
<table>
  <thead>
    <tr>
      <th>종류</th><th>타입</th><th>크기</th><th>표현 범위 (32bit)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan=2>정수형</th><th>int</th><th>4byte</th><th>-2^31 ~ (2^31)-1</th>
    </tr>
    <tr>
      <th>long</th><th>무제한</th><th>무제한</th>
    </tr>
    <tr>
      <th>실수형</th><th>float</th><th>8byte</th><th>약 10^(-308) ~ 10^308</th>
    </tr>
  </tbody>
</table>

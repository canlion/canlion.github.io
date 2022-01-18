---
title: "파이썬 - 기초"
excerpt: "파이썬 개요, 환경 설정, Variables"

categories:
  - boostcamp
tags:
  - []

toc: true
toc_sticky: true

date: 2022-01-17 13:17
last_modified_at: 2021-01-17 13:17
---
# 파이썬
## 파이썬 개요
  * 플랫폼 독립적 인터프리터 언어
    * 운영체제에 독립적 - 각 운영체제에서 작동하는 인터프리터만 설치되면 코드 동작
  * 객체지향 - 단위 모듈 중심으로 코드 작성
  * 동적 타이핑 - 프로그램 실행 시점에서 데이터 타입 결정

## Variable & List

### 변수
* 데이터를 저장하는 메모리 공간에 붙인 이름.
* 변수 선언하면 메모리에 물리적인 공간을 점유하고 그 주소를 보유, 주소를 통해 데이터를 메모리에 할당.
  * 컴퓨터 구조 - 폰 노이만 아키텍처: 데이터를 메모리에 저장, CPU가 순차적으로 데이터를 해석하고 계산.

### 간단한 연산

* +, -, /, *, %, ...
* type, int, float, str, ...

#### 데이터 타입
<table>
  <thead>
    <tr>
      <th colspan=3>유형</th>
      <th>설명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan=2>수치자료형</td>
      <td>정수형</td>
      <td>integer</td>
      <td>정수</td>
    </tr>
    <tr>
      <td>실수형</td>
      <td>float</td>
      <td>소수점 포함된 실수</td>
    </tr>
    <tr>
      <td colspan=2>문자형</td>
      <td>string</td>
      <td>문자 데이터</td>
    </tr>
    <tr>
      <td colspan=2>논리형</td>
      <td>boolean</td>
      <td>참/거짓</td>
    </tr>
  </tbody>
</table>

* 데이터 타입마다 메모리 점유 크기 다름

### list
* 시퀀스 자료형 - 데이터를 원소로 삼는 집합, 원소 데이터 타입은 무관
* mutable 객체이므로 객체 복사에 주의. (copy.deepcopy 이용)
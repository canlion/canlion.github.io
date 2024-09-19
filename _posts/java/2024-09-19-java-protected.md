---
title: "접근 제어자 - protected"
# folder: "blog"
excerpt: ""

categories:
  - java
tags:
  - [java]

toc: true
toc_sticky: true

date: 2024-09-19T00:00:00+09:00
#last_modified_at: 2022-03-05T23:27:00+09:00
---

# 접근 제어자 - protected

 "같은 패키지의 클래스에서 또는 자식 클래스에서의 접근"만 허용하는 접근 제어자.

자식 클래스가 부모 클래스로 캐스팅되면 protected는 어떻게 적용되는지 등이 궁금해서 몇 가지 시도를 해보니 아마도 다음과 같은 규칙이라 생각된다.
1. protected 메서드가 정의/오버라이딩된 클래스의 패키지에서 접근할 수 있다.
2. {자식 클래스}에서 {자식 클래스 객체}의 protected 메서드에 접근할 수 있다.
3. 위의 규칙들은 객체의 실제 타입이 아닌 캐스팅된 타입을 기준으로 적용된다.


# 예시

```java
package org.package1;

public class A {
    public void publicMethod() {System.out.println("A-public");}
    protected void protectedMethod() {System.out.println("A-protected");}
    void defaultMethod() {System.out.println("A-default");}
    private void privateMethod() {System.out.println("A-private");}
}

```

## protected 메서드가 정의/오버라이딩된 클래스의 패키지에서 접근할 수 있다.

```java
package org.package2;

import org.package1.A;

public class BExtA extends A {
}

```
```java
package org.package1;

import org.package2.BExtA;

public class C {
    public void method() {
        BExtA BExtA = new BExtA();
        BExtA.protectedMethod();
    }
}

```
```java
package org.package2;

public class D {
    public void method() {
        BExtA BExtA = new BExtA();
        BExtA.protectedMethod();
    }
}

```

### 다른 패키지의 자식 클래스에서 오버라이딩하지 않은 경우 - 부모 클래스의 패키지에서 접근할 수 있다.

```
/src/main/java/org/package2/D.java:6: error: protectedMethod() has protected access in A
        BExtA.protectedMethod();
             ^
```

`package1.A`를 상속한 `package2.BExtA`의 `protectedMethod`는 `package1.A`에서 정의되었으므로 `package1`에서 접근 가능하다.

따라서 `package2.D`에서의 접근에서 에러 발생

### 다른 패키지의 자식 클래스에서 오버라이딩한 경우 - 자식 클래스의 패키지에서 접근할 수 있다.

```java
package org.package2;

import org.package1.A;

public class BExtA extends A {
    @Override
    protected void protectedMethod() {
        super.protectedMethod();
    }
}

```
```
/src/main/java/org/package1/C.java:8: error: protectedMethod() has protected access in BExtA
        BExtA.protectedMethod();
             ^
```

`package2.BExtA`에서 `protectedMethod`를 오버라이딩하면 `package2`에서 접근 가능하다.

따라서 `package1.C`에서의 접근에서 에러 발생

## {자식 클래스}에서 {자식 클래스 객체}의 protected 메서드에 접근할 수 있다.

### {자식 클래스}에서 {부모 클래스 객체}의 protected 메서드에 접근할 수는 없다.

```java
package org.package2;

import org.package1.A;

public class BExtA extends A {
    public static A a = new A();

    public void method() {
        a.protectedMethod();
    }
}

```
```
/src/main/java/org/package2/BExtA.java:9: error: protectedMethod() has protected access in A
        a.protectedMethod();
         ^
```

"protected 메서드는 자식 클래스에서 접근할 수 있다."라는 표현에서 위와 같은 형식도 가능한가 싶었으나 "자식 클래스가 물려받은 것을 사용할 수 있다." 정도의 의미인 듯 하다.


## 위의 규칙들은 객체의 실제 타입이 아닌 캐스팅된 타입을 기준으로 적용된다.

```java
package org.package2;

import org.package1.A;

public class BExtA extends A {
    @Override
    protected void protectedMethod() {  // package2 또는 자식 클래스에서 접근 가능 
    }
}

```
```java
package org.package1;

import org.package2.BExtA;

public class CExtB extends BExtA {
    public void method() {
        CExtB c = new CExtB();
        c.protectedMethod();

        BExtA b = (BExtA) c;
        b.protectedMethod();  // ERROR

        A a = (A) c;
        a.protectedMethod();
    }
}

```

```
/src/main/java/org/package1/CExtB.java:11: error: protectedMethod() has protected access in BExtA
        b.protectedMethod();
         ^
```

* 상속 관계: `pacakge1.A` <- `package2.BExtA` <- `package1.CExtB`
* 3번의 `protectedMethod` 호출
  * `c.protectedMethod()`: `package1.CExtB`가 `package2.BExtA`를 상속하였으니 접근 가능
  * `b.protectedMethod()`: `package1.CExtB` 객체를 `package2.BExtA`로 캐스팅하여 `package2.BExtA` 기준으로 접근제어자가 적용되어 메서드 접근 불가
    * 메서드를 오버라이딩했으므로 `package1`에서 접근 불가
    * {자식 클래스:`CExtB`}에서 {부모 클래스 객체:`BExtA`}의 protected 메서드에 접근 불가
  * `a.protectedMethod()`: `package1.CExtB` 객체를 `package1.A`로 캐스팅하여 `package1.A` 기준으로 접근제어자가 적용된다. `package1.A`의 `protectedMethod`는 `package1`에서 접근 가능하므로 `package1.CExtB`에서 접근 가능하다.

다형성 측면에서는 당연한 듯 하다. 캐스팅된 타입에 맞게 동작해야지.

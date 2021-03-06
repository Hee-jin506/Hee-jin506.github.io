---
layout: post
title: "[자바 문법] 변수"
categories:
  - Eomcs
tags:
  - 자바 문법

---

> - 2020-07-23, 27 일자 수업 내용
> - [eomcs/eomcs-java](https://github.com/eomcs/eomcs-java) src/main/com/eomcs/lang/ex04
> - 자바의 정석 Chap2

#  변수

변수(variable)은 값을 저장하는 메모리를 준비하는 명령어이다. 변수를 선언하면, 값을 저장할 메모리의 위치와 크기를 결정하고, 그 메모리에 이름을 부여한다. 그 이름으로 변수를 다시 호출하면, 해당 이름에 맞는 메모리에 접근하여 값을 초기화하거나, 수정하거나, 꺼낸다. 변수를 선언하는 것을 "변수를 생성"한다고 표현하기도 한다.

변수를 선언하는 문법은 변수의 데이터 타입(자료형), 즉, 메모리의 종류를 적고 그 뒤에 변수명을 지정하는 것이다.

```java
int i;
```

> **변수명 짓기**
>
> - 대소문자를 구분하며, 보통 소문자로 시작한다.
> - 여러 단어로 구성된 경우, 두번째 단어의 시작 알파벳은 대문자로 한다.
> - 상수인 경우 보통 대문자로 이름을 짓는다. 단어와 단어의 사이는 `_`를 집어넣는다.

### 여러 개의 변수 한번에 선언하기

다음과 같이 한 문장으로 같은 유형의 변수를 여러 개 선언할 수 있다.

```java
int j1, j2, j3;
```

### 변수에 값 할당하기

변수가 가리키는 메모리에 값을 저장하는 문법은 다음과 같다.
`변수명 = 변수 또는 리터럴`

```java
int age;
age = 20; 
```

여기서 `=`는 할당 연산자(assignment operator)로서, 오른쪽에 지정된 값을 왼쪽 변수에 가리키는 메모리에 저장한다.

- l-value: 왼쪽에 있는 변수를 가리키며, 리터럴이 올 수 없다.
- r-value: 오른쪽에 있는 변수나 리터럴을 가리킨다.

### 변수 선언과 값 할당을 동시에 하기

변수를 선언함과 동시에 값을 즉시 할당할 수가 있다. 문법은 다음과 같다.
`데이터타입 변수명 = 값;`

```java
int age = 20;
```

여러 개의 변수를 선언할 때에도 한 개를 선언할 때와 마찬가지로 값을 저장할 수 있다. 모든 변수를 다 초기화할 필요 없이 선언한 변수들 중 일부만 초기화할 수도 있다.

```java
int a1 = 100, a2 = 200;
int a4, a5 = 200, a6, a7 = 400, a8;
```

이미 초기화가 이뤄진 변수에 대해 새로운 값을 다시 할당할 수 있다.

```java
int age;
age = 20; 
age = 30;
```

### 변수 선언 오류

변수 선언보다 변수 사용이 위에 올 수 없다.

```java
count = 50; // 컴파일 오류
int count;
```

같은 블록 안에서 같은 이름의 변수를 중복해서 선언할 수 없다.

### 변수 사용

변수를 한번 생성하여 초기화까지 하고 나면, 다른 메서드에 값을 전달하거나, 혹은 또 다른 변수에 값을 복사하여 저장할 수가 있다.

```java
int age = 20;
System.out.println(age);
int age2 = age;
```

위의 예시에서 age와 age2는 같은 메모리가 아니다. age에 있는 값을 복사해서 별도로 생성된 age2 메모리에 저장하기 때문이다.

### 변수 사용 오류

값이 저장되지 않은 변수를 사용하면 문법 오류이다.

```java
int age;
System.out.println(age); // 컴파일 오류
```

___

## 변수의 종류

자바 원시 타입의 값을 저장하는 변수와 메모리 주소를 저장하는 변수가 있다. 자바 원시 타입 변수(primitiive variable)에는 정수, 부동소수점, 논리, 문자코드와 같은 값을 저장할 수 있고, 레퍼런스 변수(reference variable)는 객체의 메모리 주소를 참조한다.

### 자바 원시 데이터 타입 변수

- 정수

  - byte : 1byte 메모리 (-128 ~ 127)
  - short : 2byte 메모리(-32768 ~ 32767)
  - Int : 4byte 메모리 (약 -21억 ~ 21억)
  - long : 8byte 메모리(약 -922경 -922경)

  ```java
      byte b;  // 1바이트 크기의 메모리
      short s; // 2바이트 크기의 메모리 
      int i;   // 4바이트 크기의 메모리 
      long l;  // 8바이트 크기의 메모리
  ```

- 부동소수점

  - float : 4byte 메모리 (유효자릿수 7자리)
  - double : 8byte 메모리

  ```Java
      float f;   // 4바이트 크기의 메모리
      double d;  // 8바이트 크기의 메모리
  ```

- 문자

  - char : 2byte 메모리(0~65535) UCS-2 코드 값 저장

  ```java
      char c;  // 2바이트 크기의 메모리
  ```

- 논리

  - boolean : 기본적으로는 4byte int 메모리를 사용하지만 배열의 요소일 경우 1byte byte 메모리를 사용

  ```java
      boolean bool; 
  ```

- 레퍼런스 변수

  - 데이터가 저장된 메모리의 주소를 저장하는 메모리

  ```java
      String str;
      Date date;
  ```

___

## 정수 변수

### 변수의 메모리 크기

정수 값을 담을 수 있는 변수 타입의 종류는 네 가지로, 각각 1바이트, 2바이트, 4바이트, 8바이트 크기의 메모리를 갖는다.

| 변수        | byte     | short        | int                    | long                                     |
| ----------- | -------- | ------------ | ---------------------- | ---------------------------------------- |
| 메모리 크기 | 1바이트  | 2바이트      | 4바이트                | 8바이트                                  |
| 최대/최소   | -128/127 | -32768/32767 | -2147483648/2147483647 | -9223372036854775808/9223372036854775807 |

값을 메모리에 저장하려면 2진수 형태로 바꿔야 한다. 정수를 2진수로 바꿀 때에는 `2의 보수` 방식을 사용한다. 

다음은 메모리 크기 때문에 발생한 오류가 아니다. 리터럴 크기의 문제이다. 

```java
i = -2147483649; // 컴파일 오류!
i = 2147483648; // 컴파일 오류!
```

### 변수와 리터럴의 크기

- 4바이트 정수 리터럴 :  정수 변수에 저장할 떄에는 메모리 크기에 상관없이 저장만 가능하다면 컴파일 오류가 발생하지 않는다.
- 8바이트 정수 리터럴 : 8바이트 리터럴인 경우 값이 크고 작음에 상관없이 byte, short, int 메모리에 값을 저장하면 컴파일 오류가 발생한다.

### 크기가 다른 변수끼리 값 할당

변수의 값을 다른 변수에 저장할 때에는 값의 크기에 상관없이 같거나 큰 크기의 메모리여야 한다.

```java
    long l = 100;
    int i = 100;
    short s = 100;
    byte b = 100;
    char c = 100;

    long l2;
    int i2;
    short s2;
    byte b2;
    char c2;

    // long ===> long 이상
    l2 = l;
    //i2 = l; // 컴파일 오류
    //s2 = l; // 컴파일 오류
    //b2 = l; // 컴파일 오류!
    //c2 = l; // 컴파일 오류!

    // int ===> int 이상
    l2 = i;
    i2 = i;
    //s2 = i; // 컴파일 오류!
    //b2 = i; // 컴파일 오류!
    //c2 = i;  // 컴파일 오류!
    
    // short ===> short 이상
    l2 = s;
    i2 = s;
    s2 = s;
    //b2 = s; // 컴파일 오류!
    //c2 = s; // 컴파일 오류! char(0 ~ 65535) | int(-32768 ~ 32767)
    
    // byte ===> byte 이상
    l2 = b;
    i2 = b;
    s2 = b;
    b2 = b;
    //c2 = b; // 컴파일 오류! char(0 ~ 65535) | byte(-128 ~ 127)
```

___

## 부동소수점 변수

### 변수의 메모리 크기

부동소수점 변수에는 4바이트, 8바이트 크기의 메모리를 가진 타입 두 가지가 있다.

| 변수        | float   | double  |
| ----------- | ------- | ------- |
| 메모리 크기 | 4바이트 | 8바이트 |
| 유효자릿수  | 7자리   | 15자리  |

> **유효자릿수**
>
> 10진수로 부동소수점을 표현할 경우, 정상적으로 메모리에 저장될 수 있는 숫자의 소수점을 제외한 자릿수

유효자릿수를 확인할 때에는 소수점을 뗏을 때의 총 자릿수를 세되, 숫자 앞에 한개 이상의 0이 있다면 0을 자릿수에서 제외한다.

```java
    float f;    
    f = 0.000009876545f; // 소수점을 떼면 숫자가 13개 이지만, 앞의 0을 제외하면 실제 7개이다.
```

유효자릿수를 넘은 숫자를 저장하면 값이 정상적으로 저장되지 않을 수 있다.

```java
    f = 9.8765456f; // 맨 뒤의 값이 반올림 된다.
    System.out.println(f); // 9876546.0
```

> **단정도(single precision)와 배정도(double precision)**
>
> double 변수는 float 변수에 두 배 정도 더 정밀한 값을 저장할 수 있으므로 배정도라고 불리고, float은 단정도라 불린다.

### 변수와 리터럴의 크기

변수의 크기보다 큰 리터럴 값을 저장하면 컴파일 오류가 발생한다.

```java
float f = 9999.88; // 컴파일 오류
```

단, 이미 잘못된 리터럴 값을 저장하면 컴파일 오류가 발생하지 않고, 잘린 값이 저장된다. 이것은 리터럴로 값이 표현되는 단계에서 이미 잘린 값을 변수에 저장하기 때문이다.

```java
    d = 99999.8888877777f;
    System.out.println(d); // 99999.890625
```

리터럴보다 값이 큰 변수에 저장하면, 계산 방식에 의해 소수점 이하의 수가 근삿값으로 변환된다. 따라서 4바이트 리터럴은 float에, 8바이트 리터럴은 double에 저장하는 편이 좋다.

```java
    d = 99999.88f;
    System.out.println(d); // 99999.8828125
```

각 변수의 값이 개별적으로 적절히 저장될 수 있는 값이라도, 두 변수의 연산 결과가 해당 타입의 크기를 벗어나면 그 결과 값이 잘리게 된다. 

```java
    float f1 = 99988.88f;
    float f2 = 11.11111f;
    float f3 = f1 + f2;
    System.out.println(f3); // 99999.99
```

따라서 부동소수점을 다룰 때에는 가능한 float보다 두배 정밀한 double을 사용하는 편이 좋다.

### 크기가 다른 변수끼리 값 할당

값의 유효 여부에 상관없이 메모리 크기가 큰 변수의 값을 그보다 작은 변수에 저장할 수 없다.

```java
float f;
double d = 3.14;
f = d; // 컴파일 오류
```

___

## 문자 변수

### 변수의 메모리 크기

자바는 UCS(ISO10646)라는 국제 표준 인코딩 규격에 따라 문자를 다룬다. UCS-2라고도 부르며, 각 글자를 0~65535까지의 16비트(2바이트) 코드 값으로 저장한다.

char 변수에 코드 값을 수로 저장할 수 있으며 2바이트 크기의 메모리 범위를 초과하면 컴파일 오류가 발생한다.

```java
    char c;

    c = 0;
    c = 65535;

    c = -1; // 컴파일 오류 발생!
    c = 65536; // 컴파일 발생!
```

### UCS-2 문자 코드 값 저장 

작은 따옴표를 사용하면 특정 문자에 해당하는 코드 값을 리턴받을 수 있으므로 다음과 같이 간단하게 문자를 변수에 저장할 수가 있다.

```java
    char c = 'A';  // c 변수에 저장되는 것은 문자 'A'의 UCS-2 코드 값이다.
    System.out.println(c); // A
```

마찬가지로 원하는 문자의 코드 값을 정수 변수에 저장하고 싶다면 작음 따옴표를 사용하여 바로 저장이 가능하다.

```java
    int i = 'A';
    System.out.println(i); // 65
```

___

## 논리값 변수

### 변수의 메모리 크기

논리 값을 담을 변수는 JVM 내부에서 4바이트 크기의 int 타입 메모리를 사용한다.

```java
    boolean b1, b2;
    
    b1 = true; // 실제 메모리에는 1을 저장한다.
    b2 = false; // 실제 메모리에는 0을 저장한다.

    System.out.println(b1); // true
    System.out.println(b2); // false
```

> **논리값 변수에 정수 저장?**
>
> JVM 내부에서 true, false를 정수 값으로 다룬다고 해서 boolean에 직접 1과 0을 저장할 수는 없다.
>
> ```java
>     boolean b1 = 1; // 컴파일 오류!
>     boolean b2 = 0; // 컴파일 오류!
> ```

___

## 레퍼런스 변수


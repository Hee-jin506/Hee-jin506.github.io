---
layout: post
title: "[모던 JavaScript 튜토리얼] 2.5 자료형"
categories:
  - Javascript
tags:
  - 모던 JavaScript 튜토리얼s

---

> **[모던 JavaScript 튜토리얼](https://ko.javascript.info/)을 읽고 정리하였습니다.**

# 자료형

자바스크립트에서 값은 항상 문자열이나 숫자형 같은 특정한 자료형에 속한다. 자바스크립트에는 여덟 가지 기본 자료형이 있으며 이 파트에서는 이 유형들을 간단히 소개한다.

자바스크립트의 변수는 어떤 자료형의 데이터든 할당될 수 있다.

```javascript
// no error
let message = "hello";
message = 123456;
```

이처럼 자료의 타입은 있지만 저장되는 값의 타입은 언제든지 바꿀 수 있는 언어를 **동적 타입(dynamically typed) 언어**라고 부른다.

___

### 숫자형

```javascript
let n = 123;
n = 12.345;
```

숫자형(number type)은 정수 및 부동소수점 숫자(floating point number)를 나타낸다. 숫자형과 관련된 연산은 +, -, /, * 등이 대표적이다. 숫자형엔 일반적인 숫자 외에 Infinity, -Infinity, NaN 같은 특수 숫자 값(special numeric value)이 포함된다.

- Infinity는 어떤 숫자보다 큰 특수 값, 무한대를 나타낸다. 어느 숫자든 0으로 나누면 무한대를 얻을 수 있다.

```javascript
alert( 1 / 0 );
```

Infinity를 직접 참조할 수도 있다.

```javascript
alert( Infinity ); 
```

- NaN은 계산 중에 에러가 발생했다는 것을 나타내주는 값이다. 부정확하거나 정의되지 않은 수학 연산을 사용하면 계산 중에 에러가 발생하여 NaN이 반환된다.

```javascript
alert( "숫자가 아님" / 2 );
```

- NaN은 여간해선 바뀌지 않는다. NaN에 어떤 추가 연산을 해도 결국 NaN이 반환되므로, 연산 과정 중간에 NaN이 반환되었다면 이는 모든 결과에 영향을 미친다.

```javascript
alert( "숫자가 아님" / 2 + 5 );
```

> 수학 연산은 안전하다.
>
> 자바스크립트에서 행해지는 수학 연산은 안전하다. 여기서 안전하다는 말의 의미는 0으로 나누거나 숫자가 아닌 문자열을 숫자로 취급하는 등의 이례적인 연산이 자바스크립트에서는 가능하다는 것이다. 다만 NaN을 반환한다.

___

### BigInt

내부 표현 방식 때문에 자바스크립트에선 (2^53^-1)`(`9007199254740991) 보다 큰 값 혹은 -(2^53^-1)보다 작은 정수는 숫자형을 사용해 나타낼 수가 없다. 따라서 암호 관련 작업과 같이 아주 큰 숫자가 필요한 상황이거나 아주 높은 정밀도로 작업을 해야 할 때는 이런 큰 숫자가 필요하다. 

BigInt 형은 표준으로 채택된지 얼마 안된 자료형으로, 길이에 상관없이 정수를 나타낼 수 있으며, 정수 리터럴 끝에 n을 붙이면 만들 수 있다.

```javascript
const bigInt = 1234567890123456789012345678901234567890n;
```

 >**호환성 이슈**
 >
 >현재(2021-01-13)는 인터넷 익스플로러만이 BigInt를 지원하지 않고 있다.

___

### 문자형

자바스크립트에선 문자열(string)을 따옴표로 묶는다.

```javascript
let str = "Hello";
let str2 = 'Single quotes are ok too';
let phrase = `can embed another ${str}`;
```

따옴표는 세 종류가 있다.

1. 큰따옴표: "Hello"
2. 작은따옴표: 'Hello'
3. 역따옴표: \`Hello\`

큰 따옴표와 작은 따옴표는 기본적인 따옴표로 자바스크립트에서는 이 둘에 차이를 두지 않는다. 한편, 역 따옴표로 문자열을 감싼 후 원하는 변수나 표현식을 ${} 안에 넣어주면, 아래와 같이 원하는 변수 값이나 표현식의 결과 값을 문자열 중간에 손쉽게 넣을 수가 있다.

```
let name = "John";

alert( `Hello, ${name}!` ); // Hello, John!

alert( `the result is ${1 + 2}` ); // the result is 3
```

>**글자형**
>
>자바스크립트는 character와 같은 글자형을 지원하지 않으며 문자형만이 존재한다. 

___

### 불린형

불린형(논리 타입)은 true와 false 두 가지 값 밖에 없는 자료형이다.

```javascript
let nameFieldChecked = true;
let ageFieldChecked = false; 
```

 불린형은 비교 결과를 저장할 때에도 사용된다.

```javascript
let isGreater = 4 > 1;
alert( isGreater );
```

___

### null

null 값은 지금까지 소개한 자료형 중 어느 자료형에도 속하지 않는 값이다. null 값은 오로지 null 값만 포함하는 별도의 자료형을 만든다. 

```javascript
let age = null;
```

자바스크립트의 null은 자바스크립트 이외 언어의 null과 성격이 다르다. 다른 언어에선 null을 존재하지않는 객체에 대한 참조, 혹은 널포인터(null pointer)를 나타내는 데 사용되지만, 자바스크립트에서는 존재하지 않는(nothing) 값, 비어있는(empty) 값, 알 수 없는(unknown) 값을 나타내는 데 사용한다.

`let age = null;`은 나이(age)를 알 수 없거나 그 값이 비어있음을 보여준다.

___

### undefined

undefined 값도 null 값처럼 자신만의 자료형을 형성하며 '값이 할당되지 않은 상태'를 나타낼 때 사용한다. 변수는 선언했지만, 값을 할당하지 않았다면 해당 변수에 undefined가 자동으로 할당된다. 개발자가 변수에 undefined를 명식적으로 할당하는 것도 가능은 하다. 그러나 이렇게 직접 할당하는 것은 권장되지 않으며, 값이 할당되지 않은 변수의 초기값을 위해 예약어로 남겨둔다.

```javascript
let age;
alert(age);

age = undefined;
alert(age);
```

____

### 객체와 심볼

객체(object)형은 특수한 자료형이다. 객체형을 제외한 다른 자료형은 문자열이든 숫자든 한 가지만 표현할 수 있으므로 원시(primitive) 자료형이라고 부른다. 

반면 객체는 데이터 컬렉션이나 복잡한 개체(entity)를 표현할 수가 있다. 

심볼(symbol)형은 객체의 고유한 식별자(unique identifier)를 만들 때 사용된다.

___

### typeof 연산자

typeof 연산자는 인수의 자료형을 반환한다. 자료형에 따라 처리 방식을 다르게 하고 싶거나, 변수의 자료형을 빠르게 알고자 할때 유용하다.

typeof 연산자는 두 가지 형태의 문법을 지원한다.

1. 연산자: typeof x
2. 함수: typeof(x)

결과는 모두 인수의 자료형을 나타내는 문자열을 반환한다.

```javascript
typeof undefined // "undefined"
typeof 0 // "number"
typeof 10n // "bigint"
typeof true // "boolean"
typeof "foo" // "string"
typeof Symbol("id") // "symbol"
typeof Math // "object"  (1)
typeof null // "object"  (2)
typeof alert // "function"  (3)
```

위의 예시에서 마지막 세 줄에 대한 설명은 다음과 같다.

1. Math는 수학 연산을 제공하는 내장 객체이므로 object가 출력된다.
2. typeof null의 결과는 object이다. null은 별도의 고유한 자료형을 가지는 특수 값으로 객체는 아니지만, 하위 호환성을 유지하기 위해 이런 오류를 수정하지 않았다.
3. typeof는 피연산자가 함수면 "function"을 반환한다. 그런데 함수형은 따로 없으며 함수는 객체형에 속한다. 이런 동작 방식이 형식적으로 잘못되기는 했으나, 아주 오래전에 만들어진 규칙이므로 하위 호환성을 위해 오류를 수정하지 않고 남겨두었다.
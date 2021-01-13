---
layout: post
title: "[모던 JavaScript 튜토리얼] 2.4 변수와 상수"
categories:
  - Javascript
tags:
  - 모던 JavaScript 튜토리얼s
---

> **[모던 JavaScript 튜토리얼](https://ko.javascript.info/)을 읽고 정리하였습니다.**

# 변수와 상수

대다수의 자바스크립트와 애플리케이션은 사용자나 서버로부터 입력받은 정보를 처리하는 방식으로 동작하며, 자바스크립트의 변수는 이 정보를 저장하는 용도로 사용된다.

___

### 변수

변수(variable)는 데이터를 저장할 때 쓰이는 '이름이 붙은 저장소'이다. 자바스크립트에서는 let 키워드를 사용해서 변수를 생성한다.

아래 문은 message라는 이름의 변수를 생성한다. 

```javascript
let message;
```

할당 연산자 = 를 통해 변수 안에 데이터를 저장할 수가 있다.

```javascript
let maessage;
message = 'Hello';
```

문자열이 변수와 연결된 메모리 영역에 저장되었으므로 변수명을 이용해서 문자열에 접근할 수가 있다.

```javascript
let message;
message = 'Hello!';

alert(message);
```

아래와 같이 변수 선언과 값 할당을 한 줄에 작성할 수도 있다.

```javascript
let message = 'Hello!'; 
alert(message);
```

한 줄에 여러 변수를 선언하는 것도 가능하다. 이렇게 작성하면 코드가 더 짧아지지만 가독성을 위해 한 줄에는 하나의 변수만 작성하는 것이 좋다.

```javascript
let user = 'John', age = 25, message = 'Hello';
```

어떤 사람은 이 한 문을 줄바꿈을 통해 여러 줄로 나누기도 한다. ','는 줄 맨 뒤나 맨 앞에 둔다.

```javascript
let user = 'John',
  age = 25,
  message = 'Hello';
  
let user = 'John'
  , age = 25
  , message = 'Hello';
```

> **var**
>
> 오래된 스크립트에서 let 대신 var라는 키워드를 사용하기도 하는데, var과 let은 거의 동일하게 동작하지만 미묘한 차이가 있는데, 이는 나중에 자세히 다룰 것이다.

변수 안의 값을 원하는 만큼 변경할 수 있다.

```javascript
let message;
message = 'Hello!';
message = 'World!';
alert(message);
```

변수 두 개를 선언하고, 한 변수의 값을 다른 변수에 복사할 수도 있다.

```javascript
let Hello = 'Hello world!';
let message;

message = Hello;

alert(Hello); // Hello world!
alert(message); // Hello world!
```

>**같은 변수를 여러 번 선언하면 에러가 발생한다.**
>
>```javascript
>let message = "This";
>let message = "That"; 
>// SyntaxError: 'message' has already been declared
>```

>**함수형 언어**
>
><u>함수형(functional)</u> 프로그래밍 언어는 변숫값 변경을 금지한다. 이 언어에서는 변수에 값이 저장되면 그 값을 영원히 유지한다. 다른 값을 저장하고 싶다면 새로운 변수를 선언해야하며 기존 변수를 재사용할 수는 없다.
>
>함수형 언어의 이러한 제약은 병렬 계산(parallel computation)과 같은 영역에서 장점으로 작용한다. 

___

### 변수 명명 규칙

자바스크립트에선 변수 명명 시 두 가지 제약 사항이 있다.

1. 변수명에는 오직 문자와 숫자, 기호 $와 _만 들어갈 수 있다.
2. 첫 글자는 숫자가 될 수 없다.

유효한 변수명의 예시이다.

```javascript
let userName;
let test123;
```

여러 단어를 조합하여 변수명을 만들 때에는 카멜 표기법(camelCase)이 흔히 사용된다. 카멜 표기법은 단어를 차례대로 나열하면서 첫 단어를 제외한 각 단어의 첫 글자를 대문자로 작성한다.

다음과 같은 변수명도 유효하다.

```javascript
let $ = 1; 
let _ = 2; 

alert($ + _); // 3
```

아래는 잘못된 변수명의 예시이다.

```javascript
let 1a; // 숫자로 시작
let my-name; // '-' 사용
```

> **자바스크립트는 변수명에 있어서 대소문자를 구별한다.**

> **비 라틴계 언어도 변수명에 사용할 수 있지만 권장하진 않는다.**

> **예약어**
>
> 예약어(reserved name) 목록에 있는 단어는 자바스크립트 내부에서 이미 사용중이므로 변수명으로 사용할 수가 없다.
>
> ex) let, class, return, function
>
> 아래 코드는 문법 에러를 발생시킨다.
>
> ```javascript
> let let = 5;
> let return = 5;
> ```

> **use strict 없이 할당하기**
>
>변수는 대개 정의되어있어야 사용할 수 있다. 그러나 예전에는 let 없이 값을 바로 할당하여 변수를 생성할 수가 있었다. use strict를 쓰지 않으면 과거 스크립트와의 호환성을 유지할 수 있으므로 여전히 이 방식을 사용할 수 있다.
>
>```javascript
>// 참고: 이 예제에는 "use strict"가 없습니다.
>
>num = 5;
>```

___

### 상수

변화하지 않는 변수, 즉 상수를 선언할 땐, let 대신 const 사용한다. 상수는 재할당할 수 없으므로 상수를 변경하려고 하면 에러가 발생한다.

```javascript
const myBirthday = '18.04.1982';
myBirthday = '01.01.2001'; // error, can't reassign the constant!
```

___

### 대문자 상수

 다음과 같이 기억하기 힘든 값은 상수에 할당하는데, 이때 상수명을 오직 대문자와 밑줄로 구성하는 것이 관습이다. 

```javascript
const COLOR_RED = "#F00";
const COLOR_GREEN = "#0F0";
const COLOR_BLUE = "#00F";
const COLOR_ORANGE = "#FF7F00";
```

대문자로 상수를 만들어 사용하면, 값을 그대로 기억하는 것보다 훨씬 용이하게 값을 사용할 수가 있고, 가독성도 높아진다.

따라서 코드가 실행되기 전부터 이미 값을 아는 상수는 대문자 상수로 선언하고, 그 이외의 런타임에서 최초 계산 결과를 더이상 바꾸지 않을 때에는 일반적인 방법으로 상수명을 지정하면 된다. 즉, 대문자 상수는 하드 코딩한 값의 별칭을 만들 때 사용한다는 것이다.

___

### 바람직한 변수명

변수명은 간결하고, 명확해야 한다. 변수가 담고 있는 것이 무엇인지 잘 설명할 수 있어야 한다. 아래는 변수 명명 시 참고하기 좋은 규칙들이다.

-  사람이 읽을 수 있는 이름을 사용해라.
- 무엇을 하고 있는지 명확히 알고 있지 않을 경우 외에는 줄임말이나 a,b,c와 같은 짧은 이름은 피하라.
- 최대한 서술적이고 간결하게 명명해라. 코드 문맥상 변수가 가리키는 데이터나 값이 아주 명확할 때에만 data, value 같은 이름을 사용하며 그 이외에는 자제해라.
- 자신만의 규칙이나 소속된 팀의 규칙을 따라야 한다. 

> **재사용 vs 선언**
>
> 새로운 변수를 선언하기보다 기존 변수를 재사용하는 것을 선호하는 개발자가 종종 있다. 변수를 재사용하면 변수 선언에 쏟는 노력을 덜 수 있으나, 디버깅에 더 많은 시간을 쏟아야 한다. 따라서 변수를 추가하는 것은 좋은 습관이다. 모던 자바스크립트 압축기(minifier)와 브라우저는 코드 최적화를 잘해주고 있으므로 변수를 몇가지 더 추가한다고 해서 성능 이슈가 생기지 않는다.
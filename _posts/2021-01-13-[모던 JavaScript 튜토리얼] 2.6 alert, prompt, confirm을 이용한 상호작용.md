---
layout: post
title: "[모던 JavaScript 튜토리얼] 2.6 alert, prompt, confirm을 이용한 상호작용"
categories:
  - Javascript
tags:
  - 모던 JavaScript 튜토리얼s
---

> **[모던 JavaScript 튜토리얼](https://ko.javascript.info/)을 읽고 정리하였습니다.**

# alert, prompt, confirm을 이용한 상호작용

이번에는 브라우저 환경에서 사용되는 최소한의 사용자 인터페이스 기능인 alert, prompt, confirm에 대해 알아본다.

___

### alert

alert 함수는 앞선 예제에서 살펴본 바가 있다. 이 함수가 실행되면 사용자가 확인(OK) 버튼을 누를 때까지 보여주는 창이 뜬다.

```javascript
alert("Hello");
```

메시지가 있는 작은 창은 모달 창(modal window)이라고 부른다. 모달이라는 단어는 페이지의 나머지 부분과 상호작용이 불가능하다는 의미를 내포하므로 사용자는 모달 창을 끄기전까지는 모달 창 밖에 있는 버튼을 누르는 등의 행동을 할 수가 없다.

___

### prompt

브라우저에서 제공하는 Prompt 함수는 두 개의 인수를 받으며 함수가 실행되면 텍스트 메시지와 입력 필드(input field), 확인(OK) 및 취소(Cancel) 버튼이 있는 모달창을 띄워준다.

```javascript
result = prompt(title, [default]);
```

- `title`: 사용자에게 보여줄 문자열
- `default`: 입력 필드의 초깃값(선택값)

> 인수를 감싸는 대괄호 []의 의미
>
> default를 감싸는 대괄호는 이 매개변수가 필수가 아닌 선택값이라는 것을 의미한다.

사용자는 프롬프트 대화상자의 입력 필드에 원하는 값을 입력하고 확인을 누른다. 혹은 취소 버튼을 누르거나 esc를 눌러 대화상자를 빠져나갈 수도 있다.

prompt 함수는 사용자가 입력 필드에 적은 문자열을 반환한다. 사용자가 입력을 취소한 경우는 null이 반환된다. 두 번째 매개변수에 특정 값을 지정하면, 모달창이 뜰 때 이미 입력필드에 해당 값이 작성되어 있게 된다.

```javascript
let age = prompt('나이를 입력해주세요.', 100);
alert(`당신의 나이는 ${age}살 입니다.`); 
```

> **Internet Explorer에서는 항상 기본값을 넣어주어야 한다.**
>
> 프롬프트 함수의 두번째 매개변수는 선택사항이지만, IE는 매개변수를 title 하나만 지정해줄 경우, undefined를 입력 필드에 명시한다. 
>
> ```javascript
> let test = prompt("Test"); // IE에서는 입력 필드에 undefined가 뜬다.
> ```
>
> IE 사용자를 비롯한 모든 사용자에게 깔끔한 프롬프트를 보여주려면 아래와 같이 두 번째 매개 변수를 항상 전달해 줄 것을 권장한다.
>
> ```javascript
> let test = prompt("Test", '');
> ```

___

### 컨펌 대화상자

confirm 함수는 매개변수로 받은 question과 확인 및 취소 버튼이 있는 모달 창을 보여준다. 사용자가 확인 버튼을 누르면 true, 그 외의 경우에는 false를 반환한다.

```javascript
let isBoss = confirm("당신이 주인인가요?");
alert( isBoss );
```


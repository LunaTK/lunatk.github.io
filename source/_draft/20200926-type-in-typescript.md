---
title: 타입스크립트 이해하기
category:
  - Guide 
  - TypeScript
toc: true
comments: true
date: 2020-09-26 12:42:04
tags:
  - typescript
  - type_system
thumbnail:
---

타입스크립트는 편의성과 낮은 러닝커브 덕분에 많은 자바스크립트 개발자들에게 사랑받고 있습니다. 하지만 다른 프로그래밍 언어를 배울때와 마찬가지로, 타입스크립트를 처음 배울때 사용법과 예제를 무작정 암기해 배우는 경우가 많습니다. 이 글에서는 타입 시스템의 기능을 단순히 나열하고 암기를 강요하는 대신, 타입스크립트에서 타입 시스템이 어떤 자리를 차지하고 있고 그 근본이 무엇인지에 대한 이해를 예시를 통해 알아 보고자 합니다.

<!--More-->

# What is TypeScript?

> TypeScript = JavaScript + JSNext + *Type System*

<center>
  ![TypeScript Diagram](https://raw.githubusercontent.com/basarat/typescript-book/master/images/venn.png)
  <small>
    [TypeScript is JavaScript](https://basarat.gitbook.io/typescript/recap)
  </small>
</center>

타입스크립트는 자바스크립트의 확장판으로, 정적 타입 시스템과 차세대 자바스크립트 문법이 추가된 버전 정도로 생각할 수 있습니다.

일반적으로 타입스크립트를 도입해서 얻을 수 있는 장점은 다음과 같이 알려져 있습니다.
1. IDE에서 자동완성 제공
2. 협업(Teamwork)에 용이
3. 버그 감소 및 조기 발견

실제로 Airbnb는 타입스크립트를 도입해 버그가 38% 정도 감소할 수 있었다고 보고하기도 하였습니다[^1]. 

물론 타입스크립트를 도입하는데는 비용(학습, 개발환경 셋팅)이 들기때문에, 이를 감수하면서 까지 타입스크립트를 써야하는지[^2], 실제 타입스크립트가 버그 방지에 어느정도 효과가 있는지[^3] 등에 대한 논쟁은 아직까지 이어지고 있는건 사실입니다. 

그럼에도 편의성과 버그방지 효과가 크다고 느끼는 사람이 많아서인지, 많은 프로젝트에서도 널리 쓰이고 있고 기업에서도 타입스크립트 개발자를 우대하기 때문에 꼭 배워야 하는 언어라고 말씀드릴 수 있습니다.

# 일반적인 타입스크립트 학습법

타입스크립트 기본 문법 골격이 자바스크립트와 동일이기 때문에, 자바스크립트를 알고 있는 개발자라면 타입스크립트를 배우는것은 정말 간단합니다.

극단적인 예시로, 여러분이 가지고 있는 자바스크립트 프로젝트 폴더를 열어서 모든 `.js` 파일 확장자를 `.ts`로 바꾸고, 변수 선언에 뻔한 타이핑(`any`)만 몇개 추가한 다음 이제 나도 타입스크립트 개발자! 라고 해도 아주 틀린말은 아니라 볼 수 있습니다. (물론 아주 잘못된 타입스크립트 사용 사례가 되겠지요😅)

<center>
  ![Typescript vs Javascript](https://miro.medium.com/max/1374/0*6l6YgwULnhwwTDKx.png)
  <small>
    [TypeScript vs JavaScript](https://medium.com/front-end-weekly/typescript-vs-javascript-a3c0beb8b6d9)
  </small>
</center>

실제로 근사한 라이브러리를 만들게 아니라면 위 그림처럼 타이핑 정도만 추가하더라도 타입스크립트를 쓰는데 큰 문제가 없다고 생각합니다.

이렇듯 타입스크립트를 배우는건 어렵지 않기 때문에, 타입에 어떤 종류가 있고, `interface`, `type` 등을 어떻게 쓰는지 정도만 간단히 알아보고 일단 프로젝트를 만들어 보며 부딪히며 배우는 사람들이 많습니다.

그렇게 야매로 배운 타입스크립트 지식을 가지고 개발을 하다 보면, 타입 에러를 마주했을때 왜 그 에러가 발생했는지 이해하지 못해 구글링에 에러를 그대로 검색하게 됩니다. 그리고 운 좋게 찾으면 해결하고 못찾으면 결국 `any`로 도배하는 상황, 한번쯤 겪어 보았을 것입니다.

<center>
  {% asset_img type-error.png %}
  <small>
    [타입에러 예시](https://stackoverflow.com/questions/60993961/typescript-typing-3rd-party-library-xxx-only-refers-to-a-type-but-is-being)[^2]
  </small>
</center>

만약 여러분이 저 사진만 보고 에러의 원인이 무엇인지 정확히 설명할 수 있다면, 타입스크립트에서의 타입에 대해 탄탄한 이해를 가지고 있다고 할 수 있으며 굳이 이 글의 나머지 부분을 읽지 않으셔도 되는 훌다한 타입스크립트 개발자라 생각됩니다.

나중에 다시 설명하겠지만, 위 에러는 **자바스크립트 값**을 넣어야 할 부분에 **타입스크립트 타입**을 넣어서 발생하는 에러입니다.

무슨 말인지 아직 이해가 안되시나요?

걱정하지 마세요. 이제부터 타입스크립트에서 타입이 무엇인지, 또 어떻게 작동하고 어떤걸 가능하게 해주는지 지금부터 알아 볼 것이며, 이 글을 다 읽고난 뒤에는 여러분도 자신있게 대답할 수 있을것입니다.

# 이 글의 목적

이 글을 읽고 난 다음 여러분이 다음과 같은 지식을 얻어가길 바랍니다.
- 타입스크립트의 타입 시스템이 무엇이며 어떻게 작동하는지
- 어디까지가 자바스크립트이고 어디부터가 타입스크립트인지 구분할 수 있는 능력

이 글에서는 다음과 같은 지식을 제공하지 않습니다.
- 타입스크립트에 있는 모든 기능
- 타입스크립트에서 제공하는 새로운 자바스크립트 문법 (JSNext)

# 타입스크립트에서의 타입

구체적인 설명에 앞서, 결론부터 요약해 보겠다.

우선, 타입스크립트에서의 타입 시스템은 자바스크립트 언어와는 완전히 별개의 것으로 보아야 한다. 아래의 공식을 기억하자.

> 타입스크립트 = 자바스크립트 + 타입 시스템

당연한 공식이지만, 타입스크립트 문법이 자바스크립트와 버무러져 있고 상당부분이 자바스크립트이기 때문에 타입스크립트의 본질을 놓치기가 쉽다.

타입스크립트의 본질은 자바스크립트가 아니라 "타입 시스템"이다. 그리고 이 타입 시스템은 타입을 만드는 **또하나의 프로그래밍 언어**이다.

타입스크립트의 타입 시스템은 자바스크립트의 문법과는 완전히 별개의 문법으로 보아야 한다.

아직은 이해가 안되겠지만, 아래의 세가지 관점에서 왜 타입 시스템이 또하나의 프로그래밍 언어인지를 설명해 보겠다.

1. 타입스크립트 타입은 값이며, 이는 자바스크립트에서의 값과 다른것이다. 
2. 타입 시스템은 타입 끼리의 연산을 수행하는 시스템이다.
3. 제네릭 등의 기능은 타입을 만드는 타입 생성 함수 이다.


# 타입스크립트에서의 선언 공간

**선언 공간**(Declaration Space)에 대해 알아보기 전에, **선언**(Declaration)에 대한 정의를 먼저 생각해 보자.

## 선언(Declaration) 이란?

선언에 대한 정의는 완벽하진 않지만 대략적으로 아래와 같이 생각해 볼 수 있다.

> In programming, a declaration is a statement describing an identifier, such as the name of a variable or a function.[^1]
> 
> 프로그래밍에서 선언은 변수나 함수의 **이름**과 같은 **식별자(identifier)를 기술하는 구문이**다.

즉, 프로그래밍 언어 자체에서 예약된 keyword(`for`, `while` 등등)을 제외한, **프로그래머가 어떤 것에 이름을 부여하는 행위**를 선언 이라고 할 수 있다.

## 자바스크립트에서의 선언

그렇다면 아래 자바스크립트 코드에서 declaration은 어떤게 있을까?

```javascript
var name = 'Alice';

function hello(to) {
  console.log('Hello,', to);
}
```

위에서 언급한 선언의 정의에 따르면, 우리가 위 코드에서 부여한 **이름**에는 `name`과 `hello`, 그리고 `to` 가 있다.
(참고로 `console`, `log` 또한 어딘가에 **선언**이 되어 있겠지만, 위 코드에서 선언한것은 아니므로 생략.)

# References

[^1]: [Airbnb think 38% of their bugs could have been preventable by using Typescript](https://www.reddit.com/r/typescript/comments/c079bt/airbnb_think_38_of_their_bugs_could_have_been/)
[^2]: [The TypeScript Tax](https://medium.com/javascript-scene/the-typescript-tax-132ff4cb175b)
[^3]: [To Type or Not to Type: Quantifying Detectable Bugs in JavaScript](https://www.semanticscholar.org/paper/To-Type-or-Not-to-Type%3A-Quantifying-Detectable-Bugs-Gao-Bird/955baf8c8a2a42a78aca39fc5e755b8d7536636a)
[^2]: [https://www.computerhope.com/jargon/d/declare.htm](https://www.computerhope.com/jargon/d/declare.htm)
[^3]: [타입에러 예시](https://stackoverflow.com/questions/60993961/typescript-typing-3rd-party-library-xxx-only-refers-to-a-type-but-is-being)
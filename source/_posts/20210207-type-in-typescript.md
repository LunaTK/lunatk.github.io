---
title: 타입스크립트에서 타입
tags:
  - typescript
  - type-system
category:
  - Programming Language
  - null
thumbnail: 'https://secure.meetupstatic.com/photos/event/1/c/1/9/highres_483427193.jpeg'
toc: true
comments: true
date: 2021-02-07 12:42:04
---


타입스크립트를 쓰다보면 타입을 다루는 법을 암기해서 쓰는 경우가 많다. 나 또한 그랬는데, 타입에 대한 개념을 정확히 이해하고 사용하니 더 자연스럽게 사용할 수 있게 되었다. 내가 이해한 타입스크립트에서 타입을 공유해 보고자 한다.

<!--More-->

# 값 공간과 타입 공간

타입스크립트 문법에서는 두가지 공간 있다. [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/project/declarationspaces)[^1]에 관련 내용이 잘 설명되어 있다.

1. **값 공간** (value space)
   - 스트링, 숫자, 객체 등 구체적인 값이 들어가는 공간
   - 타입스크립트가 자바스크립트로 변환되어도 유지
2. **타입 공간** (type space)
   - 타입이 들어가는 공간
   - 타입스크립트가 자바스크립트로 변환되면 사라진다 (`class`, `enum`은 예외)

공간은 쉽게 생각해 타입스크립트 코드에서 **Named Entity** 또는 **리터럴**(Literal)이 올 수 있는 자리이다. 

Named Entity와 리터럴의 뜻은 아래와 같다.
- Named Entity : 변수, 함수, 타입 등등 **특정 이름으로** 선언된 것들.
- 리터럴 : 스트링 리터럴, 숫자 리터럴, 타입 리터럴 등 하드코딩된 상수값들을 의미한다. (ex. `x = "foobar"`, `y = 123` 에서 우변들)

우리가 일반적으로 변수나 함수를 선언하면 이들은 "값 공간"에 선언되고, 타입을 선언하면 "타입 공간"에 선언이 된다. 리터럴은 컴파일러가 "값 공간"에 속하는 리터럴인지 "타입 공간"에 속하는 리터럴인지 문맥에 따라 판단한다.

아래는 리터럴과 Named Entity가 사용(할당, 함수 매개변수로 전달)될 때, 각 자리가 어느 공간으로부터 채워지는지를 나타낸 표이다.

||값 공간|타입 공간|
|:----:|:----:|:-----:|
|Literal|let x: string = `"Alice"`|type Human = `{name: string, age: number}`|
|Named Entity|let x: string = `myName`;|type MyType = `Human \| null`|

위 표에서, 변수 선언을 할 때 우변에 반드시 "값 공간"에 속한 Named Entity 또는 리터럴이 와야하며,
반대로 타입 선언을 할 때는 우변에 반드시 "타입 공간"에 속한 Named Entity 또는 리터럴이 와야함을 알 수 있다.

> 타입스크립트 코드에서, 타입이 와야하는 자리에는 타입 공간에 속한 것들이, 값이 와야하는 자리에는 값 공간에 속한 것들이 위치해야 한다.

매우 당연한 소리같아 보이지만, 이 간단한 규칙만 기억해도 아래와 같은 타입 에러를 예방할 수 있다.

<center>
  {% asset_img type-error.png %}<br/>
  <small>
    타입 공간과 값 공간을 혼용해 발생하는 에러
  </small>
</center>

하지만 왠지 타입스크립트에서는 위 룰을 위반하는 행위, 즉 값이 와야하는 자리에 타입이 오거나 타입이 와야하는 자리에 값이 오는 행위가 유효한 것처럼 느껴지며, 실제로 그런 착각을 일으키는 코드가 있기 때문이다.

# 헷갈리는 부분들 (예외케이스)

```typescript
// #1. Literal Type
type AMPM = "AM" | "PM";  // 타입 공간에 쓰이는 string
const ampm = "AM";        // 값 공간에 쓰이는 string

// #2. Class
class SomeClass {}
type MyClass = SomeClass; // 타입 공간에 쓰이는 class
const MyValue = SomeClass;// 값 공간에 쓰이는 class
```

위와 같이 타입 공간과 값 공간의 경계를 허물어 버리는 것 같은 코드가 있다. 하지만 이들은 약간 예외적인 케이스이다.

1. Literal Type이라고, 가질 수 있는 값을 특정 리터럴 들로만 제한하는 문법이다. 값 공간의 리터럴이 타입으로 쓰인 예외라고 볼 수 있다.

2. `class`와 `enum`은 선언시 값 공간과 타입 공간에 모두 선언되는 예외적인 케이스이다.

# 공간 변환

## 값을 타입으로 변환

간혹 어떤 값의 타입을 명시적으로 쓰고싶은데, 타입을 뭐라 해야할지 모르는 경우가 있다.

예를들어, 다음 코드에서 `???`에 와야 할 타입은 무엇일까?

```typescript
import React from 'react';

const myReact: ??? = React;     // 1
```

```typescript
class SomeClass {}

const MyClass: ??? = SomeClass; // 2
```

여기서 잠깐 멈추고, 한번 어떻게 해야하는지 생각해보자. VSCode에서 뜨는 타입 힌트를 안보고 답을 알겠다면 타입스크립트를 훌륭히 숙지하고 있다고 볼 수 있으며, 이 글은 재미삼아 읽고 있을 확률이 높다.

1번의 경우, import된 React는 `namespace`이다. 타입스크립트에서 `namespace`는 사실 객체, 즉 값 공간의 것이다.

2번의 경우, SomeClass가 타입 공간에도 속하므로 그냥 SomeClass를 쓰면 안되나 생각할 수 있겠지만, SomeClass가 타입으로 쓰일때 가리키는것은 SomeClass의 인스턴스이지 클래스 그 자체가 아니라 안된다.

이러한 경우 값에 해당하는 타입으로 변환해주는 마법의 연산자 `typeof`를 사용하면 된다.

```typescript
import React from 'react';

const myReact: typeof React = React;     // 1
```

```typescript
class SomeClass {}

const MyClass: typeof SomeClass = SomeClass; // 2
```

쉽게 생각해 `typeof 값` 을 하면 값이 가지는 타입이 그 자리에 들어가며, 타입 공간이 와야하는 자리에 값을 쓸 수 있게 해준다 볼 수 있다.
`typeof`는 `declaration(d.ts)` 파일을 작성할 때 매우 유용하게 쓸 수 있다.

## 타입을 값으로 변환

값을 타입으로 바꾸는것은 가능하지만, 타입을 구체적인 값으로 바꾸는것은 불가능하다.
물론 `class`나 `enum`은 값 공간과 타입 공간 둘 모두에 속하기 때문에 변환하지 않더라도 쓸 수 있다. 둘은 **타입**이기도 하면서, 인스턴스를 만드는 **생성자 함수**이기도 하기 때문이다.
```typescript
class SomeClass {
  // 생략...
}

type MyType = SomeClass | null; // valid
const myValue = SomeClass;      // valid
```

하지만 아래와 같이 타입을 값으로 쓰는건 안된다.

```typescript
const myBullshit = MyType;      // invalid
```

이건 생각해보면 당연한것이
- 값과 타입은 (N:1) 매칭이라 값->타입 변환은 결과가 유일하지만, 타입과 값은 (1:N) 매칭이라 타입->값 변환을 하려하면 N개중 하나로 결정할 수 없다는 문제가 발생한다.
- 저런 코드는 자바스크립트로 변환시 어떻게 되어야 하는가? 타입을 나타내는 문법은 자바스크립트에 존재하지 않는다.

# 타입 공간에서 타입은 값

지금까지 값 공간과 타입 공간의 분리에 대해 알아보았다.

그런데 놀라운 점은, 타입 공간 안에서만 보면 타입도 결국 타입을 나타내는 "값"이라는 것이다.
왜냐하면 타입에도 연산(operation)을 할 수 있기 때문이다. 물론 "타입 공간에서 값"과 "값 공간에서의 값"은 다른 것이며 섞어서 연산할 수 없다.

## 값은 연산을 할 수 있다

연산은 "값"을 다른 "값"으로 바꾸는 행위이다. 예를들어 숫자에 덧셈을 하면, 숫자가 바뀐다.

> 마찬가지로 타입에 "**타입 연산**"을 하면 다른 타입으로 바뀐다.

## 타입 연산

그렇다면 타입 연산에는 무엇이 있을까? 놀랍게도 여러분은 이미 타입 연산을 쓰고있다.

```typescript
type MyType = SomeClass | null;             
type MyType = SomeClass & { field: string };
```

`|` 연산자와 `&` 연산자가 바로 그것이다. 각각은 곱 타입과 합 타입을 만들어 내는 가장 기초적인 **타입 연산자**이다.

이 외에도 한 타입으로 다른 타입을 만드는 다양한 타입 연산이 있다. 아래는 대충 정리해 놓은것인데, 한줄 요약으로 이해하기는 쉽지 않을것이라 공식 문서를 참고하면 좋을것 같다. ([Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#inference-from-mapped-types))[^2]

- [Index Type](https://www.typescriptlang.org/docs/handbook/advanced-types.html#index-types)
  - `keyof` 연산자를 통해 오브젝트의 키로 사용되는 타입을 리턴한다.
- [Index Signature](https://www.typescriptlang.org/docs/handbook/advanced-types.html#index-types-and-index-signatures)
  - 배열 index 접근하듯이 객체 타입에 key로 접근하면 해당 key 에 대응하는 value의 타입을 리턴한다.
- [Mapped Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types)
  - 객체 타입에 각각의 필드 타입을 다른 타입으로 매핑한 객체 타입을 리턴한다.
- [Conditional Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#conditional-types)
  - 삼항 연산자 문법을 이용해 조건에 따라 다른 타입을 리턴한다.

공식 문서에서는 이러한 것들을 타입 연산이라고 소개하지는 않지만, 내 생각에 타입 연산이라 생각하고 이해하면 훨씬 도움이 될 것 같다.

# 정리

결국 중요한 핵심 개념을 정리하면 다음과 같다.

- 타입스크립트에서 타입을 써야하는 **자리**와 값을 써야하는 **자리**는 정해져 있다. **둘 모두를 사용할 수 있는 자리는 없다.**
  - `class`와 `enum`은 자리와 상관없이 값과 타입 두가지 모두로 사용될 수 있는것.

- `typeof` 연산자로 **값 공간의 것을 타입 공간의 것으로 변환**할 수 있다.
  - 타입이 와야하는 자리에 값을 쓰고싶을 때 사용.

- 타입 공간 내에서는 **타입도 결국 값**이다.
  - 한 타입을 다른 타입으로 바꾸는 **타입 연산** 가능.
  - 공식 문서의 [Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#inference-from-mapped-types)는 결국 타입 연산에 대한 설명.


이 개념을 이해한다면 타입스크립트를 사용할 때 자유 자재로 타입을 다룰 수 있을것이라 기대한다.

# References

[^1]: [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/project/declarationspaces)
[^2]: [Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#inference-from-mapped-types)
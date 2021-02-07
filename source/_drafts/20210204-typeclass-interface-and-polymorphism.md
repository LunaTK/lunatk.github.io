---
title: 타입클래스와 다형성
category:
  - Programming Language
toc: true
comments: true
date: 2021-02-04 18:56:11
tags:
thumbnail: /2021/02/04/20210204-typeclass-interface-and-polymorphism/polymorphism.png
---

타입클래스와 인터페이스, 그리고 다른 비슷한 개념들은 어떻게 서로 비슷하고 다른것일까? 다형성의 관점에서 이를 정리해 보았다.

<!--More-->

주의 : 이 글은 누군가에게 간결한 설명을 하기 위한 글이 아니라 제 생각을 주저리주저리 써놓은 글입니다.

# 문득 든 생각

프로그래밍 언어를 하나를 능숙히 다룰수 있으면, 두번째 언어 부터는 훨씬 빨리 배울 수 있다는 말이 있다.

<center>
  {% asset_img nomad_coder.png %}<br/>
  <small>
    새로운 언어를 배울때 기존에 알고있던 언어와 비교하며 배우기 (출처: <a href="https://www.youtube.com/watch?v=fqnKJa02GK0">노마드코더</a>)
  </small>
</center>

위 사진처럼 누군가에게 이러한 방법을 배우지 않았다 하더라도, 아마 많은 사람들이 새 프로그래밍 언어를 배울때 자연스럽게 자신이 알고있는 언어와 비교하며 배울것이라 생각한다.

자바를 첫 언어로 배우고, 새로운 언어를 배워 나가면서 프로그래밍 언어들의 공통점에 대한 내 인식은 아래 순서대로 변화했다.

1. C++, 파이썬 : 문법만 서로 조금씩 다르고 제공하능 기능은 다 거기서 거기네
2. Swift : `optional`? `protocol`? 람다 함수? 이런 독특한 기능을 가진 언어도 있구나
3. Kotlin : 얘도 optional이 있고 스위프트랑 엄청 비슷한데?
4. Haskell, Rust : 얘네도 `optional`이 있고, `typeclass`랑 `trait`는 `interface`랑 비슷한거같은데?

여러가지 언어를 간단히나마 공부해 보며 느낀점은, 프로그래밍 언어는 다 다른 언어들을 참고하면서 디자인 한다는 것이며 결국 제공하는 기능이 서로 비슷하다는 것이었다.

# 인터페이스, 타입클래스, 프로토콜, 트레이트, 컨셉

대부분 언어들이 비슷한 기능에 대해서는 대개 같은 이름을 갖는데(ex. `class`, `optional`), 유독 하나의 개념에 대해서 다양한 이름을 갖는것이 있었다. 바로 자바에서는 `interface` 라는 개념이었다.

이게 Swift는 `protocol`, Haskell은 `typeclass`, Rust는 `trait`, C++20는 `concept` 등등 진짜 가지각색의 이름을 가지고 있었다.

각각을 배울때는 그냥 "typeclass는 interface같은거구나" 식으로 이해하고 넘어가버렸는데, 갑자기 이름이 다 제각각인데는 이유가 있지 않을까 싶은 생각이 들었다.

실체로 찾아보니 아래처럼 각각을 비교해놓은 유튜브 영상이 있었다.

{% linkPreview https://www.youtube.com/watch?v=E-2y1qHQvTg&t=1223s _blank nofollow %}

해당 영상에서 서로가 어떻게 같고 어떻게 다른지, 그리고 이들은 다형성(Polymorphism)을 위한 기능이라 설명하고 있긴 하였으나, 뭔가 명확한 느낌이 들지 않았다.

그래서 다른 수많은 글들을 찾아가며 내가 정리한 결론은 다음과 같다.

1. 이들은 결국 다형성을 달성하기 위해 존재하는 기능들이다
   - 하지만 서로 다른 형태의 다형성을 제공한다
2. 각각이 제공하는 기능, 제약조건이 조금씩 달라 서로 이름이 다르다
   - 예를들어 스위프트의 `protocol`은 static 필드를 가질 수 있지만 자바의 `interface`는 못가진다

이를 잘 이해하기 위해 우선 다형성에는 어떤것이 있는지 알아보자

# 3가지 다형성

다형성(Polymorphism)은 하나의 자료형이 실제 가지고 있는 인스턴스에 따라 다른 행동을 할 수 있는 기능을 의미하며, 코드 재사용성을 높여준다는 장점이 있다.

다형성에는 크게 세가지 형태가 있다.
- Subtype Polymorphism
- Ad-hoc Polymorphism
- Parametric Polymorphism

각각이 어떤것이고, 앞서 언급한 `interface`, `protocol`, `typeclass` 등등은 어떤 다형성을 제공하는지 알아보자.

## Subtype Polymorphism

- 주로 객체지향 언어에서 제공하는 기능
- 상속을 통해 동작 (`interface`도 여기에 포함)

## Ad-hoc Polymorphism

Ad-hoc은 즉석의, 임시방편의 란 뜻인데, 내생각에 "야매" 라고 번역하면 딱인 단어인것 같다.
즉, 야매 다형성을 뜻한다.

- 일반적으로 오버로딩이라 이해하면 됨
- `typeclass`, `trait` 등이 여기에 속함
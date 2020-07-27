---
title: 클로저에서 캡처된 변수는 어디 저장될까?
category:
  - Programming Language
toc: true
comments: true
date: 2020-07-27 20:43:21
tags:
  - Go
  - Python
  - Javascript
thumbnail: /2020/07/27/20200727-closure-and-free-vars/thumbnail.png
---

클로저(Closure)는 자신이 생성(선언)된 외부 환경을 기억(Capture)하는 함수이다. 이 글에서는 Python, Javascript, Go가 각각 어떻게 이를 구현하는지를 알아보았다. 

<!--More-->

클로저에 대한 자세한 설명은 [여기](https://poiemaweb.com/js-closure)[^1]를 참조하고 생략하고, 이 글에서는 클로저의 기능중 클로저 함수 외부에 선언된 변수를 접근하는 Capturing이 여러 언어에서 어떻게 구현되는지를 살펴보고자 한다.

# 여러 언어에서의 Closure

우선 클로저를 지원하려면 언어 차원에서 함수를 일급 객체(First-class object)로 취급을 해야한다. 

물론 함수가 일급 객체가 아닌 언어에서 클로저를 지원하는 경우도 있다. 그 예로 자바의 경우 Java8 부터 람다 함수가 도입되고 Closure를 사용할 수 있지만, 언어차원에서의 지원이 아니라 함수 하나당 그때그때 익명 클래스를 하나씩 생성하는 꼼수를 사용하며, final로 선언된 변수만 참조할수 있는 등 제약이 많아 완벽하다 할 수는 없다. 

그래서 주로 동적 언어에서 잘 구현되어 있는데, 구글에 클로저를 검색하면 대부분 자바스크립트 언어에 대한 글이 나온다.
아마 자바스크립트가 클로저를 제일 아름답고 직관적으로 쓸 수 있어서 그런것 같다. 특히 ES6부터 생긴 화살표 함수를 통한 currying[^2]은 아주 아름다운 클로저를 만들어 낸다.

아래 코드는 자바스크립트에서 커링을 통해 클로저를 만들어내는 예시이다. 

{% codeblock closure.js lang:javascript %}
const makeAdder = (add) => (to) => add + to;

const add2 = makeAdder(2);
const add5 = makeAdder(5);

console.log(add2(10)); // 12
console.log(add5(10)); // 15
{% endcodeblock %}

파이썬의 경우 언어차원에서 지원을 하긴 하는데... 파이썬으로 만들면 클로저가 뭔가 조잡해 보인다. 람다함수가 무조건 하나의 statement이어야 하는 제약때문에 한계도 많고, 무엇보다 **안이쁘다**. 위 코드를 파이썬으로 만들면 아래와 같다.

{% codeblock closure.py lang:python %}
make_adder = lambda add: lambda to: add + to

add2 = make_adder(2)
add5 = make_adder(5)

print(add2(10)) # 12
print(add5(10)) # 15
{% endcodeblock %}

그런데 의외로 컴파일 언어인 Go에서 함수가 일급 객체이며, 클로저를 지원한다고 한다.

각각의 언어는 어떻게 캡처(Capture)한 변수를 저장할까?

## Javascript
Javascript 관련한 내용은 워낙 좋은 자료가 많은데, Toast에서 게시된 [이 글](https://meetup.toast.com/posts/86)[^3]이 제일 잘 설명하고 있는 것 같으니, 꼭 한번 보길 권장한다.

자바스크립트는 함수가 선언되어 있는 *Lexical Scope*를 기억하며, 이 Scope 객체 안에 함수 내에 선언된 변수들이 저장된다. 

용어가 무서운데, *Dynamic Scope*와 비교하면 이해하기 쉽다.

- *Lexical Scope* : **소스코드상의 위치**를 기준으로 context 판단
- *Dynamic Scope* : **실행 상태**를 기준으로 context 판단

함수가 여러겹 겹쳐있을 수 있으니, Lexical Scope는 체인 형태로 함수가 내부적으로 가지고 있게 된다. 함수 내에서 변수에 접근하면, 제일 가까운 Scope부터 해당 변수가 선언되어 있는지 찾으며 마지막은 Global scope를 탐색한다.

<center>
  {% asset_img lexical_scope.png %}
  <small>
    Javascript에서 Lexical Scope를 통해 변수를 찾는 과정
  </small>
</center>

클로저에서 Capturing이 발생하면, 클로저가 속해있던 Lexical Scope를 클로저를 만들어 낸 함수가 종료된 후에도 체인에서 없애지 않고 유지한다. 

이때 클로저에서 사용되지 않는 변수는 가비지 컬렉터에 의해 수거되어 Scope 객체에서 사라진다.

## Python
파이썬에서는 함수도 객체(Object)이다. 파이썬은 인터프리터가 코드를 해석하면서 클로저에 Capture된다고 판단한 외부 변수는 함수 객체 내의 `__closure__` 라는 특별한 변수에 저장한다[^4].

객체에 어떤 속성들이 있는지 확인하는 매직 메소드인 `__dir__`을 통해 클로저에는 `__closure__`라는 변수가 존재함을 확인할 수 있다.

{% codeblock closure2.py lang:python %}
def outer_func(): 
  msg = 'Hi'
  print('outer_func local variables :', locals())

  def inner_func():
    inner_var = 'Bye'
    print(msg)
    print('inner_func local variables :', locals())

  return inner_func

my_func = outer_func()
my_func()
print(my_func.__dir__())
{% endcodeblock %}

위 코드를 실행하면 아래와 같은 결과가 나온다.

```
outer_func local variables : {'msg': 'HI'}
HI
inner_func local variables : {'inner_var': 'Bye', 'msg': 'HI'}
['__repr__', '__call__', '__get__', '__new__', '__closure__', '__doc__', '__globals__', '__module__', '__code__', '__defaults__', '__kwdefaults__', '__annotations__', '__dict__', '__name__', '__qualname__', '__hash__', '__str__', '__getattribute__', '__setattr__', '__delattr__', '__lt__', '__le__', '__eq__', '__ne__', '__gt__', '__ge__', '__init__', '__reduce_ex__', '__reduce__', '__subclasshook__', '__init_subclass__', '__format__', '__sizeof__', '__dir__', '__class__']
```

신기한점은 `locals`라는 로컬 변수를 출력하는 함수를 통해 확인해보면, 캡처된 변수도 일반 지역변수처럼 보인다는 것이다. 프로그래머의 편의성을 위한것으로 보이며, 실제로 저장은 `__closure__`에 된다.

## Go
Go는 컴파일 언어답게, 이 문제를 컴파일러가 해결한다. 컴파일 하면서 선언된 함수 밖에서도 사용된다고 판단한 변수는 Stack이 아니라 Heap에 저장하게 한다고 한다. 

신기한게 이 외에도 `new`를 통해 동적 할당한 메모리도 함수 내에서만 쓰이면 Stack에 할당하는 듯 힙 사용 최적화를 하는듯 하다[^5]. 

어떤 경우에 스택에 선언되고, 어떤 경우에 힙에 선언되는지는 [여기](https://jacking75.github.io/go_stackheap/)[^5] 잘 정리되어 있으니, 꼭 한번 보길 추천한다. 

# Summary

위 내용을 표로 정리하면 다음과 같다.

|언어|저장위치|방식|
|-----|-----|-----|
|Javascript|Lexical Scope 체인 유지|캡처되지 않는 변수는 스코프 객체에서 제거|
|Python|함수 Object 내 `__closure__` 필드|캡처되는 변수만 추가|
|Go|Heap 메모리|컴파일 타임에 저장위치 결정|

결국은 다 Heap에 저장한다.

# References

[^1]: [클로저(closure)의 개념](https://poiemaweb.com/js-closure)
[^2]: [Currying in Javascript](https://dev-momo.tistory.com/entry/Currying-in-Javascript)
[^3]: [자바스크립트의 스코프와 클로저](https://meetup.toast.com/posts/86)
[^4]: [파이썬 - 클로저 (Closure)](http://schoolofweb.net/blog/posts/파이썬-클로저-closure/)
[^5]: [golang - 스택과 힙에 대해](https://jacking75.github.io/go_stackheap/)
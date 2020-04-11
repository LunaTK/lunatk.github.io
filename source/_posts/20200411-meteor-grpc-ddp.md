---
title: Meteor에서 gRPC, DDP 사용하기
category:
  - Development
  - Web
toc: true
comments: true
date: 2020-04-11 13:39:33
tags: 
  - meteor
  - gRPC
thumbnail: /2020/04/11/20200411-meteor-grpc-ddp/meteor_grpc.png
---

Meteor는 Node.js 기반 Fullstack Web Framework이다. 이 글에서는 백엔드에서 프론트엔드 이외의 외부 클라이언트와 통신하는 기능을 추가하는 방법에 대해 이야기해 보고자 한다.

<!--More-->

Meteor는 기본적으로 웹소켓을 이용해 프론트엔드와 백엔드가 통신할 수 있는 기능을 제공한다. 이 기능을 통해 프론트엔드 개발을 할 때 MongoDB를 마지 서버에서 접근하듯이 손쉽게 다룰 수 있다.

하지만 이번 프로젝트 요구사항에는 프론트엔드, 백엔드가 아닌 제3의 클라이언트가 간단한 DB operation을 할 수 있는 기능이 필요하였다. 

간단히 소개하자면, 실행이 오랫동안 걸리는 반복 수행 작업을 쉘 스크립트로 돌릴 때, 어느정도 진행되었는지를 Meteor 서버에게 알려주고 웹 프론트엔드로 모니터링 할 수 있는 서비스이다.

여러 조사 끝에 정리한 후보는 다음과 같다.

1. RESTful API
2. [gRPC](https://grpc.io)
3. [Distributed Data Protocol (DDP)](https://blog.meteor.com/introducing-ddp-6b40c6aff27d)

결론부터 말하면, 최종적으로 **gRPC**를 사용하기로 결정하였다.

아래 부터는 각각이 무엇이고, 어떤 장단점이 있는지 설명한다.

## RESTful API

쉽게말해 HTTP GET, POST 요청등을 이용해 클라이언트와 서버가 소통하는 방법이다. 가장 흔하게 사용되는 방법이라 먼저 떠올랐지만, 다음과 같은 이유로 선정하지 않았다.

1. Meteor에서 지향하는 방법이 아니다.
   * Meteor는 Fullstack 프레임 워크라, 애초에 백엔드와 프론트엔드의 통합이 잘 이루어져 있는데, 특이하게 REST API가 아니라 웹소켓을 사용한다.
   * 따로 REST API서버를 Meteor에 만들 순 있지만, 복잡해 보여 시도하지 않았다.
2. 오버헤드가 클것같다.
   * 실제 측정을 해본것은 아니지만, 아무래도 HTTP 프로토콜 레이어가 들어가니 다른 선택지에 비해 오버헤드가 있을것 같았다.

## gRPC

gRPC는 구글에서 만들고 오픈 소스로 운영 중인 RPC(Remote Procedure Call) 프레임워크이다. 원래 RPC가 성능이 안좋던것을 개선해 지금은 REST API보다도 성능적으로 우세한것 같다.
다음과 같은 이유로 선정하였다.

1. 빠르다
2. [grpcurl](https://github.com/fullstorydev/grpcurl) 이라는 커맨드라인용 툴이 존재한다. (심지어 go로 만들어서 빠르다!)


### 사용법

일반적으로 Node.js에서 gRPC 서버 여는 방법을 따라하였다. ([Node.js gRPC Services in Style](https://medium.com/trabe/node-js-grpc-services-in-style-222389be991a))

다만 gRPC 서버를 Meteor에서 사용하려만 한가지 제약이 있는데, gRPC 서버에서 Meteor의 DB 함수를 사용하려면 `Meteor.bindEnvironment` 라는 함수로 래핑 해주어야 한다. 그렇게 복잡하지는 않고 코드 한두줄이 추가되는 정도이다. 자세한 내용은 [Meteor code must always run within a Fiber](https://forums.meteor.com/t/meteor-code-must-always-run-within-a-fiber-try-wrapping-callbacks-that-you-pass-to-non-meteor-libraries-with-meteor-bindenvironmen/40099) 를 참고하였다.

{% codeblock rollup.config.js lang:javascript %}
const bound = Meteor.bindEnvironment((callback) => {callback();});

const newSession = (call, callback) => {
  console.log(call.request);
  
  bound(() => {
    Tasks.insert({
      title: call.request.title,
      createdAt: new Date(),
    }, (err, res) => {
      console.log(err, res);
    });
    
    callback(null, { /* response data here */ });
  });
};
{% endcodeblock %}

## Distributed Data Protocol (DDP)

공식 소개글 : [Introducing DDP](https://blog.meteor.com/introducing-ddp-6b40c6aff27d)

Meteor에 gRPC 올리는 방법을 찾아보다 알아낸 기능인데, 아마 이게 뭔지 아는사람은 거의 없을것 같다. Meteor 개발자팀이 만든 프로토콜로, **Meteor의 프론트엔드와 백엔드가 웹소켓으로 통신할때 사용하는 프로토콜이다**. 

재밌는 점은 WebSocket 이라는것 자체가 웹 브라우저 밖에서도 쓸 수 있기 때문에, 그점을 이용해 **Meteor와 아무상관없는 제3의 클라이언트에서도 미티어 백엔드와 DDP를 통해 통신할수 있다!!**

사람들도 이미 DDP를 Remote Procedure Call(RPC)의 한 종류로 취급하고, 실제로 그렇게 쓰는 경우도 있어 보였다. 사실이라면 굳이 또다른 RPC인 gRPC를 도입할 필요가 없어보였다.

DDP는 사람들이 다양한 언어에서 라이브러리로 구현해 놓았는데, 4년 전이 마지막 업데이트인게 대부분이었다... 

* [Meteor 와 다른 언어/플랫폼과의 통신 수단인 DDP](http://spectrumdig.blogspot.com/2013/06/meteor-ddp.html)
* [플랫폼 별 DDP Clients](http://www.meteorpedia.com/read/DDP_Clients)

그래서 [Node.js로 구현한 DDP 클라이언트](https://gist.github.com/LunaTK/412b1b4071ab3d439285a789862a40cf)와 gRPC와 성능 비교를 해 보았다. MongoDB document를 10개 만드는데 걸리는 시간을 비교하였다.

```shell
1.69s user 0.41s system 42% cpu 4.954 total  # DDP  (node.js)
0.10s user 0.07s system 82% cpu 0.204 total  # gRPC (go)
```

결과는 DDP의 처참한 패배였다. DDP가 Node.js에서 돌아가긴 했지만, 감안하더라도 25배의 성능 차이는 용납할 수 없었다.

## Discussion

Client 입장에서의 실행 시간을 기준으로만 gRPC를 선택하였지만, 사실 Meteor 백엔드에 gRPC를 위한 서버가 하나 더 생기는 것이기 때문에 이 방법이 최선이라는 보장은 없다.
또한 DDP 클라이언트는 node.js로 실험했기 때문에, 매번 스크립트 파싱, DDP 라이브러리 로드 등등 오버헤드가 커서 성능 차이가 더 심각한 것일수도 있다.

나는 어차피 요청 클라이언트를 쉘 스크립트 안에서 매번 다시 실행할것이기 때문에 위 실험 환경이 유의미 하지만, 지속적으로 돌아가는 클라이언트나 node.js가 아닌 다른 환경에서 실행하면 DDP의 성능이 gRPC에 비해 그렇게 나쁘지 않을수도 있다 생각한다.

## Conclusion

구글이 만든 gRPC를 구글이 만든 go로 구현한게 최고인것 같다. 구글 만세 😊

## References
* [npm.js/ddp](https://www.npmjs.com/package/ddp)
* [Node.js gRPC Services in Style](https://medium.com/trabe/node-js-grpc-services-in-style-222389be991a)
* [Meteor code must always run within a Fiber](https://forums.meteor.com/t/meteor-code-must-always-run-within-a-fiber-try-wrapping-callbacks-that-you-pass-to-non-meteor-libraries-with-meteor-bindenvironmen/40099)
* [grpcurl](https://github.com/fullstorydev/grpcurl)
* [Introducing DDP](https://blog.meteor.com/introducing-ddp-6b40c6aff27d)
* [Meteor 와 다른 언어/플랫폼과의 통신 수단인 DDP](http://spectrumdig.blogspot.com/2013/06/meteor-ddp.html)
* [플랫폼 별 DDP Clients](http://www.meteorpedia.com/read/DDP_Clients)
---
title: Node.js 에서 request 한글 깨짐 문제
date: 2019-10-25 17:28:11
tags: node.js
category:
    - Development
    - Node.js
thumbnail: /2019/10/25/20191025-nodejs-request-encoding/Untitled-26832150-96d7-4be8-840b-5729f29cb4b7.png
toc: true
widgets:
  - 
    type: toc
    position: right
  -
    type: recent_posts
    position: right
  -
    type: profile
    position: left
    author: 문태근
    author_title: Graduate Student @ SKKU
    location: Suwon, South Korea
    avatar: /res/profile.jpg
    avatar_rounded: true
    follow_link: 'https://github.com/LunaTK'
    social_links:
        Github:
            icon: fab fa-github
            url: 'https://github.com/LunaTK'
        LinkdIn:
            icon: fab fa-linkedin
            url: 'https://www.linkedin.com/in/taegeun-moon-9bb2bb133/'
  -
    type: category
    position: left
---

Node.js 에서 `request` 모듈을 사용하여 학교 홈페이지를 크롤링 하는 도중 한글이 깨지는 문제가 발생하였다.

<!--More-->

![](/2019/10/25/20191025-nodejs-request-encoding/Untitled-d219658d-d514-4845-9409-4415100cc5b5.png)

한글 깨짐 현상

한국인으로서 담담하게 받아들일 수 있는 현상이다.

확인해보니, Node.js는 `UTF-8`을 기본 인코딩으로 사용하는데 학교 홈페이지는 `EUC-KR`을 사용해서 글자가 깨지는 것 같았다.

![](/2019/10/25/20191025-nodejs-request-encoding/Untitled-a33c1e21-f337-49fa-88e8-5c9a295c8edc.png)

웹페이지 인코딩 확인

그래서 [EUC-KR 디코딩](https://stories.pe.kr/215)을 하게 해 주었는데, 여전히 표시가 잘 안되었다.

![](/2019/10/25/20191025-nodejs-request-encoding/Untitled-26832150-96d7-4be8-840b-5729f29cb4b7.png)

EUC-KR 처리 후 한글 깨짐 현상

# 원인

그런데 이 `占쏙옙` 이라는 글자, 낯설지가 않은 사람들이 많을것이다...

문제를 해결하기에 앞서 왜 한글이 깨지면 저렇게 `占쏙옙` 이 미친듯이 반복되는지 궁금해 검색 해 보았다.

![](/2019/10/25/20191025-nodejs-request-encoding/Untitled-2cb0ef53-9f81-4a33-8be9-a59c61bd3a30.png)

占쏙옙 의 정체

출처 : [https://namu.wiki/w/占쏙옙](https://namu.wiki/w/%E5%8D%A0%EC%8F%99%EC%98%99)

정리하자면 다음과 같다.

1. `EUC-KR`로 표현된 데이터 `UTF-8`로 저장한다.
2. 이때 `UTF-8`로 표현될 수 없는 문자는 모두 "표현 불가" 문자, 즉 � 로 치환된다.
3. 이를 다시 `EUC-KR`로 취급하여 `UTF-8`로 변환하려 하면, `占쏙옙` 이 된다.

이때문에 `EUC-KR`을 잘못 다루면 `占쏙옙` 파티를 볼 수 있게 되는 것이었다.

내가 겪은 문제는 아마 request 모듈에서 `EUC-KR`을 제대로 처리하지 않고 바로 `UTF-8`로 저장을 해서, 이미 � 으로 다 치환이 된 상태에서 `EUC-KR` 디코딩을 시도한게 원인인 것 같았다.

# 해결방법

해결방법은 간단하다. request 모듈이 문자열 인코딩 처리를 안하게 해주면 된다.

```javascript
    request({
    		url:"http://icampus.ac.kr" // 원하는 url값을 입력
    		,encoding: null //해당 값을 null로 해주어야 제대로 iconv가 제대로 decode 해준다.
    	}
    	,(err, res, body) => {
    		// 생략
    	}
    })
```
출처 : [http://b1ix.net/322](http://b1ix.net/322)

![](/2019/10/25/20191025-nodejs-request-encoding/Untitled-ffb78e78-64e5-4140-b438-bfa4bba91389.png)

야! 잘된다
---
title: webpack:// 경로는 무엇일까?
date: 2019-10-21 19:37:29
tags: webpack
category:
    - Programming
    - Web
thumbnail: /2019/10/21/20191021-what-is-webpack-path/img2.png
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

학교 아이캠퍼스 소스를 보다, 웹사이트 리소스에서 신기한걸 발견했다.

<!-- more -->

![아이캠퍼스 원본 소스](/2019/10/21/20191021-what-is-webpack-path/img1.png)

아이캠퍼스가 리액트로 만들어졌는데, 개발 모드로 열려있는지 소스가 다 보이는 것이었다.
(아마 `webpack-dev-server`로 열어 놓은 것 같다)

평소 리액트 개발을 할 때 개발자 도구에서 원본 파일로 디버깅이 되는걸 당연시 했었는데, 사용자 입장에서 원본 파일이 보이니 갑자기 이상한 점이 눈에 띄었다.

![webpack://](/2019/10/21/20191021-what-is-webpack-path/img2.png)

# 파일 출처가 `webpack://` 인데, 이게 뭐지? 

다른 파일들은 모두 URL 주소가 출처로 되어있었다.

그런데 저 리액트 앱 원본 코드들은(심지어 node_modules 폴더도 있다) 출처가 `webpack://`으로 되어있었다.

도대체 어떻게 가져왔길래 저렇게 표시가 되는지 원리가 궁금해 찾아보게 되었다.


# 여러가지 시도

## 1. 일반 js 파일처럼 HTTP 요청을 통해 가져 올 것이다

개발자 도구에서 일반 js 파일과 똑같이 보이니, 가져오는 방법도 일반 js 파일과 똑같을 것이란 생각이 먼저 들었다.

웹 사이트 내에서 불러오는 모든 리소스는 개발자 도구의 Network 탭에서 확인할 수 있다.

![Network](/2019/10/21/20191021-what-is-webpack-path/img3.png)

페이지 로딩이 다 끝난 후 확인 해 보니, `webpack://` 디렉토리에서 확인할 수 있었던 수많은 js 파일이 **단 하나도** 없었다.

그래서 일단 `webpack://`이 뭔지 구글링을 해 보기로 하였다.

![what is webpack://](/2019/10/21/20191021-what-is-webpack-path/img4.png)

[https://github.com/angular/angular-cli/issues/11058](/2019/10/21/20191021-what-is-webpack-path/https://github.com/angular/angular-cli/issues/11058)

정확히 나와 똑같은 생각을 한 질문이 있었다.

해당 질문에서 얻은 내용을 정리해 보면 다음과 같다.
- webpack source map 에 쓰이는 커스텀 프로토콜이다 (~~나중에 알았지만 커스텀 프로토콜이 아니었다... 이 답변때문에 엄청 헷갈렸다~~)![reply](/2019/10/21/20191021-what-is-webpack-path/img5.png)
- webpack-dev-server 와 관련이 있다


## 2. Source Map에 정보가 있을것이다

위 답변을 통해 Source Map 과 큰 관련이 있다는 사실을 알 수 있었다.

그래서 우선 Source Map 이 무엇인지 검색해 보았다.

> 클라이언트측 코드를 결합하거나 최소화하거나 컴파일한 후에도 읽을 수 있고 디버그할 수 있게 합니다. 
> 소스 맵을 사용하여 소스 코드를 컴파일된 코드에 매핑합니다

정리하자면, `webpack과` 같이 원본 코드를 압축, 변형하는 경우 변형된 코드를 원본 소스코드를 보며 디버깅 할 수 있도록 변환된 파일을 원본 파일에 맵핑해 주는 파일이라고 한다.

맵핑할 js 파일 하단에 `//# sourceMappingURL=http://example.com/sourcemap.map` 과 같이 맵핑 파일의 URL을 넣어주면 자동으로 디버깅시 원본 소스코드를 보여준다고 한다.

![source map](/2019/10/21/20191021-what-is-webpack-path/img6.png)

위 사진처럼, Source Map detected 란 문구와 함께 맨 아랫줄에 주석으로 source map의 URL이 있는걸 확인할 수 있었다.

해당 URL로 들어가보니, 정신이 혼미해지는 source map 파일을 확인할 수 있었다.

![source map](/2019/10/21/20191021-what-is-webpack-path/img7.png)

현재 사용되는 소스맵은 스펙은 V3 버전인데, 다음과 같은 구조를 가진다고 한다.

```javascript
{
    version : 3,
    file: "out.js", // source map 을 해줄 컴파일 된 js 파일
    sourceRoot : "", // 원본 소스파일들 앞에 붙여줄 공통 경로
    sources: ["foo.js", "bar.js"], //  output file을 만드는데 쓰인 js 파일들
    names: ["src", "maps", "are", "fun"], // javascript 안에 있는 함수, 변수 등의 이름
    mappings: "AAgBC,SAAQ,CAAEA" // base64로 인코딩 된 실제 맵핑 테이블
}
```

위 스펙에 따르면, `webpack:///./public/javascripts/dummyl18nResource.js?1e80`같은게 실제 파일이 있는 URL이어야 한다.

근데 경로에 `.` 이 들어가 있질 않나, `webpack://` 같은걸로 시작하는 등 일반적인 URL 모양이 아니었다.

그래도 스펙이 그렇다 하니 일단 해당 URL로 접근 해 보았다.

![not found](/2019/10/21/20191021-what-is-webpack-path/img8.png)

역시나 안됬지만, 이건 뭐 되는게 더 이상했을 것 같다.


## 3. webpack 프로토콜이 있을것이다

우리가 아는 일반적인 URL은 `프로토콜://도메인:포트`로 생겼다. 일단 `webpack://`의 생김새가 URL 처럼 생겼고, 실제로 그렇게 쓰여야 하니, 혹시 사이트 내에서 webpack이라는 커스텀 프로토콜을 정의한게 아닌가 생각이 들었다.

요새 웹은 정말 희안한 기능들이 많아서(ex. `ws://`), 충분히 가능할거라 생각하였다.

![custom protocol](/2019/10/21/20191021-what-is-webpack-path/img9.png)

그리고 진짜 커스텀 프로토콜을 정의하는 기능이 있긴 있었다!!

하지만 깃허브 `webpack`, `webpack-dev-server` 저장소에서 검색을 해 봐도, 딱히 커스텀 프로토콜을 정의하는 부분을 찾을 수 없었다.

![webpack repo](/2019/10/21/20191021-what-is-webpack-path/img10.png)


## 4. Source Map을 이용해 원본 소스코드를 다운해 보자

개발자 도구에서 다른 사이트 Source Map이 보인다면 원본 소스코드를 다운할수 도 있다는건데, 이를 구현해놓은 코드가 없나 찾아보다 다음 포스팅을 발견했다.

[Extracting Javascript From SourceMaps](https://pulsesecurity.co.nz/articles/javascript-from-sourcemaps)

이분이 딱 내가 원하는 코드를 만들어서 github에 공개까지 해 놓으셨다 [(denandz/sourcemapper)](https://github.com/denandz/sourcemapper)


그런데 Source Map 스펙에 따르면 분명 원본 파일을 다운받으려면 `sources`에 포함된 파일 하나당 한번의 HTTP 요청을 해야하는데, 위 소스코드에는 source map 파일을 받아오는 최초 한번의 HTTP 요청을 제외하고는 HTTP 요청이 없었다.

도대체 그럼 어떻게 원본 파일을 가져오는지 소스코드를 한번 다 읽어 보았다.
```go
type sourceMap struct {
    Version        int      `json:"version"`
    Sources        []string `json:"sources"`
    SourcesContent []string `json:"sourcesContent"`
}
```

위 구조체가 `sourceMap`을 저장하는 구조체인데, `sourcesContent` 라는 처음보는 속성이 있었다.

구글링 해 보니, [Source Map Revision 3 Proposal](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?pli=1)을 참고하라는 [StackOverflow 게시글](https://stackoverflow.com/questions/19802462/do-source-maps-include-the-source-text)을 하나 찾을 수 있었다.

![sourcesContent Spec](/2019/10/21/20191021-what-is-webpack-path/img11.png)

2019년 2월 19일에 추가된 스펙이었다. (~~그럼 그 이전에는 어떻게 소스맵을 제공했을까?~~)

하지만 스펙문서 설명이 너무 부실해서 저 내용만 봐서는 저게 정확히 무슨 기능인지 알기 어려웠다. 링크를 찾은 StackOverflow 게시글에 따르면,

> An **optional** list of **source content**, useful when the "source" can't be hosted

아... 그러니까 *선택적*으로 Source Map 안에 Source Content, 즉 원본 소스코드를 넣을 수 있다는 말이었다...

![Source Map: sourcesContent](/2019/10/21/20191021-what-is-webpack-path/img12.png)

![Source Map File Size](/2019/10/21/20191021-what-is-webpack-path/img13.png)

원본 코드를 싹다 가지고 있어서인지, 저 소스맵 파일 하나의 용량이 20 메가바이트였다...

# 결론

`webpack://` 은 프로토콜이나 URL이 아니고, 그냥 웹팩이 임의로 파일 이름에 붙인 접두사 같은 것 이었다.

그리고 `sources에` 있는 파일 이름과 `sourcesContent` 에 있는 소스코드 내용이 순서대로 매칭되고, 실제 파일 내용을 `sourcesContent에서` 가져오더라도 `sources` 에 있는 파일 URL로부터 가져온것 처럼 개발자 도구에 표시가 되는 것이었다.

글로 정리하니 얼마 안되는 양이지만, 이걸 알아내는데 하루가 걸렸다.

소스맵이 W3C 표준은 아니라고 하는데, 왜 아닌지 알 것 같은 경험이었다😅
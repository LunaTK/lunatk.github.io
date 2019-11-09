---
title: Rollup & Svelte 에서 PurgeCSS 사용하기
category:
  - Development
  - Web
toc: true
comments: true
date: 2019-11-09 21:27:16
tags: 
  - rollup
  - css
  - svelte
thumbnail: /2019/11/09/20191109-purgecss-with-rollup-and-svelte/thumbnail.png
---

외주 프로젝트용 Web Component 개발에 UI 라이브러리를 사용하니, 빌드된 컴포넌트 사이즈가 너무 큰 문제가 발생해 PurgeCSS를 이용해 최적화를 해 보기로 하였다.

<!--More-->

개발 환경은 다음과 같았다.

- **Front-End Framework** : [Svelte](https://svelte.dev/blog/svelte-3-rethinking-reactivity)
- **Module Bundler** : [Rollup.js](https://rollupjs.org)
- **UI Library** : [Bulma](https://bulma.io)

디자인을 전달받기 전이라 우선 UI 라이브러리를 사용하여 개발을 진행하였다.
*Rollup*은 자바스크립트 번들러이기 때문에, 기본적으로는 진짜 자바스크립트 파일밖에 번들링을 못한다.
따라서 `.svelte`, `.css` 파일을 번들링 하기 위해서는 적절한 플러그인을 적용해 주어야 한다.

### 1. Rollup Plugin 추가

{% codeblock rollup.config.js lang:javascript %}
  import svelte from 'rollup-plugin-svelte';
  import postcss from 'rollup-plugin-postcss';
  // 몇가지 Plugin을 더 사용했지만, 생략

  export default {
    input: 'src/index.js',
    output: [
      { file: `public/${pkg.module}`, 'format': 'es' },
      { file: `public/${pkg.main}`, 'format': 'umd', name }
    ],
    plugins: [
      svelte(),
      postcss()
    ]
  }
{% endcodeblock %}

`rollup-plugin-svelte` 를 통해 .svelte 파일을 로드하고, `rollup-plugin-postcss` 를 통해 .css 파일을 로드할 수 있게 설정하였다.

이제 svelte 파일의 `script` 부분에서 *Bulma*의 css 파일를 import 할 수 있다.

### 2. svelte 컴포넌트에서 `css` 파일 import 하기

{% codeblock component.svelte lang:html %}
  <script>
    import 'bulma/css/bulma.css';
    //생략
  </script>
  // Svelte HTML 부분
{% endcodeblock %}

이렇게 해서 만들어진 컴포넌트는 아래처럼 생겼다.

<center>
  {% asset_img img1.png %}
  <small>
    Svelte와 Bulma를 이용한 스마트폰 검색 Web Component
  </small>
</center>

그런데 문제가 생겼다. css 파일을 import 하면 이를 string으로 바꿔서 통째로 번들링 해버리기 때문에, 쓰지 않는 속성까지 다 포함되어 용량이 너무 컸던 것이었다.

<center>
  {% asset_img img2.png %}
  <small>
    Web Component 치고는 무거운 결과물
  </small>
</center>

요즘 웹사이트 하나 *webpack*으로 빌드하면 기본이 몇 MB 이니, 260kb 정도면 작다 생각할 수 있지만, 이건 웹 컴포넌트로 쓸꺼라 최대한 줄일 필요가 있었다.

그래서 사용하지 않는 css 속성을 제거해준다는 [PurgeCSS](https://www.purgecss.com)라는 툴을 사용해 보기로 했다. (이름이 ㅎㄷㄷ 하다. CSS 숙청...😦)

## PurgeCSS

우선 PostCSS는 Webpack, Gulp, Grunt, Rollup 등 그냥 아무데나 다 갖다 쓸 수있고, 심지어 그냥 Standalone 으로도 쓸 수 있는 툴이다.

나는 빌드환경을 Rollup으로 잡아놓았기 때문에, 공식 사이트에 나와있는 Rollup 가이드를 확인해 보았다.

{% linkPreview https://www.purgecss.com/with-rollup _blank nofollow %}

<center>
  {% asset_img img3.png %}
  <small>
    PurgeCSS With Rollup
  </small>
</center>

이게 끝이다. 그냥 저 페이지에 저거밖에 없었다...

정보가 너무 빈약해서 Configuration을 살펴보았다.

{% codeblock Options lang:javascript %}
{
  content: Array<string | RawContent>,
  css: Array<string | RawContent>,
  extractors?: Array<ExtractorsObj>,
  whitelist?: Array<string>,
  whitelistPatterns?: Array<RegExp>,
  whitelistPatternsChildren?: Array<RegExp>,
  keyframes?: boolean,
  fontFace?: boolean,
  rejected?: boolean
}
{% endcodeblock %}

중요한 부분만 보자면 다음과 같았다.

- `content` : css 사용 여부를 확인할 파일 (ex. html, js)
- `css` : 숙청할 css 파일
- `extractors` : content에서 css selector를 추출해 주는 친구
- `whitelist` : 사용되지 않더라도 숙청하지 않을 css selector 목록

**With Rollup** 가이드를 보면, `content`로 html 파일을 사용하고 있다. `.svelte` 파일도 사용 가능한지 공식 문서를 확인해 보았다.

> PurgeCSS provides a default extractor that is working with **all types of files** 
> but can be limited and not fit exactly the type of files or css framework that you are using.

확인해 보니 Default Extractor로도 모든 파일이 다 된다고 한다~~(우와 개쩐다)~~. 하지만 좀더 정확히 하려면 Custom Extractor를 만들어 쓰라고 한다. 

[langbamit/purgecss-from-html](https://github.com/langbamit/purgecss-from-svelte) 이라는 누가 만들어 놓은 Svelte용 Extractor가 있었다. 결론부터 말하면 이거 안써도 똑같이 잘 되서 처음엔 썼지만 마지막 단계에서는 그냥 안쓰기로 하였다.

이제 PurgeCSS를 Rollup에 적용해 보았다.

{% codeblock rollup.config.js lang:javascript %}
  import svelte from 'rollup-plugin-svelte';
  import postcss from 'rollup-plugin-postcss';
  import purgecss from 'rollup-plugin-purgecss';
  import PurgeSvelte from "purgecss-from-svelte";
  // 몇가지 Plugin을 더 사용했지만, 생략

  export default {
    input: 'src/index.js',
    output: [
      { file: `public/${pkg.module}`, 'format': 'es' },
      { file: `public/${pkg.main}`, 'format': 'umd', name }
    ],
    plugins: [
      svelte(),
      postcss(),
      purgecss({
        content: ["./src/**/*.svelte"],
        extractors: [
          {
            extractor: PurgeSvelte,
            extensions: ["svelte"]
          }
        ]
      })
    ]
  }
{% endcodeblock %}

여기서 한방에 되면 재미가 없으니, 바로 오류를 뿜어주는 Rollup...

<center>
  {% asset_img img4.png %}
</center>

CssSyntaxError 라는 키워드로 구글링을 하다 다음 StackOverflow 게시글을 발견했다

{% linkPreview https://github.com/webpack-contrib/mini-css-extract-plugin/issues/358 _blank nofollow %}

> you have **multiple loader** for css please check your configuration

css loader를 여러개 설정해서 생기는 에러라고 한다. 그래서 `postcss` 플러그인을 빼고 `purgecss`만 남겨보았는데, 또 에러가 발생하면서 안되었다.

좀더 구글링을 하던중, 어느 외국회사 **GitLab**이 떴다...! (다음에 **GitLab** 쓸일이 있으면 조심해야겠다...) 운이 좋게도 그곳에서 해답을 얻을 수 있었다.

**PurgeCSS**를 **Rollup** Plugin이 아니라 **PostCSS의 Plugin으로** 만들어 놓은게 있었다 ([FullHuman/postcss-purgecss](https://github.com/FullHuman/postcss-purgecss)). 

성공한 Rollup 설정은 다음과 같다.

{% codeblock rollup.config.js lang:javascript %}
  import svelte from 'rollup-plugin-svelte';
  import postcss from 'rollup-plugin-postcss';
  import Purgecss from "@fullhuman/postcss-purgecss"
  import PurgeSvelte from "purgecss-from-svelte";
  // 몇가지 Plugin을 더 사용했지만, 생략

  const purgeCss = Purgecss({
    content: ["./src/**/*.svelte"],
    extractors: [
      {
        extractor: PurgeSvelte,
        extensions: ["svelte"]
      }
    ]
  });

  export default {
    input: 'src/index.js',
    output: [
      { file: `public/${pkg.module}`, 'format': 'es' },
      { file: `public/${pkg.main}`, 'format': 'umd', name }
    ],
    plugins: [
      svelte(),
      postcss({
        plugins: [
          purgeCss
        ]
      }),
    ]
  }
{% endcodeblock %}

<center>
  {% asset_img img5.png %}
  <small>
    PurgeCSS 적용한 결과물 File Size
  </small>
</center>

결과는 감동적이었다. 264kb 에서 46kb로 무려 **83% 감소**하였다...😭

<center>
  {% asset_img img6.png %}
  <small>
    PurgeCSS 적용한 결과물
  </small>
</center>

그런데 위 사진처럼 폰트가 좀 이상하게 나왔다. 확인해 보니 아래 selector가 날라갔다. 아마 **Bulma** 라이브러리가 폰트를 모든 Element에 적용하기 위해 `body` 태그에 적용했는데, `.svelte` 파일 내에서 `body`태그를 직접적으로 사용하는 부분이 없어서 안쓴다고 판단되 삭제된것같다.

<center>
  {% asset_img img8.png %}
  <small>
    `body` selector로 폰트가 적용되어있다
  </small>
</center>

<center>
  {% asset_img img7.png %}
  <small>
    `body` selector가 삭제되었다
  </small>
</center>

그래서 **PurgeCSS** 의 `whitelist`에 `body`를 추가해 주었다.

# Solution

{% codeblock rollup.config.js lang:javascript %}
  import svelte from 'rollup-plugin-svelte';
  import postcss from 'rollup-plugin-postcss';
  import Purgecss from "@fullhuman/postcss-purgecss"
  // import PurgeSvelte from "purgecss-from-svelte";
  // purgecss-from-svelte 는 안써도 문제 없다
  // 몇가지 Plugin을 더 사용했지만, 생략

  const purgeCss = Purgecss({
    content: ["./src/**/*.svelte"],
	  whitelist: ['body'] // whitelist 추가
  });

  export default {
    input: 'src/index.js',
    output: [
      { file: `public/${pkg.module}`, 'format': 'es' },
      { file: `public/${pkg.main}`, 'format': 'umd', name }
    ],
    plugins: [
      svelte(),
      postcss({
        plugins: [
          purgeCss
        ]
      }),
    ]
  }
{% endcodeblock %}

<center>
  {% asset_img img1.png %}
</center>

폰트가 성공적으로 잘 적용되었다. 빌드된 모듈 용량은 거의 그대로였다.
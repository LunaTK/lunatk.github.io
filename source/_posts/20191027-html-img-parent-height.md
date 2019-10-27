---
title: img 태그와 부모 태그의 높이가 왜 다를까?
date: 2019-10-27 23:28:45
tags: css
category:
    - Development
    - Web
thumbnail: /2019/10/27/20191027-html-img-parent-height/img1.png
---

블로그에 URL 미리보기 기능을 만들던 도중 `img` 태그와 부모 태그의 높이가 다른 버그가 발생하였다.

<!--MORE-->

<center>
  {% asset_img img1.png %}
  <small>
  빨간 부분 만큼 높이 차이가 발생
  </small>
</center>
<br>

해당 HTML과 CSS는 대략 다음과 같다

{% codeblock html lang:html %}
<div>
  <div class="og-image">
    <a>
      <img src="~~">
    </a>
  </div>

  <div>
    <!-- 웹사이트 요약 텍스트가 들어가는 부분 -->
  </div>
<div>
{% endcodeblock %}

{% codeblock css lang:css %}
.og-image {
  background: red;
}

img {
  height: 120px;
}
{% endcodeblock %}

하지만 이상하게도 `og-image` 와 `img` 의 높이가 일치하지 않았다.

{% jsfiddle j06Lactw html,css,result %}
위는 비슷하게 구현해 본 코드이다.
코드만 보면 img와 og-image의 높이가 같을 것 같지만, 실제 result를 보면 높이가 다르다.

<center>
  {% asset_img img2.png %}
  <small>
    img 태그의 사이즈 (596.33 x 200) 
  </small>
</center>
<center>
  {% asset_img img3.png %}
  <small>
    div.og-image 태그의 사이즈 (596.33 x 206) 
  </small>
</center>
<br>

`img`를 `a`로 감싸서 그런가 `a` 태그를 제거해 보았지만, 여전히 오차가 있었다.
혹시나 해서 `og-image` 의 높이를 직접 120px로 고정해 보니 `img`와 높이가 일치하긴 했다. 
하지만 responsive design을 적용 할 계획이라 높이를 고정시키지 않고 해결 할 필요가 있었다.


개발자 도구를 켜놓고 `height`, `flex`, `max-height` 등등 의심이 가는 원인을 다 건드려 보다, `display`옵션을 바꿔보던 중 버그가 해결되었다.

어떻게 해결 되었나 보니, `img` 태그의 `display` 값을 `block` 으로 바꾸니 해결되었던 것이었다.

{% jsfiddle j06Lactw/3 html,css,result %}
<center>
  <small>
    문제를 해결한 JSFiddle
  </small>
</center>
<br>

# 마치며


`img` 태그의 `display` 속성 기본 값은 `inline-block` 이라고 한다 (그래서 `width`, `height` 값을 줄 수 있다고 한다).

**근데 웃긴건 그렇다고 해서 다른 `inline-block` 속성을 준 태그에서 저런 문제가 발생하지는 않는다.**
(아마 img 태그는 좀 특별한 케이스의 inline-block인것 같다.)

`img` 태그에서만 저런 문제가 발생하는것 같은데, 섬세한 관심이 필요한 친구인 것 같다.

기회가 되면 아래 링크의 `img` 태그 스펙을 자세히 살펴봐야 할것 같다.

{% linkPreview https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img _blank nofollow %}

사실 URL 미리보기가 잘 작동하는거 자랑할려고 괜히 한번 링크를 걸어보았다. 😏👍🏻
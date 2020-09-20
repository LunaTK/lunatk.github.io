---
title: 모던 CLI 툴 추천 리스트
category:
  - Development
  - Tools
toc: true
comments: true
date: 2020-09-10 22:40:54
tags:
  - cli
thumbnail:
---
리눅스나 macOS에서 터미널을 사용하다 보면 CLI 명령어를 자주 사용하게 된다. 대부분의 강의나 포스팅에서는 초창기부터 있던 전통적인 CLI 툴 위주로 알려주는데, 요즘에는 더 이쁘고 빠르고 편한 툴들이 많이 나와서 괜찮은 것들을 한번 정리해 보았다.

<!--More-->

아래 나열된 툴들 대부분은 기존 CLI 명령어를 대체, 개선하고자 만들어졌다.

그래서 제공하는 기능과 목표는 비슷하지만, 조금씩 다른 명령어 인터페이스를 제공한다.

또한 모두 공통적으로 colorized output을 제공하여 시각적 만족도가 높다.

# exa
- [https://github.com/ogham/exa](https://github.com/ogham/exa)
- 현대판 `ls`
- 컬러 하이라이트, tree 모드 등을 제공한다
- 나는 `exa`를 `ls`로 alias해서 사용하는데, 아주 만족스럽다 😎

# ripgrep
 [https://github.com/BurntSushi/ripgrep](https://github.com/BurntSushi/ripgrep)
- 현대판 `grep`
- `grep`보다 훨씬 빠르다고 하다
- regex 친화적인 검색 (Rust 내장 regex 엔진 사용)

# fd
- [https://github.com/sharkdp/fd](https://github.com/sharkdp/fd)
- 현대판 `find`
- `find`보다 더 빠르다고 하다

# jq
- [https://github.com/stedolan/jq](https://github.com/stedolan/jq)
- 커맨드라인 JSON 프로세싱 툴
- JSON path를 통해 입력된 JSON에서 특정 필드를 추출하는 등의 작업 가능
- `curl`과 pipe하여 REST api 리스폰스를 파싱하기 좋음
- 속도와 표현력이 좋다

# fx
- [https://github.com/antonmedv/fx](https://github.com/antonmedv/fx)
- `jq`와 비슷한 JSON 프로세싱 툴
- `jq`는 C언어로 작성되었는데, `fx`는 node.js라서 좀 더 느리다
  - 대신 nodejs 3rd-party 라이브러리를 쓸 수 있어 `jq`보다 확장성이 좋다
  - 기본 기능만 봐서는 `jq`가 더 좋아보임
- interactive mode, streaming 등 지원

# httpie
- [https://github.com/httpie/httpie](https://github.com/httpie/httpie)
- 현대판 `curl`
- 쉽고 사람 친화적인 사용성을 지향한다

# curlie
- [https://github.com/rs/curlie](https://github.com/rs/curlie)
- `curl` 인터페이스로 wrapping 해놓은 `httpie`
- 아웃풋은 `httpie`지만 인풋, 옵션 등 사용법이 `curl`과 동일

# bat
- [https://github.com/sharkdp/bat](https://github.com/sharkdp/bat)
- 현대판 `cat`
- 줄번호 표시, syntax highlight 등등

# nnn
- [https://github.com/jarun/nnn](https://github.com/jarun/nnn)
- 터미널 파일 매니저 (터미널에서 쓰는 탐색기라고 보면 된다)
- 보통 `mv`, `cp`, `mkdir` 등등 명령어를 개별로 써야하는데 `nnn` 속에서 단축키로 여러 작업 가능

# ranger
- [https://github.com/ranger/ranger](https://github.com/ranger/ranger)
- `nnn`이랑 비슷, 터미널 파일 매니저
- 좀 더 기능이 많고 복잡하다
- python으로 구현되어있어 좀 느리다

# tldr
- [https://github.com/tldr-pages/tldr](https://github.com/tldr-pages/tldr)
- 현대판 `man` ... 보다는 특정 명령어로 자주 사용하는 예제만 보여주는 `man`
- Community driven이라 사람들이 추가한 커맨드에 대한 정보만 나온다

# fzf
- [https://github.com/junegunn/fzf](https://github.com/junegunn/fzf)
- fuzzy finder라고, 검색한 문자열로 매칭될 수 있는 모든 파일, 커맨드 히스토리 등을 찾아줌
  - 부분 매칭, 전체 매칭 등 아주 강력하다
- 원래 터미널에는 없는 기능인데, 진짜 사기적으로 편하다. 다른건 몰라도 이건 꼭!! 설치하는것을 추천
  - 특히 커맨드 히스토리 찾는 기능이 정말 편하다
- 여기서 소개한 툴 중 유일하게 한국인이 만들었다 🇰🇷

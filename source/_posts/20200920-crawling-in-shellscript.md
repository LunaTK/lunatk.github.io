---
title: 쉘스크립트로 웹사이트 모니터링 크롤러 만들기
category:
  - Development
  - Terminal
toc: true
comments: true
date: 2020-09-20 12:44:28
tags:
  - bash
  - shellscript
  - crawling
thumbnail:
---

크롤링을 통해 웹사이트 내의 정보 업데이트를 모니터링하면 편한 경우가 많다. 나는 주로 파이썬이나 node.js를 이용해 크롤러를 만들었는데, 문득 쉘스크립트를 통해 훨씬 단순하게 구현할 수 있겠다는 생각이 들어 만들어 보았다.

<!--More-->

아마 단순 모니터링을 위한 크롤러 대부분은 아래와 같은 순서를 따를것이다.

1. HTTP 요청을 통한 웹사이트 데이터 가져오기
2. 데이터 전처리를 통해 필요한 부분 추출하기
3. 업데이트가 있는지 이전 데이터와 비교하기
4. 업데이트가 있을시 사용자에게 알림 보내기


그동안에 파이썬이나 node.js로 위같은 일을 하는 크롤러를 구현하기 위해 준비단계가 좀 길었다.

- 프로젝트 폴더 만들고
- npm init 하고
- dependency 설치하고...

그렇게 node.js로 만든 크롤러는 각 부분을 아래와 같은 방법으로 구현하였다.

1. HTTP 요청
  - request 라이브러리
2. 데이터 전처리
  - javascript 코드
3. 업데이트가 있는지 이전 데이터와 비교
  - mongodb 사용
4. 업데이트 알림
  - nodemailer 이용, 이메일로 알림 전송
5. 위 과정 반복
  - pm2로 백그라운드 실행


mongodb 사용은 약간 오버스러운 느낌이 있지만, 그걸 제외하더라도 크롤러 하나를 위해서 꽤 많은 라이브러리와 코드가 들어갔었다.

그런데 쉘스크립트를 공부하다 보니 저정도 기능이면 굳이 파이썬이나 node.js 쓰면서 라이브러리를 덕지덕지 가져다 쓸 필요도 없겠다는 생각이 들었다. 

# 쉘스크립트 구현

쉘스크립트로 크롤러를 만들때는 각 부분을 아래와 같은 방법으로 구현하였다.


1. HTTP 요청
  - curl를 통해 REST API 요청
2. 데이터 전처리
  - jq를 통해 JSON 전처리
3. 업데이트가 있는지 이전 데이터와 비교
  - 텍스트파일로 저장
4. 업데이트 알림
  - terminal-notifier
5. 위 과정청반복
  - cron


데이터 전처리를 위해 터미널 JSON 프로세서인 [`jq`](https://github.com/stedolan/jq), 맥북에서 알림을 띄워주는 프로그램인 [`terminal-notifier`](https://github.com/julienXX/terminal-notifier) 정도만 추가로 설치하였고, `curl`, `cron` 같이 대부분의 기능은 리눅스나 macOS에서 기본 제공하는 터미널 명령어로 해결이 가능했다.

# 사용한 명령어

각 명령어를 사용한 이유는 아래와 같다.

- curl
  - 터미널에서 HTTP 요청 하는데 별다른 방법이 있나 싶다
- jq
  - 내가 크롤링 할 사이트는 운좋게도 내부적으로 REST API 엔드포인트를 가지고 있었다
  - 리스폰스 데이터가 json이라 터미널용 JSON 프로세서인 `jq`를 사용하였다
  - 개발자들 가져다 쓰라고 오픈해놓은건 아니지만 사용에 딱히 제약은 없어보여서 찾아내서 사용하였다
  - 만약 리스폰스 데이터가 HTML이거나 하면 다른 프로세서 툴을 사용하면 될것같다
- terminal-notifier
  - 원래 구현대로 mail 보내는 방법도 생각해 보았다. `curl`로 gmail 보내는 방법도 있긴 하였지만[^1], 생각해보니 메일로 알림은 핸드폰으로 오는데 알림을 주로 보는 시간대는 내가 맥북을 쓰고있을때라 맥북 알림으로 오는게 더 좋을것 같았다.
  - 실제로 써보니 맥북 알림이 훨씬 편하였다
  - ![](/2020/09/20/20200920-crawling-in-shellscript/notification.png)
- cron
  - 쉘 스크립트 자체에 sleep을 주고 백그라운드에서도 실행하는 방법이 있겠지만, `cron`을 쓰는게 더 깔끔하고 매번 부팅할때마다 다시 실행할 필요가 없어서 선택하였다.

# 코드

아래는 실제 크롤링에 사용한 코드이다.

<script src="https://gist.github.com/LunaTK/9c7e40a581eb0ed50c48c51ad2892650.js"></script>

crontab 설정할때는 쉘 스크립트에서 실행할 바이너리 파일을 찾을 `PATH` 경로를 따로 명시해 주어야 했다. corntab 설정 파일은 아래와 같다.

```
PATH="/usr/local/bin:/usr/bin:/bin:$PATH"
NOTICE_PATH="/Users/LunaTK/Developer/notice-crawler"

*/10 * * * * cd $NOTICE_PATH && $NOTICE_PATH/notice-crawler.sh >> $NOTICE_PATH/log.txt 2>&1
```

크롤링은 10분마다 실행되게 하였고, `log.txt`에 업데이트가 있을시 기록하게 설정하였다.

# 장점

이 크롤링하는 코드는 원래 node.js로 만들어 놓았던게 있었는데, 쉘스크립트로 다시 구현해보면서 느낀 장점은 아래와 같다.

1. 관심사 분리가 터미널 명령어로 자연스럽게 되어서 코드가 훨씬 깔끔하다
   - HTTP요청은 `curl`, 데이터 전처리는 `jq`, 알림은 `terminal-notifier`
   - 각 부분을 연결하는 코드만 쉘스크립트로 작성
2. ~~왠지 터미널 빡고수가 된듯한 느낌이 들어 기분이 좋다~~

여러 부분에 응용해서 써먹으면 좋을것 같다.

# References

[^1]: https://stackoverflow.com/questions/14722556/using-curl-to-send-email
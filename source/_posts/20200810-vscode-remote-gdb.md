---
title: VSCode에서 원격으로 gdb 디버깅하기
category:
  - Development
  - Tools
toc: true
comments: true
date: 2020-08-10 17:13:37
tags:
  - gdb
  - debug
  - vscode
thumbnail:
---

macOS 환경에서 비주얼 스튜디오 코드를 통해 원격으로 리눅스 환경의 C/C++ 프로그램을 디버깅 해보자. Remote Debug를 통해 Remote Development 보다 가볍게 원격 환경에서 디버깅을 할 수 있다.

<!--More-->

# Preface

Visual Studio Code(VSCode)는 Remote Development[^1] 플러그인을 통해 원격 머신에 연결해 마치 로컬 머신에서 VSCode를 사용하는 것 같은 인터페이스를 제공한다.
이를 통해서 윈도우나 맥에서도 Docker나 SSH를 통해 리눅스에서와 똑같은 개발 환경을 경험할 수 있게 되었다. 올해 Remote Development가 업데이트 되면서 MacOS를 원격 환경으로 하여 사용할수도 있게 되면서, 집에 맥북을 두고 다니면서 어디에서든지 VSCode만 깔아서 내 맥북에서와 똑같은 개발 환경을 유지할 수 있게 되어 나도 아주 유용하게 사용하고 있다.

그런데 Remote Development 플러그인은 **Remote OS에 VS Code Server라는 꽤 무거운 데몬 프로세스를 실행해야 한다는 단점**이 있다. 

<center>
  <img src="https://code.visualstudio.com/assets/docs/remote/remote-overview/architecture.png"><br/>
  <small>
    그림 1. Remote Development Architecture
  </small>
</center>

내가 사용중인 개발 환경은 Local OS(맥북)에 C 소스코드가 있고, 소스코드 폴더를 Docker 인스턴스인 Ubuntu에 마운트 해서 빌드와 실행만 Remote OS 내에서 할 수 있는 구조였기 때문에, 모든 개발 과정이 아닌 디버깅 과정만 Remote OS에서 진행할 수 있으면 충분하였다.

내가 고려한 원격으로 C/C++을 디버깅 하는 방법은 크게 두가지가 있다.
1. **CLion**으로 원격 디버깅[^2]
  - 장점 : C/C++ 특화 IDE다 보니 디버깅 인터페이스가 좋지 않을까 싶다.
  - 단점 : 이미 VSCode로 개발중인데 IDE가 하나 추가되고, CLion에서 프로젝트도 새로 생성해야 한다. 프로젝트에 C/C++ 언어만 사용중인게 아니라 다양한 언어 지원이 중요하였다.
2. **VSCode**로 원격 디버깅
  - 장점 : 이미 사용중이라 gdb를 원격으로 연결만 하면 된다.
  - 단점 : 디버깅 인터페이스가 CLion보다는 하급인것 같다.

디버깅에 뭐 거창한 기능을 바라는게 아니라 그냥 쓰고있던 VSCode에서 원격으로 *gdb* 연결하는 방식을 선택하였다.

**말이 원격으로 gdb 연겷하는 것이지, 개발자 입장에서는 로컬 IDE에서 디버깅 하는것처럼 VSCode에서 브레이크 포인트도 걸 수 있고 다 할 수 있다. 내부적으로 gdb를 쓰는지는 설정할 때 빼고는 크게 체감은 안된다.**

# 설정 환경

- Local machine
  - macOS Catalina 10.15.6
  - GNU gdb (GDB) 9.2
  - VSCode 1.47.3

- Remote machine
  - Ubuntu 20.04 LTS (Docker instance)
  - docker desktop community 2.3.0.4
  - GNU gdb (Ubuntu 7.11.1-0ubuntu1~16.5) 7.11.1

# VSCode Remote Debugger 연결하기

맥북 VSCode에서 Remote OS에 *gdb*를 연결하기 위해서는 크게 세가지 단계가 필요하다.

1. macOS에 *gdb* 설치
2. linux에 *gdbserver* 설치
3. macOS에 VSCode 설정 

여러 포스팅을 참고 하였는데[^3][^4][^5], 안되는 부분들이 몇 있어서 조금씩 수정하면서 진행하였다.

## macOS에 gdb 설치

VSCode는 *gdb*가 내장되어 있지 않기 때문에, 따로 설치를 해야한다. macOS에는 기본적으로 *gdb*가 없는것 같은데, 나는 homebrew를 통해 최신버전을 설치를 해 주었다.

```bash
$ brew install gdb
```

<small>**참고**: macOS에서 linux 바이너리를 디버깅 해야하기 때문에 다른 포스팅들을 보면 `brew install gdb --with-all-targets` 를 통해 모든 아키텍쳐 정보를 포함하게 gdb를 설치하라 하는데, 나는 저렇게 하면 `--with-all-targets` 플래그에서 에러가 났다. 그래서 플래그 없이 설치했는데 잘 되는거 보니 최신 버전에는 기본적으로 모든 아키텍쳐 데이터가 포함되나 보다.</small>

<br/>

이때 설치 이후에도 두가지를 더 신경써야 한다.

### 1. 기본 설치되어 있는 gdb 링크 업데이트

`brew install gdb` 이후 출력하는 메세지를 잘 보면, 이미 `/usr/local/bin/gdb` 파일이 존재하는 경우 *brew*는 새로 설치된걸로 덮어쓰기를 하지 않는다. 그래서 직접 이를 overwrite 해주어야 한다. 

```bash
$ brew link --overwrite gdb
```

### 2. *gdb*에 codesign 적용

디버깅은 커널 권한을 사용하기 때문에, macOS에서는 프로그램에 서명을 해야 정상적으로 사용할 수 있다.

이 과정이 어렵진 않은데 좀 길기때문에, 기회가 되면 다음에 따로 포스팅을 하도록 하고 중요한 부분만 짚고 넘어가도록 하자.

1. self-signed 인증서 발급 후 gdb 바이너리 파일 서명
  - [https://sourceware.org/gdb/wiki/PermissionsDarwin](https://sourceware.org/gdb/wiki/PermissionsDarwin)[^6]
2. `.gdbinit` 설정
  - macOS 10.12 (Sierra) 부터는 gdb 7.12.1 이상을 사용해야 하고, 아래 옵션을 *gdb*를 실행하고 입력해 적용시켜 줘야 한다.
    - `set startup-with-shell off`
  - 또는 홈 디렉토리에 `.gdbinit` 파일을 만들어서 해당 옵션을 적어 놓으면 자동으로 *gdb*를 실행할때마다 적용한다.
    ```
    set startup-with-shell off
    ```

2번까지 완료했으면 hello world 를 만들어서 디버깅이 되나 확인해보도록 하자.

## linux에 gdbserver 설치

Linux에서는 원격 디버깅을 열어주는 *gdbserver*를 설치해야 하는데, *gdb* 패키지에 내장되어 있기 때문에 *gdb*가 깔려있으면 따로 설치할 필요가 없고, 안깔려 있으면 다음 명령어를 통해 설치해 주면 된다.

```bash
$ sudo apt install gdb
```

그다음 디버깅을 할 바이너리 파일을 `gdbserver`로 열어놓으면 된다.

```bash
$ gdbserver :9091 ./path/to/binary arg1 arg2 ...
```

당연한 얘기지만 `:9091`에서 포트는 꼭 9091일 필요는 없으며, command line arguments가 있으면 바이너리 파일을 명시한 이후 순서대로 적어주면 된다.

다음과 같은 메세지가 뜨면 성공!

```
Process ./path/to/binary created; pid = 171
Listening on port 9091
```
그리고 디버깅 세션이 끝나면 gdbserver가 꺼지기 때문에, VSCode에서 디버깅을 실행하기 전 매번 다시 위 명령어를 실행해야 한다. (계속 켜놓는 옵션이 있는지는 모르겠다)

## macOS VSCode에서 디버깅 설정

소스코드 파일을 보면서 디버깅을 하기 위해서는 당연한 얘기이지만, 소스코드가 있어야한다 (로컬에도 있어야 한다). 바이너리 파일은 로컬, 리모트에 둘다 있어야 하는데 소스코드는 로컬에만 있으면 되는것 같다.

우선 나는 Local OS에 있는 소스코드 폴더를 Docker에 마운트 시킨 상태이기 때문에, 모든 파일이 공유되고 있는 상황이다. 클라우드 서버로 원격 디버깅을 하는 상황 등에서는 *rsync*, *sshfs*, *nfs* 등을 사용해 파일을 동기화 시키는 방법을 주로 쓴다고 하는데, 내생각에는 실시간 파일 동기화는 필요 없기 때문에 미리 옮겨놓는 정도면 충분할 것 같다.

<br/>

아래 그림에서 **1번**을 누르면 디버깅 메뉴가 뜬다. **2번**을 누르면 `launch.json` 파일이 생성되고, 편집기로 열어준다.

<center>
  {% asset_img add_debug_config.png %}<br/>
  <small>
    그림 2. Debug Configuration 추가
  </small>
</center>

`configurations` 필드에는 어레이가 들어가는데, 다음과 같은 json 오브젝트를 추가해 주자.

{% codeblock launch.json lang:json %}
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "My Remote Debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceRoot}/path/to/bin",
            "miDebuggerServerAddress": "localhost:9091",
            "stopAtEntry": false,
            "cwd": "${workspaceRoot}/path/to/src",
            "environment": [],
            "externalConsole": true,
            "linux": {
              "MIMode": "gdb"
            },
            "osx": {
              "MIMode": "gdb"
            },
            "windows": {
              "MIMode": "gdb"
            }
          }
    ]
}
{% endcodeblock %}

이때, `program`, `miDebuggerServerAddress`, `cwd` 세개의 필드가 중요하다.

1. `program`
   - local에 있는 바이너리 파일의 경로를 써주어야 한다. 
2. `miDebuggerServerAddress`
   - 원격 서버의 IP 주소와 *gdbserver*를 실행항때 사용한 포트를 적어주어야 한다. 나는 docker 포트 매핑을 하였기 때문에 localhost를 IP로 사용하였다.
3. `cwd`
   - 이 경로를 기준으로 소스코드 파일을 찾으며, 컴파일 할때의 상대경로를 기준으로 한다.
   - 쉽게 생각해서 컴파일을 수행한 `cwd`와 일치시켜주면 된다. 

# 디버깅 시작하기

gdbserver를 시작하고, VSCode에서 추가한 설정으로 디버깅을 시작하면 gdbserver 에서 다음과 같은 메세지가 뜨면 성공이다.

```
Process ./path/to/binary created; pid = 178
Listening on port 9091
Remote debugging from host 172.17.0.1
```

<center>
  {% asset_img debugging.png %}<br/>
  <small>
    그림 3. Debugging 시작
  </small>
</center>

성공!

이제부터는 평소 IDE에서 디버깅 하던대로 하면 된다 😉

# References

[^1]: [VS Code Remote Development](https://code.visualstudio.com/docs/remote/remote-overview)
[^2]: [Remote Debug via GDB/gdbserver﻿ Of CLion With Docker](https://www.popit.kr/remote-debug-via-gdbgdbserver%EF%BB%BF-of-clion-with-docker/)
[^3]: [How to Debug Programs on Remote Server using GDBServer Example](http://blog.lovecoco.net/191)
[^4]: [Remote gdb from MacOS X to Linux with C++ STL pretty-printing](http://tomszilagyi.github.io/2018/03/Remote-gdb-with-stl-pp)
[^5]: [Remote debugging of C/C++ code with Visual Studio Code](https://o7g8.github.io/posts/vscode-remote-debugging/)
[^6]: [PermissionsDarwin](https://sourceware.org/gdb/wiki/PermissionsDarwin)
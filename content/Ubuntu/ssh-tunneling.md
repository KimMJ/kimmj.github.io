---
title: "SSH Tunneling 사용법"
menuTitle: "SSH Tunneling 사용법"
date:  2020-02-13T21:16:02+09:00
weight: 10
draft: false
tags: ["ubuntu", "tunneling"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## `-D` 옵션으로 socks proxy 사용하기

A라는 서버에서 B라는 서버에 있는 서비스를 보려고 합니다.
이 때, 해당 웹 어플리케이션은 B에서만 연결된 특정 IP로 통신을 하고 있고, 이 때문에 A에서 어플케이션이 제대로 동작하지 않는 상황입니다.

이 때 사용할 수 있는 것이 `-D` 옵션입니다.

예시

```bash
ssh -D 12345 user@server.com
```

해당 세션이 꺼져있지 않은 상태에서 A 서버에서 웹 브라우저가 localhost:12345를 프록시로 사용하도록 하면 해당 웹 어플리케이션이 제대로 동작합니다.

만약 windows라면 다음과 같이 진행하면 socks proxy를 사용하도록 할 수 있습니다.
CMD를 열고 다음과 같이 입력하면 새로운 창으로 chrome이 뜰 것입니다.

```cmd
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --user-data-dir="%USERPROFILE%\proxy-profile" --proxy-server="socks5://localhost:12345"
```

해당 창에서 어플리케이션을 실행하면 실제 B 서버의 desktop에서 동작하는 것과 같은 효과를 볼 수 있습니다.

## `-L` 옵션으로 특정 포트로 접속하기

A에서 B라는 서버의 어플리케이션을 사용하려 합니다.
이 어플리케이션은 특정 포트를 사용하고 있습니다.
쉬운 예시를 위해 jenkins를 B에서 구동한다고 하고 포트를 8080이라고 하겠습니다.

이 상황에서 A에서는 server-b.com:8080에 접속할 수 없는 상황입니다.

이 경우 `-L` 옵션을 사용하면 됩니다.

```bash
ssh -L 1234:localhost:8080 user@server-b.com
```

해당 설정을 읽어보자면, A서버의 `localhost:1234`로 들어오는 요청들을 `user@server-b.com`으로 접속한 뒤 `localhost:8080`으로 보내라는 의미입니다.

다음과 같은 설정도 있을 수 있습니다.

```bash
ssh -L 12345:server-c.com:8080 user@server-b.com
```

이 설정은 A에서 C로 직접 통신이 안되지만, B에서 C로 통신이 되는 상황입니다.

이 때, 위의 설정은 `localhost:12345`로 들어오는 요청들을 `user@server-b.com`으로 접속한 뒤 `server-c.com:8080`으로 보내라는 의미입니다.

## `-R` 옵션을 이용하여 Reverse Proxy 사용하기

A에서 B로 ssh 접속을 하고 있습니다.
이 때, B에서 A로는 접속이 방화벽으로 막혀있는 상황입니다.

이런 상황에서 `-R` 옵션을 주면 B에서도 A로 접속할 수 있습니다.

```bash
ssh -R 12345:localhost:22 user@server-b.com
```

해당 설정은 A서버에서 접속하는 localhost의 22 포트를 B서버의 12345 포트와 연결하라는 의미입니다.

그 다음 해당 세션이 유지된 상태에서 B에서 다음과 같이 접속합니다.

```bash
ssh localhost -p 12345
```

그러면 B서버에서 A서버로 접속이 가능한 것을 볼 수 있습니다.

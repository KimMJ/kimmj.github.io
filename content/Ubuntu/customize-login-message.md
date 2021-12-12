---
title: "Ubuntu의 Login Message 수정하기"
date:  2020-03-11T14:20:42+09:00
weight: 10
draft: false
tags: ["ubuntu", "login-message", "motd"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## TLDR

{{% expand %}}

> This package seeks to make the `/etc/motd` (Message of the Day) more dynamic and valuable, by providing a simple, clean framework for defining scripts whose output will regularly be written to `/etc/motd`.

Ubuntu에서는 `/etc/update-motd.d` 안에 있는 파일들을 확인하여 console, ssh 등 어떤 방법으로든 로그인했을 때 메시지를 띄워줍니다.
여기서 파일들을 사전순으로 로딩하게 됩니다.

따라서 해당 폴더에 적절한 파일들을 생성하게 된다면 로그인 시 출력되는 메시지를 조작할 수 있습니다.


{{% /expand %}}

## 적용법

### `/etc/update-motd.d/`로 이동

```bash
cd `/etc/update-motd.d`
```

### `99-message` 파일 생성

`99-message` 이름으로 파일을 생성합니다.

```bash
#!/bin/sh

printf "hello. this is customized message.\n"
printf "\n"
```

그다음 실행파일로 변경해줍니다.

```bash
chmod +x 99-message
```

### 다시 세션 로그인하기

```bash
hello. this is customized message.

Last login: Wed Mar 11 14:19:56 2020 from x.x.x.x
wanderlust@wonderland $
```

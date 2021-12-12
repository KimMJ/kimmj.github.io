---
title: "Gateway를 이용하여 SSH 접속하기"
menuTitle: "Gateway를 이용하여 SSH 접속하기"
date:  2020-02-12T22:08:16+09:00
weight: 10
draft: false
tags: ["ssh", "linux"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## ssh cli 이용하는 방법

`-J` 옵션을 이용한다.

```bash
ssh user@server -J user2@server2
```

두개 이상의 경우 `,`로 구분한다.

예: user2@server2로 접속 후 user3@server3로 접속한 뒤 user@server로 접속해야 할 경우

```bash
ssh user@server -J user2@server2,user3@server3
```

이 상황에서 `ssh-copy-id`를 이용해 패스워드를 입력하지 않고 이동하려면

```bash
localuser@localhost $ ssh-copy-id user2@server2
localuser@localhost $ ssh user2@server2
user2@server2 $ ssh-copy-id user3@server3
user2@server2 $ ssh user3@server3
user3@server3 $ ssh-copy-id user@server
```

이후 `ssh`를 통해 진입하면 패스워드 없이 접속 가능.

만약 port가 필요한 경우 `server:port` 형태로 입력

```bash
ssh user@server:port -J user2@server2:port2,user3@server3:port3
```

## ssh config 파일 이용하는 방법

```
Host server
    HostName remote-server
    User user
    ProxyJump gateway2
Host server2
    HostName gateway1
    User user2
Host server3
    HostName gateway2
    User user3
    ProxyJump gateway1
```

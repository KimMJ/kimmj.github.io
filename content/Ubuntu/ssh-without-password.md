---
title: "password 없이 ssh 접속하기"
date:  2020-02-15T15:48:57+09:00
weight: 10
draft: false
tags: ["ssh", "passwordless"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

자주 접속하는 서버에 패스워드를 항상 입력하는 것은 귀찮은 일이 될 것입니다.

여기에서는 ssh key를 생성하고, 이를 이용하여 인증을 해 password를 입력하지 않는 방법을 알아볼 것입니다.

## ssh-keygen을 통한 ssh key 생성{#ssh-keygen}

`ssh` 접속을 할 때 password를 입력했던 것처럼, 항상 `ssh` 접속을 위해서는 인증을 위한 key가 필요합니다.

인증에 사용할 키를 `ssh-keygen`으로 생성하는 방법은 다음과 같습니다.

```bash
ssh-keygen -t rsa -b 4096
```

`-t`는 `rsa` 알고리즘을 통해 key를 생성하겠다는 의미이며, `-b`는 key의 사이즈를 정해주는 것입니다.

다른 알고리즘들과 다른 옵션들은 <https://www.ssh.com/ssh/keygen>에서 더 확인할 수 있습니다.

이제 `ssh` 인증을 위한 public key를 생성하였습니다.

## ssh-copy-id를 통한 public key 복사

[위](#ssh-keygen)에서 생성한 public key를 접속하고자 하는 서버에 복사를 하면, 서버에 접속할 때 서버는 해당 파일을 참조하여 인증을 시도할 것입니다.

```bash
ssh-copy-id user@server
```

이 때 최초 1회만 패스워드를 입력하면 됩니다.
그 뒤 다시한번 로그인을 시도해보면 패스워드 없이 로그인에 성공할 것입니다.

<!-- ## ssh jump host를 사용할 때 password 없이 접속하는 방법

예를 들어, 다음과 같은 명령어를 통해 A 서버에서 B 서버를 경유하여 D 서버에 접속한다고 가정해 보겠습니다.

```bash
ssh user-d@host-d -J user-b@host-b,user-c@host-c
```

이럴 경우,  -->
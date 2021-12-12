---
title: "git-secret을 통한 github 파일 암호화"
date:  2020-03-07T23:39:23+09:00
weight: 10
draft: false
tags: ["git-secret", "github", "git"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

git을 사용하다 보면 password나 credential같은 정보가 git에 올라가는 경우가 종종 있습니다.
aws같은 cloud provider의 crediential을 git에 생각없이 올리고, 이를 다른 해커가 크롤링을 통해 얻어 비트코인을 채굴하는 사례도 있었습니다.

이처럼 보안이 필요한 파일을 git에 올릴 때, 암호화를 하여 업로드하는 방법이 있습니다.

<https://github.com/sobolevn/git-secret>

## `git-secret`

`git-secret`은 파일에 대한 암화를 지원하기 위해 사용되는 프로그램입니다.

`git-secret add` 명령어를 통해 파일을 암호화하고, `git-secret reveal`을 통해 복호화합니다.
이 때 `gpg`를 이용하게 됩니다.

## Install

사용자 환경에 따라 `brew`, `apt`, `yum`을 통해 설치할 수 있습니다.
또한 `make`를 통해서 빌드할 수도 있습니다.

저의 경우 Ubuntu를 사용하고 있고 `apt`가 있기 때문에 이를 이용하여 설치하였습니다.

```bash
apt install git-secret
```

## 파일 암호화하기

### `gpg`를 이용하여 키 생성하기

먼저, `gpg`가 설치되어있는지 확인하고 설치합니다.

```bash
$ gpg -h | grep version
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
# 위와같은 결과가 나오지 않는다면, 설치해야합니다.
$ apt install gpg
```

설치가 되었으면 gpg key를 생성합니다.

<pre style="color: rgb(248, 248, 242); background-color: rgb(39, 40, 34); tab-size: 4; user-select: auto;">
<code class="language-bash hljs" data-lang="bash">
$ gpg --full-generate-key
gpg (GnuPG) 2.2.4; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)

Your selection? <span style="background-color: rgb(100,80,0);"> 1 </span>
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      &lt;n&gt;  = key expires in n days
      &lt;n&gt;w = key expires in n weeks
      &lt;n&gt;m = key expires in n months
      &lt;n&gt;y = key expires in n years
Key is valid for? (0) <span style="background-color: rgb(100,80,0);"> 0 </span>
Key does not expire at all
Is this correct? (y/N) <span style="background-color: rgb(100,80,0);"> y </span>

GnuPG needs to construct a user ID to identify your key.

Real name: <span style="background-color: rgb(100,80,0);"> Ibiza </span>
Email address: <span style="background-color: rgb(100,80,0);"> my@email.com </span>
Comment: <span style="background-color: rgb(100,80,0);"> Wanderlust </span>
You selected this USER-ID:
    "Ibiza (Wanderlust) &lt;my@email.com&gt;"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? <span style="background-color: rgb(100,80,0);"> O </span>
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
</code><span class="copy-to-clipboard" title="Copy to clipboard" style="user-select: auto;"></span></pre>

전부 완료되기까지 꽤 시간이 많이 걸립니다.

### `git-secret`을 통한 암호화

본격적으로 시작하기 위해 `git-secret`을 `init`합니다.

```bash
git secret init
```

그 다음 `git-secret`에 사용자를 추가합니다.

```bash
git secret tell my@email.com
```

이제 secret을 통해 암호화할 파일은 원본이 github에 올라가지 않게 하기 위해 `.gitignore`에 추가하도록 합니다.
저의 경우 `./config.toml`이라는 파일을 암호화하기 위해 다음과 같이 추가하였습니다.

```gitignore
config.toml
```

그 다음 `git-secret`이 암호화 하도록 파일을 추가합니다.

```bash
git secret add config.toml
```

이 때 다음과 같은 에러가 발생했는데, github issue 중 비슷한 문제가 있어 이를 통해 해결할 수 있었습니다.

```bash
$ git secret add config.toml
config.toml is not a file. abort.
$ git rm --cached config.toml
$ git secret add config.toml
1 items added.
```

이제 `hide`를 통해 암호화할 수 있습니다.

```bash
git secret hide
```

이후 `ls`를 통해 확인해보면 `.secret`이라는 postfix가 들어간 파일을 보실 수 있습니다.

```bash
$ ls config.toml*
config.toml config.toml.secret
```

### `git-secret`을 이용한 복호화

`reveal`을 통해 파일을 복호화할 수 있습니다.

```bash
$ git secret reveal
File '/home/wanderlust/Ibiza/config.toml' exists. Overwrite? (y/N) y
done. all 1 files are revealed.
```

이제 `.secret` 파일을 github에 올리면 안전하게 이용할 수 있습니다.

## 다른 컴퓨터에서 접속할 때는 어떻게 해야하나요?

`gpg` 알고리즘은 public key로 암호화하여 private key로 복호화합니다.
즉, 우리는 `gpg` key를 생성한 host에서 다른 host로 private key를 복사해주어야 합니다.

따라서 다음과 같이 private key를 복사하고, 이를 import합니다.

```bash
# gpg key를 생성했던 host
wanderlust@wonderland $ gpg --export-secret-keys my@email.com > private-key.asc

# key 복사
wanderlust@wonderland $ scp private-key.asc wanderlust@wonderland-laptop:~/

# gpg key를 import할 host
wnaderlust@wonderland-laptop $ gpg --import ~/private-key.asc
```

그 다음 `tell` 옵션을 사용합니다.

```bash
git secret tell my@email.com
```

마지막으로 `reveal` 옵션을 통해 복호화합니다.

```bash
$ git secret reveal
done. all 1 files are revealed.
```

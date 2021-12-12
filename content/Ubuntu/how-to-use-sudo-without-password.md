---
title: "sudo를 password 없이 사용하기"
date: 2020-01-08T01:51:54+09:00
tags: ["sudo", "passwordless", "ubuntu"]
weight: 10
pre: "<i class='fas fa-minus'></i>&nbsp;"
---


`/etc/sudoers`는 `sudo`를 사용할 수 있는 파일입니다.
이 파일을 열어보면 다음과 같은 글이 적혀 있습니다.

> Please consider adding local content in /etc/sudoers.d/ instead of directly modifying this file

즉, 직접 이 파일을 수정해서 `sudo` 권한을 주지 말고, `/etc/sudoers.d/` 폴더 내에 파일을 추가하라는 의미입니다.

이 곳에는 `/etc/sudoers`와 마찬가지로 계정에 대한 설정을 추가할 수 있습니다.
그리고 `/etc/sudoers`에서는 "NOPASSWD"라는 옵션을 주어 password없이 타 계정의 권한을 가지게 만들 수 있습니다.

이 두가지를 종합하여 내 linux 계정이 sudo 명령어를 입력할 때, 즉 root 권한을 가지게 될 때 password를 입력하지 않도록 설정할 수 있습니다.

```bash
export ACCOUNT=$(whoami)
echo "$ACCOUNT ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$ACCOUNT
sudo chmod 0440 /etc/sudoers.d/$ACCOUNT
```

이제 `sudo` 명령어를 쳐도 더 이상 password를 입력하라는 출력이 뜨지 않습니다.
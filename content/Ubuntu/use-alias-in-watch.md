---
title: "watch를 사용할 때 alias 이용하기"
date:  2020-02-22T23:33:02+09:00
weight: 10
draft: false
tags: ["watch", "ubuntu", "alias"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`watch`는 정해진 시간동안 뒤에 적은 명령어를 실행해주는 프로그램입니다.
가령 `kubernetes`를 다룰 때 `watch kubectl get pods -n kube-system`을 통해 kube-system 네임스페이스에 있는 파드들을 지속적으로 모니터링 할 수 있습니다.

그러나 `watch`는 alias된 명령어를 인식하지 못합니다.

```bash
$ ll
total 44K
drwxrwxr-x  2 wanderlust wanderlust 4.0K  1월  7 20:38 archetypes
-rw-rw-r--  1 wanderlust wanderlust 2.9K  1월 21 23:40 config.toml
drwxrwxr-x 16 wanderlust wanderlust 4.0K  2월 22 23:08 content

$ watch ll

Every 2.0s: ll

sh: 1: ll: not found
```

이 때 해결할 수 있는 가장 편한 방법은 `watch` 자체를 alias 시켜버리는 것입니다.

저는 `zsh`을 사용하고 있으므로 `~/.zshrc`에 다음과 같이 추가하였습니다.

```zshrc
alias watch='watch '
```

그 다음 설정파일을 다시 불러오거나 새로운 session을 생성합니다.

```bash
# 설정파일 다시 불러오기
source ~/.zshrc
```

다시 `watch`를 하여 확인해 봅니다.

```bash
$ watch ll
Every 2.0s: ls --color=tty -lh

total 44K
drwxrwxr-x  2 wanderlust wanderlust 4.0K  1월  7 20:38 archetypes
-rw-rw-r--  1 wanderlust wanderlust 2.9K  1월 21 23:40 config.toml
drwxrwxr-x 16 wanderlust wanderlust 4.0K  2월 22 23:08 content
```

정상적으로 alias된 명령어를 인식하게 되었습니다.

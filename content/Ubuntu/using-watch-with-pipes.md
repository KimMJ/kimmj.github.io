---
title: "pipe를 사용한 명령어를 watch로 확인하기"
date:  2020-02-23T16:07:38+09:00
weight: 10
draft: false
tags: ["watch", "pipe"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`pipe`(`|`)는 `grep`과 다른 기타 명령어들과 함께 사용하면 좀 더 다양한 작업을 할 수 있습니다.

`watch`는 특정 명령어를 주기적으로 입력하여 결과 메시지를 확인합니다.
즉, 무엇인가를 모니터링할 때 주로 사용하곤 합니다.

바로 본론으로 들어가서 pipe를 사용한 명령어를 watch로 확인하는 방법은 다음과 같습니다.

```bash
watch '<command>'
```

위와같이 quote로 감싸주세요.

`ls -al`을 가지고 확인해 보도록 하겠습니다.

```bash
$ ls -al | grep config
-rw-rw-r--  1 wanderlust wanderlust 2.9K  1월 21 23:40 config.toml

$ watch ls -al | grep config # quote를 사용하지 않은 것
^C # 결과 출력되지 않음

$ watch "ls -al | grep config"
Every 2.0s: ls -al | grep config

-rw-rw-r--  1 wanderlust wanderlust 2960  1월 21 23:40 config.toml

$ watch "ll | grep config"
Every 2.0s: ll | grep config

sh: 1: ll: not found
```

특히 마지막을 보시면 alias된 명령어는 인식하지 못하는 것을 알 수 있습니다.

[다른 페이지](../use-alias-in-watch)에서 했던 `alias watch="watch "`를 적용했다고 하더라도, alias된 것을 quote로 감쌌을 때에는 인식하지 못하는 것을 볼 수 있습니다.

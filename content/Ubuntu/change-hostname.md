---
title: "Hostname 변경하기"
date: 2020-01-14T01:00:02+09:00
draft: false
weight: 10
tags: ["hostname", "ubuntu"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

hostname을 바꾸는 일은 흔치 않지만 최초 셋업할 때 많이 사용하곤 합니다.

```bash
# hostnamectl set-hostname <host name>
hostnamectl set-hostname wonderland
```

변경 후 터미널을 끄고 재접속을 하면 변경된 사항을 볼 수 있습니다.

```bash
hostname
```
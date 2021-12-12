---
title: "Docker를 sudo없이 실행하기"
date:  2020-02-29T02:51:19+09:00
weight: 10
draft: false
tags: ["docker", "sudo"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`docker` 명령어는 `docker` group으로 실행됩니다.
그러나 저희가 기존에 사용하던 일반 user는 해당 group에 속하지 않기 때문에 `docker` 명령어를 쳤을 때 permission에 관한 에러가 발생하게 됩니다.

이 때 다음과 같이 조치를 하면 `sudo` 없이 user가 `docker` 명령어를 사용할 수 있게 됩니다.

```bash
sudo usermod -aG docker $USER
```

session을 다시 열고 `docker ps` 명령어를 입력하여 에러가 발생하는지 확인합니다.

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

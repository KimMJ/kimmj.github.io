---
title: "http를 사용하는 docker registry를 위한 insecure registry 설정"
date:  2020-03-15T04:18:15+09:00
weight: 10
draft: false
tags: ["docker", "insecure-registry", "docker-registry"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

회사같은 곳에서는 보안상의 문제 때문에 Dockerhub에다가 이미지를 올리지 못하는 경우가 많습니다.
이를 위해서 docker에서도 docker registry라는 툴을 제공하는데요, 이는 자신의 local server를 구축하고, dockerhub처럼 이미지를 올릴 수 있는 툴입니다.

이러한 docker registry는 사용자의 환경에 따라 http를 사용하는 경우가 있습니다.
이 때, `docker`는 default로 https 통신을 하려 하기 때문에 문제가 발생합니다.
이 경우 다음과 같이 조치를 하면 http 통신을 할 수 있습니다.

## 절차

### insecure-registry 설정

`/etc/docker/daemon.json` 파일을 열어 예시처럼 작성합니다.
없을 경우 생성하면 됩니다.

```json
{
    "insecure-registries" : ["docker-registry:5000"]
}
```

### docker 재시작

```bash
# flush changes
sudo systemctl daemon-reload
# restart docker
sudo systemctl restart docker
```

---

이제 다시한번 `docker pull` 명령어를 통해 이미지가 제대로 다운로드 되는지 확인합니다.

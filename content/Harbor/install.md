---
title: "Harbor 설치"
date:  2020-03-03T10:07:50+09:00
weight: 10
draft: false
tags: ["harbor", "docker-registry"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## `docker-compose` 설치

```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
# 설치 후 docker-compose 명령어가 실패한다면, symbolic link를 직접 걸어주도록 합니다.
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
$ docker-compose --version
docker-compose version 1.24.1, build 1110ad01
```

## harbor installer 다운로드

[공식 Github](https://github.com/goharbor/harbor/releases)에서 원하는 인스톨러를 다운로드 받습니다.
저는 online installer를 사용할 예정입니다.

다운로드가 완료되었으면 압축을 해제합니다.

```bash
$ tar xvf harbor-*.tgz
harbor/prepare
harbor/LICENSE
harbor/install.sh
harbor/common.sh
harbor/harbor.yml
```

## `harbor.yml` 설정

`vi`로 `harbor/harbor.yml`을 열고 적당하게 편집합니다.

여기서는 http로 간단하게 배포하는 설정을 해볼 것입니다.

```yaml
hostname: <domain or IP> # 192.168.x.x
```

그리고 https와 관련된 value를 모두 주석처리해줍니다.

```yaml
# https:
  # https port for harbor, default is 443
  # port: 443
  # The path of cert and key files for nginx
  # certificate: /your/certificate/path
  # private_key: /your/private/key/path
```

proxy 환경이 아니라면 더 손댈 곳은 없습니다.
proxy 환경일 경우 하단에 있는 proxy 설정을 `/etc/environment` 등을 참조하여 미리 적혀있는 부분을 추가하여 작성하시면 됩니다.

## install

install시 `clair`, `notary`, `chart-museum`을 함께 설치할 수 있습니다.

여기서 `notary`는 https 설정이 필요하기 때문에 생략하고 나머지 두개를 설치합니다.

```bash
~/harbor$ ./install.sh  --with-clair --with-chartmuseum
```

## 확인

이제 `hostname`에서 설정한 곳으로 접속하여 확인합니다.
설정을 바꾸지 않았다면 80포트로 접속할 수 있기 때문에 address만 입력해주면 접속할 수 있습니다.

기본 ID/PW는 `admin/Harbor12345` 입니다.

![harbor.png](/images/Harbor/harbor.png)

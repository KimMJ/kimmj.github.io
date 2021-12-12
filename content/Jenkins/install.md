---
title: "Jenkins Install"
menuTitle: "Install"
date:  2020-02-11T17:20:41+09:00
weight: 1
draft: false
tags: ["jenkins", "install"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Configure docker-compose.yaml

다음과 같이 `docker-compose.yaml` 파일을 적절한 디렉토리에 생성합니다.

```yaml
version: '2'
services:
  jenkins:
    image: 'jenkins/jenkins:lts'
    ports:
      - '38080:8080'
      - '38443:8443'
      - '50000:50000'
    volumes:
      - 'jenkins_data:/var/jenkins_home'
volumes:
  jenkins_data:
    driver: local
    driver_opts:
      type: none
      device: $PWD/jenkins_data
      o: bind
```

`jenkins_data`는 jenkins가 사용할 데이터들입니다.
이를 local에 폴더로 만들어줍니다.

```bash
mkdir jenkins_data
```

## Start Jenkins

이제 `docker-compose` 명령어를 통해 실행합니다.

```bash
sudo docker-compose up -d
```

`http://$IP:38080` 으로 접속할 수 있습니다.
로컬에 설치하셨다면 `http://localhost:38080`으로 접속하면 됩니다.

## Jenkins 세팅

처음에 다음과 같은 화면을 볼 수 있습니다.

![Jenkins-start](/images/Jenkins/Jenkins-start.png)

여기서 `/var/jenkins_home/secrets/initialAdminPassword`의 내용을 아래 칸에 입려하라고 지시합니다.

저희는 앞서 `$PWD/jenkins_data`를 `/var/jenkins_home`에 binding했으므로, `$PWD/jenkins_data/secrets/initialAdminPassword`를 확인하면 됩니다.
또는 docker container에 직접 접속해서 해당 path로 이동 후 값을 확인해도 됩니다.

이제 다음과 같은 화면이 뜰 것입니다.

![Jenkins-plugin-install](/images/Jenkins/Jenkins-plugin-install.png)

여기서는 왼쪽의 `Install suggested plugins`를 선택해줍니다.

다음과 같이 설치가 진행됩니다.

![Jenkins-plugin-installing](/images/Jenkins/Jenkins-plugin-installing.png)

위의 plugin 설치과정이 완료되고 나면 다음과 같은 admin 계정 생성에 관한 화면이 나옵니다.

![Jenkins-admin-user](/images/Jenkins/Jenkins-admin-user.png)

알맞게 입력을 한 뒤 저장하면 접속 URL을 작성하라고 합니다.
default로 적어져 있으니 확인후 변경이 필요하면 바꾸면 됩니다.

![Jenkins-address](/images/Jenkins/Jenkins-address.png)

여기까지 하면 기본 설정은 다 끝났습니다.
Jenkins가 준비될 때까지 잠시 기다리면 home 화면이 뜹니다.

![Jenkins-home](/images/Jenkins/Jenkins-home.png)

## Reference

* <https://hub.docker.com/_/jenkins/>

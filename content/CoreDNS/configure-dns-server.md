---
title: "CoreDNS를 통한 Dns Server 구축하기 with docker"
date: 2020-11-21T01:14:13+09:00
weight: 10
draft: false
tags: ["docker", "docker-compose", "network", "bridge", "container"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`CoreDNS`는 `kubernetes`에서 dns resolve를 위해 사용하고 있는 오픈소스이다.
실제로 `kubernetes`의 `kube-system` 네임스페이스를 보면 `coredns-xxx` 파드가 떠있는 것이 보일 것이다.

사내에서 `CoreDNS`를 통한 DNS 서버를 구축하며 알게된 사실들, 구축 방법등을 공유한다.

`CoreDNS`를 시작하는 것은 정말 어렵지 않다.
`docker`를 통해 컨테이너만 실행시켜도 우선은 `CoreDNS`를 맛볼 수 있다.
다만 나의 경우 `docker`를 그 자체로 실행시키는 것 보다 `docker-compose`를 통해 실행시키는 것을 더 선호하는 편이다.
`docker-compose`의 장점이라면 복잡한 `docker` 명령어를 일일이 기억하고 있지 않아도 `docker-compose up -d` 명령어 하나로 사전에 설정된 명령을 실행시킬수 있다는 것이다.

`CoreDNS`의 `docker-compose.yml` 구성은 구글링으로도 쉽게 구할 수 있으며, 내가 구한 설정은 다음과 같았다.

```yaml
version: '2.1'

services:
  coredns:
    image: coredns/coredns
    restart: always
    command: -conf /root/coredns/config/Corefile
    ports:
      - 53:53/udp
    volumes:
      - ./config:/root/coredns/config/
```

`volumes`를 보면 config 폴더를 `docker-compose.yml` 파일과 함께 위치했다는 사실을 알 수 있다.
따라서 다음과 같은 디렉토리 구성을 한다.

```sh
$ tree
.
├── config
│   ├── Corefile
│   ├── kubeconfig
│   └── my.host.name.db
└── docker-compose.yml
```

여기서 `kubeconfig`는 `CoreDNS`의 `kubernetes` 플러그인을 사용해보기 위한 목적으로, 어떻게 `kubernetes`에서 DNS resolve를 하는 것이 밖에서도 볼 수 있는지 확인해보려고 구성해 보았다.
물론 클러스터 외부에서는 `kubernetes`의 서비스 대역과 통신이 불가능하기 때문에 거의 사용할 일이 없다.

`my.host.name.db`는 DNS resolve를 위한 DNS db 파일이다.

`Corefile`은 `CoreDNS`의 설정파일로, 매칭되는 Domain Name에 대해 어떤 rule로 DNS resolve를 할지 결정하는 파일이다.

나는 다음과 같이 `Corefile`을 구성했다.

```text
my.host.name {
    file /root/coredns/config/my.host.name.db
    cache
    log
    errors
}

xx.x.xx.172.in-addr.arpa {
    whoami
}

. {
    forward . 8.8.8.8
    log
    cache
    errors
}
```

설정을 하나씩 보면 우선 `my.host.name`을 가지는 Domain에 대해서는 `my.host.name.db` 파일을 읽도록 해 두었다.

`xx.x.xx.172.in-addr.arpa` 의 경우 DNS server의 아이피로 `hostname`을 알려주고 싶을 경우 설정하는 것이다.

`.`으로 설정한 부분은 default 세팅이라고 봐도 무방하다.
여기에서는 `8.8.8.8`로 위에 설정한 두개의 Domain 빼고는 모두 포워딩한다.
즉, 내가 resolve를 하는게 아니라 `8.8.8.8`에서 resolve한 결과값을 리턴하겠다는 뜻이다.

위와같이 설정을 하고 난뒤 다음과 같이 입력한다.

```sh
docker-compose up -d
```

내 서버로 resolve를 하고싶다면, `netplan`의 nameserver 부분을 수정하거나 `/etc/resolv.conf`를 수정하면 된다.

`/etc/resolv.conf`에 관해서는 다른 페이지에서 다뤄보도록 한다.

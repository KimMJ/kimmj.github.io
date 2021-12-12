---
title: "[docker-compose] container에서 다른 container로 접속하기"
date:  2020-02-21T18:58:15+09:00
weight: 10
draft: false
tags: ["docker", "docker-compose", "network", "bridge", "container"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## 배경

docker-compose에서는 network bridge를 설정합니다.
이 bridge로 내부 통신을 하게 되죠.
여기서 port-forward를 통해 외부로 서비스를 expose하게 되면 host의 IP와 port의 조합으로 접속할 수 있습니다.

그런데 저는 네트워크 설정의 문제인지, 하나의 container에서 host IP로 접속이 불가능했습니다.
그러면서도 저는 어떻게든 다른 docker-compose의 서비스로 네트워킹이 됐어야 했습니다.
정확히 말하자면 `harbor`라는 서비스(docker registry)에서 `jenkins`로 webhook을 날려야 하는 상황이었죠.

먼저 시도했던 것은 `jenkins`의 ip를 `docker inspect jenkins_jenkins_1`을 통해 알아내고, 이를 통해 webhook을 전송하는 것이었습니다.
그러나 실패했죠.

다음으로 생각해본 것은, 그렇다면 `jenkins`를 `harbor`의 bridge로 연결해보자는 것이었습니다.

따라서 다음과 같은 조치를 취해주었습니다.

## container에서 다른 container로 접속하기

### 이미 동작중인 container에 bridge 연결

```bash
docker network connect harbor_harbor jenkins_jenkins_1
# docker network connect <bridge> <container>
```

### 연결된 bridge에서의 ip 확인

```bash
$ docker inspect jenkins_jenkins_1

                  "harbor_harbor": {
                    "IPAMConfig": {},
                    "Links": null,
                    "Aliases": [
                        "---"
                    ],
                    "NetworkID": "---",
                    "EndpointID": "---",
                    "Gateway": "172.24.0.1",
                    "IPAddress": "172.24.0.11", # <--------IP

```

### 연결 확인

```bash
harbor-core $ curl 172.24.0.11:8080
```

이 때, 연결은 host가 port-forward한 port가 아닌, container가 expose하고 있는 port를 입력해주어야 합니다.

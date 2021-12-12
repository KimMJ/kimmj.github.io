---
title: "Install Prometheus"
menuTitle: "Install"
date:  2020-01-30T18:00:24+09:00
weight: 1
draft: false
tags: ["install", "prometheus"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Prometheus

[Prometheus](https://prometheus.io/)는 opensource monitoring system입니다.
음악을 하는 사람들이 많이 이용하는 사이트인 SoundCloud에서 개발된 오픈소스입니다.

`PromQL`이라는 Query문을 사용하여 metric을 수집할 수 있습니다.
자세한 내용은 나중에 따로 포스트를 작성하도록 하겠습니다.

이 문서에서는 docker-compose를 통해 간단하게 prometheus를 설치해볼 것입니다.
또한 prometheus와 뗄레야 뗄 수 없는 단짝 Grafana도 함께 설치할 것입니다.

## Prometheus의 설치

제가 docker-compose를 선호하는 이유는 너무나도 간단하게, dependency가 있는 어플리케이션을 설치할 수 있기 때문입니다.
아래에 예시에서도 잘 드러나있습니다.

저는 `monitoring/` 폴더 아래에 `docker-compose.yaml`이라는 이름으로 파일을 생성했습니다.

```yaml
version: '3'

networks:
  back-tier:

services:
  prometheus:
    image: prom/prometheus
    restart: unless-stopped
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    networks:
      - back-tier
  grafana:
    image: grafana/grafana:6.5.3
    ports:
      - 9080:3000
    user: "0" # $(id -u) https://community.grafana.com/t/new-docker-install-with-persistent-storage-permission-problem/10896/5
    depends_on:
      - prometheus
    volumes:
    - ./grafana/provisioning/:/etc/grafana/provisioning/
    - ./grafana_data:/var/lib/grafana
    networks:
      - back-tier
```

위의 예시에서 `services.grafana.user="0"`으로 설정이 되어있는 것이 보이실 것입니다.
이 부분을 `$(id -u)`의 결과값으로 변경하시면 됩니다.
관련된 내용은 옆에 [주석](https://community.grafana.com/t/new-docker-install-with-persistent-storage-permission-problem/10896/5)에서도 확인이 가능합니다.

이렇게 하고난 뒤, 설치하는 방법은 너무나도 간단합니다.

```bash
cd monitoring
docker-compose up -d
```

잠시 내리고 싶을땐 다음과 같이 하면 됩니다.

```bash
docker-compose down
```

설정상, Prometheus는 9090 포트를 통해서 expose 되어있습니다.
또한 Grafana는 9080 포트를 사용하고 있습니다.
Grafana의 기본 ID/PW는 `admin/admin`입니다.

## 설정

Prometheus를 사용하기 위해서는 `prometheus.yml`이라는 파일을 관리해주어야 합니다.
이곳에서 어느 서비스의 metric을 가지고 올지 설정할 수 있습니다.

해당 파일은 `docker-compose.yaml`을 참조해 보았을 때, `monitoring/config/prometheus.yml`을 수정하면 된다는 것을 알 수 있습니다.
수정 후에는 `docker-compose down`을 통해 잠시 내렸다가 `docker-compose up -d`를 통해 올리면 수정사항이 반영됩니다.

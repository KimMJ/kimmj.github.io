---
title: "Choose a Storage Service"
menuTitle: "Choose a Storage Service"
date:  2020-01-19T00:46:44+09:00
weight: 5
draft: false
tags: ["install", "spinnaker", "minio"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

Spinnaker들의 데이터를 저장할 공간입니다.

여러가지 옵션들이 있지만, 저는 local로 운용할 수 있는 `minio`를 통해 데이터를 저장해 볼 것입니다.

`minio`를 `docker-compose`를 통해 쉽게 배포하도록 할 것입니다.
먼저, `docker-compose`를 설치합니다.

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

그 뒤 `minio`의 `docker-compose.yaml`을 만듭니다.

```
version: '3.7'

services:
  minio:
    image: minio/minio:RELEASE.2020-01-16T22-40-29Z
    volumes:
      - ./data:/data
    ports:
      - "9000:9000"
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
    command: server /data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
```

`docker-compose`를 통해서 deamon으로 실행합니다.

```bash
docker-compose up -d
```

이제 `halyard`와 연동을 하도록 합니다.

먼저, `~/.hal/default/profiles/front50-local.yml` 파일을 다음과 같이 생성합니다.

```yaml
spinnaker:
  s3:
    versioning: false
```

그 다음 다음의 명령어로 연동을 합니다.

```bash
ENDPOINT=http://10.0.2.4:9000
MINIO_ACCESS_KEY=minio
MINIO_SECRET_KEY=minio123

echo $MINIO_SECRET_KEY | hal config storage s3 edit --endpoint $ENDPOINT \
    --access-key-id $MINIO_ACCESS_KEY \
    --secret-access-key

hal config storage edit --type s3
```
---
title: "Install in Air Gaped Environment"
menuTitle: "Install in Air Gaped Environment"
date:  2020-01-20T01:16:52+09:00
weight: 7
draft: false
tags: ["install", "spinnaker", "air-gaped"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

이번에는 인터넷이 되지 않는 환경에서 어떻게 Spinnaker를 설치하는지에 대해 알아보도록 하겠습니다.

먼저 `halyard`에서 언제 인터넷과 통신하는지를 대강 추려보도록 하겠습니다.

1. Spinnaker의 version.yaml을 불러와서 최신의 `halyard` 버전과 최신 Spinnaker의 버전들을 보여줍니다. 
   - gs://halconfig/version.yml
2. 설치하고자 하는 Spinnaker의 버전을 선택하면, 그에 따른 배포에 필요한 yaml들을 불러옵니다.
   - gs://halconfig/bom/VERSION.yml
   - gs://halconfig/MICRO_SERVICE/TAG.yml
3. deploy를 하기 위해 Google Cloud Repository에서 이미지를 가지고 옵니다.
   - gcr.io/spinnaker-marketplace/SERVICE
4. 마지막으로 dependency가 있는 몇가지 서비스를 Google Cloud Repository에서 가지고옵니다. (`consul`, `redis`, `vault`)
   - gcr.io/kubernetes-spinnaker/SERVICE

여기서 local 설정으로 변경이 가능한 것은 2020.01.20 현재 1,2,3번 항목들입니다.
이것들을 어떻게 인터넷이 되지 않는 환경에서 설치가 가능하도록 설정하는지에 대해 알아보겠습니다.

### gsutil로 gs://halconfig 파일들을 로컬에 복사하기

우선 인터넷이 잘 되는 서버가 하나 필요합니다.
이 서버에서 우리는 필요한 BOM(Bill of Materials)를 미리 다운로드 할 것입니다.

`gsutil`이 설치되어 있어야 합니다.

```bash
gsutil -m cp -r gs://halconfig .
```

이렇게하면 로컬에 `halconfig`라는 폴더가 생겼을 것입니다.
이를 인터넷이 안되는 `halyard`가 설치된 서버로 복사합니다.
이때, `halconfig` 폴더 내의 내용들은 `~/.hal/.boms/` 폴더 내에 복사합니다.

```bash
$ ls ~/.hal/.boms/

bom  clouddriver  deck  echo  fiat  front50  gate  igor  kayenta  monitoring-daemon  orca  rosco  versions.yml
```

여기서 `rosco/master` 폴더로 들어가면 `packer.tar.gz`라는 폴더가 있습니다.
이를 `rosco` 폴더로 옮기고 압축을 해제합니다.

```bash
mv ~/.hal/.boms/rosco/master/packer.tar.gz ~/.hal/.boms/rosco
cd ~/.hal/.boms/rosco
tar xvf packer.tar.gz
```

### `halyard`에서 gcs의 `version.yml`이 아닌 로컬의 `version.yml`을 참조하도록 설정

기본적으로 `halyard`는 `gs://halconfig/version.yml`을 참조하려 할 것입니다.
이를 `local:`이라는 접두사를 붙여 로컬을 바라보게 할 수 있습니다.

```bash
hal config version edit --version local:1.17.4
```

그리고 `halyard`가 gcs를 바라보지 않도록 설정합니다.

```yaml
# /opt/spinnaker/config/halyard-local.yml
spinnaker:
  config:
    input:
      gcs:
        enabled: false
```

그러고 난 뒤, 각 서비스들의 BOM도 로컬을 바라보게 설정해야 합니다.
아까 위에서 1.17.4 버전을 사용한다고 했으니, 해당 yaml파일을 열고 `local:` 접두사를 추가합니다.

```yaml
artifactSources:
  debianRepository: https://dl.bintray.com/spinnaker-releases/debians
  dockerRegistry: gcr.io/spinnaker-marketplace
  gitPrefix: https://github.com/spinnaker
  googleImageProject: marketplace-spinnaker-release
dependencies:
  consul:
    version: 0.7.5
  redis:
    version: 2:2.8.4-2
  vault:
    version: 0.7.0
services:
  clouddriver:
    commit: 024b9220a1322f80ed732de9f58aec2768e93d1b
    version: local:6.4.3-20191210131345
  deck:
    commit: 12edf0a7c05f3fab921535723c8a384c1336218b
    version: local:2.13.3-20191210131345
  defaultArtifact: {}
  echo:
    commit: acca50adef83a67e275bcb6aabba1ccdce2ca705
    version: local:2.9.0-20191029172246
  fiat:
    commit: c62d038c2a9531042ff33c5992384184b1370b27
    version: local:1.8.3-20191202102650
  front50:
    commit: 9415a443b0d6bf800ccca8c2764d303eb4d29366
    version: local:0.20.1-20191107034416
  gate:
    commit: a453541b47c745a283712bb240ab392ad7319e8d
    version: local:1.13.0-20191029172246
  igor:
    commit: 37fe1ed0c463bdaa87996a4d4dd81fee2325ec8e
    version: local:1.7.0-20191029183208
  kayenta:
    commit: 5dcec805b7533d0406f1e657a62122f4278d665d
    version: local:0.12.0-20191023142816
  monitoring-daemon:
    commit: 59cbbec589f982864cee45d20c99c32d39c75f7f
    version: local:0.16.0-20191007112816
  monitoring-third-party:
    commit: 59cbbec589f982864cee45d20c99c32d39c75f7f
    version: local:0.16.0-20191007112816
  orca:
    commit: b88f62a1b2b1bdee0f45d7f9491932f9c51371d9
    version: local:2.11.2-20191212093351
  rosco:
    commit: 269dc830cf7ea2ee6c160163e30d6cbd099269c2
    version: local:0.15.1-20191202163249
timestamp: '2019-12-12 14:34:16'
version: 1.17.4
```

이렇게 설정하면 `echo`를 예로 들 때 `~/.hal/.boms/echo/2.9.0-20191029172246/echo.yml`을 참조하게 될 것입니다.

### 배포에 필요한 이미지들을 private registry에 불러오기

이제 실제 배포에 필요한 이미지를 로컬로 복사해두어야 합니다.
저는 내부에서 사용하는 docker registry에다가 저장해 둘 것입니다.
인터넷이 되는 서버에서 다음과 같이 작업하면 됩니다.

```bash
docker pull gcr.io/spinnaker-marketplace/SERVICE:TAG
docker tag gcr.io/spinnaker-marketplace/SERVICE:TAG private-docker-registry/repository-name/SERVICE:TAG
docker push private-docker-registry/repository-name/SERVICE:TAG
```

이렇게 private registry로 저장을 해 두었을 경우 `VERSION.yml` 파일에서 dockerRegistry 항목을 수정합니다.

```yaml
artifactSources:
  debianRepository: https://dl.bintray.com/spinnaker-releases/debians
  #dockerRegistry: gcr.io/spinnaker-marketplace
  dockerRegistry: private-docker-registry/repository-name
  gitPrefix: https://github.com/spinnaker
  googleImageProject: marketplace-spinnaker-release
```

또는 `docker pull`을 이용해서 이미지를 다운받고, 이를 `docker save` 명령어를 통해 `tar.gz` 파일로 변환한 뒤, 
Kubernetes의 모든 워커노드에서 이를 이리 `docker load` 하는 방법도 있습니다.
이렇게 하면 이미 로컬에 있는 이미지이기 때문에 외부로 접속하지 않습니다.

```bash
docker pull gcr.io/spinnaker-marketplace/SERVICE:TAG
docker save -o SERVICE.tar.gz gcr.io/spinnaker-marketplace/SERVICE:TAG

scp SERVICE.tar.gz TARGET_IP:~/path/to/target
ssh TARGET_IP docker load -i ~/path/to/target/SERVICE.tar.gz
```

이번에는 dependency와 관련된 이미지를 불러와야 합니다.
먼저 Image Registry를 변경하기 위해서는 다음과 같이 조치합니다.

1. `~/.hal/default/service-settings/redis.yml`파일을 생성합니다.
2. 다음과 같이 작성합니다.  
   ```yaml
   artifactId: private-docker-registry/repository-name/redis-cluster:v2
   ```

이 다음에는 마찬가지로 image를 pull하고 이를 private docker registry로 push합니다.

```bash
docker pull gcr.io/kubernetes-spinnaker/SERVICE:TAG # redis-cluster:v2
docker tag gcr.io/kubernetes-spinnaker/SERVICE:TAG private-docker-registry/repository-name/SERVICE:TAG
docker push private-docker-registry/repository-name/SERVICE:TAG
```

또는 이미지를 `tar`로 묶어서 복사하는 방법도 있습니다.

```bash
docker pull gcr.io/kubernetes-spinnaker/SERVICE:TAG
docker save -o SERVICE.tar.gz gcr.io/kubernetes-spinnaker/SERVICE:TAG

scp SERVICE.tar.gz TARGET_IP:~/path/to/target
ssh TARGET_IP docker load -i ~/path/to/target/SERVICE.tar.gz
```

Image Registry가 kubernetes-spinnaker로 변경된 것을 주의하시면 됩니다.

### Deploy

여기까지 왔으면 모든 준비작업은 끝났습니다.
이제 배포만 하면 됩니다.

```bash
hal deploy apply
```

## Reference

https://www.spinnaker.io/guides/operator/custom-boms/  
https://github.com/spinnaker/spinnaker/issues/3967#issuecomment-522306893

---
title: "Install Halyard"
menuTitle: "Install Halyard"
date:  2020-01-11T01:41:08+09:00
weight: 2
draft: false
tags: ["spinnaker", "install", "halyard", "proxy"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## halyard란?

`halyard`는 Spinnaker를 배포할 때 사용하는 CLI 툴입니다.

`halyard`는 Spinnaker 관련 설정들의 validation, 배포한 환경 백업, 설정 추가 및 변경에 사용됩니다.

## 설치 방법 선택하기

총 2가지 방법으로 `halyard`를 설치할 수 있습니다.

* Debian/Ubuntu나 macOS에 직접 설치하기
* Docker 사용하기

Spinnaker Docs에서는 실제 Production 환경이라면 직접 설치하는 방법을, 그게 아니라 간단하게 사용하려면 docker를 사용해도 된다고 하고 있습니다.

그리고 한가지의 옵션이 더 있습니다.

* 인터넷이 되지 않는 환경 (프록시나 방화벽 등으로 `halyard`를 통한 설치가 어려운 경우)

이 글을 작성하고 있는 환경은 인터넷이 잘 되는 환경입니다.
그리고 두가지 모두 시도해 보도록 하겠습니다.

### Debian/Ubuntu나 macOS에 직접 설치하기

공식 Docs에서 `halyard`는 다음과 같은 환경에서 동작한다고 말하고 있습니다.

* Ubuntu 14.04, 16.04 or 18.04 (Ubuntu 16.04 requires Spinnaker 1.6.0 or later)
* Debian 8 or 9
* macOS (tested on 10.13 High Sierra only)

이제 직접 설치를 시작해보도록 하겠습니다.

시작하기 전에, `halyard`를 설치하기 위해서는 root 계정이 아닌 계정이 필요합니다. 만일 root만 있다면 spinnaker를 위한 계정을 생성해 줍니다.

```bash
adduser spinnaker
```

위처럼 생성한 계정에 sudoers 권한을 줍니다.

```bash
adduser spinnaker sudo
```

#### 최신 버전의 `halyard 다운로드`

Debian/Ubuntu:

```bash
curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/debian/InstallHalyard.sh
```

macOS:

```bash
curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/macos/InstallHalyard.sh
```

#### 설치
  
```bash
sudo bash InstallHalyard.sh
```

#### 확인
  
```bash
hal -v
```

#### 추가사항

`. ~/.bashrc`를 실행하여 bash completion 활성화

여기서 proxy 환경이라면 `halyard`의 jvm에 proxy 옵션을 추가해주어야 합니다.

`vi /opt/halyard/bin/halyard`을 통해 `halyard`의 jvm 옵션을 추가할 수 있습니다.

```bash
DEFAULT_JVM_OPTS='"-Djava.security.egd=file:/dev/./urandom" "-Dspring.config.additional-location=/opt/spinnaker/config/" "-Dhttps.proxyHost=<proxyHost> -Dhttps.proxyPort=<proxyPort>" "-Dhttp.proxyHost=<proxyHost> -Dhttp.proxyPort=<proxyPort>"'
```

위의 설정에서 다음과 같이 proxy를 추가해줍니다. 그 다음 `halyard`를 재시동합니다.

```bash
hal shutdown
hal config
```

아랫줄의 `hal config`는 의도적으로 `halyard`를 구동시키기 위함입니다.

### docker로 `halyard` 사용하기

다음의 명령어는 공식 docs에서 제공하는 명령어입니다.

```bash
docker run -p 8084:8084 -p 9000:9000 \
    --name halyard --rm \
    -v ~/.hal:/home/spinnaker/.hal \
    -d \
    gcr.io/spinnaker-marketplace/halyard:stable
```

`kubernetes`로 배포하려 할 경우, `kubectl` 명령어에서 사용할 `kubeconfig` 파일이 필요합니다. 이 또한 `-v` 옵션으로 주어야 합니다. 그리고 그 kubeconfig 파일을 읽도록 설정해야 합니다.

```bash
docker run -p 8084:8084 -p 9000:9000 \
    --name halyard --rm \
    -v ~/.hal:/home/spinnaker/.hal \
    -v ~/.kube:/home/spinnaker/.kube \
    -e KUBECONFIG=/home/spinnaker/.kube/config \
    -d \
    gcr.io/spinnaker-marketplace/halyard:stable
```

사실 5번째 줄의 `-e KUBECONFIG=/home/spinnaker/.kube/config`은 없어도 default로 들어가있는 설정입니다.
하지만 혹시나 위에서 `/home/spinnaker/.kube`가 아닌 다른곳을 저장공간으로 둔다면 아래의 설정도 바뀌어야 합니다.

프록시 환경이라면 다음과 같이 JAVA_OPT를 추가해주어야 합니다.

```bash
docker run -p 8084:8084 -p 9000:9000 \
    --name halyard --rm -d \
    -v ~/.hal:/home/spinnaker/.hal \
    -v ~/.kube:/home/spinnaker/.kube \
    -e http_proxy=http://<proxy_host>:<proxy_port> \
    -e https_proxy=https://<proxy_host>:<proxy_port> \
    -e JAVA_OPTS="-Dhttps.proxyHost=<proxy_host> -Dhttps.proxyPort=<proxy_port>" \
    -e KUBECONFIG=/home/spinnaker/.kube/config \
    gcr.io/spinnaker-marketplace/halyard:stable
```

#### 그래도 안된다면..

아마도 관리가 엄격한 네트워크를 사용하고 계실 것이라고 예상됩니다. 저 또한 그랬으니까요.

`halyard`는 설정 및 버전등의 정보를 `bucket` (Google Cloud Storage)로 관리한다고 합니다. 따라서 이곳으로 연결이 되지 않는다면 validation, version list 등의 상황에서 timeout이 날 것입니다.

Spinnaker에서는 이를 확인하기 위해 `gsutil`을 사용하여 `bucket`의 주소인 `gs://halconfig`에 연결할 수 있는지 확인해보라고 합니다.

또는 `curl`을 이용해서도 확인이 가능합니다.

먼저 `gsutil`은 google storage 서비스에 접속하는 CLI 툴입니다. [Docs](https://cloud.google.com/storage/docs/gsutil_install?hl=ko#deb)에서 설치방법을 확인하여 설치할 수 있습니다.

설치가 완료되었다면 gsutil로 접속이 가능한지부터 확인합니다.

```bash
gsutil ls gs://halconfig
```

두번째로 `curl`을 사용하는 방법입니다.

```bash
curl storage.googleapis.com/halconfig
```

결과물들이 나온다면 정상적으로 `bucket`에는 접속이 가능한 것입니다.
`hal config` 명령어가 성공하지 않지만 `bucket`에 접속이 가능한 케이스는 아직 보지 못했습니다.

우선 여기까지 해서 `bucket`에 접속이 불가능하다고 판단이 되면, 인터넷이 없는 환경에서 설치하는 방법을 고려해보아야 합니다.
또는 googleapis.com url로 proxy에서 사이트가 차단되었는지 확인하고, SSL도 해제할 경우 해결될 수도 있습니다.
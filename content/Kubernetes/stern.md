---
title: "Stern을 이용하여 여러 pod의 log를 한번에 확인하기"
date:  2020-02-24T23:28:01+09:00
weight: 10
draft: false
tags: ["kubernetes", "stern", "log"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Kubernetes에서의 trouble shooting

kubernetes 환경에서 어떤 문제가 발생하면 다음과 같은 flow로 확인을 해보면 됩니다.

1. `kubectl get pods -o yaml`로 yaml을 확인하기
2. `kubectl describe pods`로 pod에 대한 설명 확인하기
3. `kubectl describe deployments(statefulset, daemonset)`으로 확인하기
4. `kubectl logs`로 로그 확인하기

보통 kubernetes 리소스의 부족과 같은 kubernetes단의 문제는 1~3을 확인하면 전부 문제점을 찾을 수 있습니다.
그러나 어플리케이션의 직접적인 원인을 알아보기 위해서는 log를 확인해야 합니다.

하지만 `kubectl`의 `logs`에는 한가지 한계점이 있는데, 바로 단일 container에 대해서만 log 확인이 가능하다는 점입니다.

`stern`은 이러한 문제를 해결하는 tool입니다.

## 설치 방법

공식 Github: <https://github.com/wercker/stern>

### Requirements

* `golang`
* `govendor`
* 기타 build에 필요한 dependency는 `govendor`로 관리

### 절차

#### `golang` 설치하기

`go` 언어는 직접 github에서 가져와 build할 수도 있지만, 간단하게 Ubuntu라면 `apt`를, macOS라면 `brew`를 이용하여 설치할 수 있습니다.

```bash
# Ubuntu
sudo apt install golang
# macOS
brew install go
```

#### `govendor` 설치하기

`govendor`는 external package를 import하는데 사용하는 툴입니다.
이를 통해 dependency를 쉽게 import할 수 있습니다.

간단하게 `go` 명령어만으로 설치할 수 있습니다.

```bash
# install
$ go get -u github.com/kardianos/govendor
# test
$ govendor --version
v1.0.9
```
만약 `govendor` 명령어가 되지 않는다면, 보통은 go의 binary 폴더가 `PATH`에 추가되지 않은 것입니다.

`go env`를 통해 `GOPATH`, `GOBIN`을 확인합니다.
`GOBIN`이 비어있을 경우 `$GOPATH/bin`이 `GOBIN`입니다.

정상적으로 binary 파일이 있다면 `PATH`에 추가합니다.
저는 `GOPATH`가 `/root/go`였으므로 다음과 같이 입력하였습니다.
terminal에 입력하는 것은 임시조치이므로 영구적으로 기록하려면 `/etc/profile` 또는 `~/.bashrc`같은 파일에 저장합니다.

```bash
export PATH=$PATH:/root/go/bin
```

#### `stern` 설치하기

`env | grep GOPATH` 명령어를 통해 `GOPATH`가 제대로 적용이 되는지 먼저 확인합니다.
제대로 안되어있을 경우 `export GOPATH=/root/go`처럼 적용해줍니다.
그 다음 다음의 명령어를 입력합니다.

```bash
mkdir -p $GOPATH/src/github.com/wercker
cd $GOPATH/src/github.com/wercker
git clone https://github.com/wercker/stern.git && cd stern
govendor sync
go install
```

#### 확인

```bash
$ stern -v
stern version 1.11.0
```

## 사용법

정확한 사용법은 [이곳](https://github.com/wercker/stern#usage)을 확인하시면 좋습니다.

간단하게 설명하면, `stern`은 `regEx`로 매칭되는 pod나 container의 log를 `tail -f`처럼 보여주는 것입니다.
또한 그 결과는 색깔로 구분지어 보기 쉽습니다.

```bash
stern -n kube-system kube-proxy-*
```
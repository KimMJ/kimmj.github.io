---
title: "Overview"
date: 2020-01-10T01:02:39+09:00
draft: false
chapter: false
weight: 1
tags: ["install", "spinnaker"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Overview of install Spinnaker

어떻게 Spinnaker를 설치 및 배포하는지 알아보도록 하겠습니다.

가장 먼저 최소 사양을 확인해보도록 하겠습니다.

링크 : https://www.spinnaker.io/guides/developer/getting-set-up/#system-requirements

![시스템 최소 사양](/images/Spinnaker/spinnaker-system-requirements.png)

* 램 18 GB
* CPU 4코어
* Ubuntu 14.04, 16.04, 18.04

Spinnaker 자체가 클라우드 환경에만 배포가 가능하기 때문에, 아마도 "전체 클라우드를 합하여 저정도면 된다"를 의미하는 것 같습니다.

설치 방법은 두가지로 나뉩니다.

1. 테스트를 목적으로 `Helm Chart`를 통한 설치
2. 실제로 사용할 목적으로 `halyard`를 통한 설치

저는 여기서 2번 `halyard`를 통한 설치를 해보려고 합니다.

전체적인 프로세스를 먼저 설명드리자면 다음과 같습니다.

1. `halyard` 설치
2. Cloud Provider(클라우드 제공자) 선택
3. 배포 환경 선택
4. Storage Service 선택
5. 배포 및 접속
6. config 백업하기

그리고 저는 다음과 같은 환경에서 테스트를 할 예정입니다.

* Cloud Provider: Kubernetes on-prem (4 VMs)
  * 1 for master (4GB Mem, 1 CPU)
  * 3 for worker (each 8GM Mem, 4 CPU)
* Environment: Distributed installation on Kubernetes
* Storage Service: Minio
* Deploy and Connect: expose by NodePort
* OS : Ubuntu 18.04.2 Server
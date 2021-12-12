---
title: "Private Docker Registry 오픈소스: Harbor란"
date:  2020-10-18T16:52:12+09:00
weight: 10
draft: false
tags: ["harbor", "docker-registry"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Harbor란

`Harbor` 는 올해(20년) 중순 쯤 `CNCF` 의 graduated된 오픈소스 프로젝트로 `Docker Hub` 처럼 이미지를 저장할 수 있는 저장소이다.
`Docker` 는 이미 `docker registry` 라는 이름으로 개인 이미지 저장소를 컨테이너화 하였었다.
`Harbor` 는 내부적으로 이 `docker registry` 를 사용하고 있으며, 여기에 `RBAC`, web page, image scan 같은 편리한 기능을 추가하였다.
외부 서비스를 이용하는 것이 아니라 내부에 이미지 파일을 저장하기 때문에 사내 보안 정책에도 알맞게 사용할 수 있다.
아니면 필요에 따라 `S3`나 `Minio`를 사용할 수 있을듯 하다.

특히 20년 11월부터 Docker Hub의 기본 정책이 image pull rate를 제한하는 것이 되었는데, 사내에서는 보통 forward proxy를 이용하므로
이러한 image pull rate를 초과할 가능성이 매우 높다.
이러한 경우에 Harbor와 같은 private docker registry를 구축하면 큰 도움이 될 것이다.

## 주요 기능

사내에서 사용하며 편리하다고 생각되었던 몇가지 기능들을 소개한다.

* 이미지 복제 (`Harbor` 외부 <-> `Harbor` 내부)
* 이미지 주기적 삭제 (tag retention)
* Project 별 Disk quota 기능
* Project 별 Private/Public 기능
* Webhook 기능 (v1.10 이상에서 가능)
* OCI Support (v2.0 이상에서 가능)
* `docker registry` 의 API 사용 가능 (인증 관련해서만 코딩에 주의)
* `Swagger` 를 통한 API Docs 제공

## 개인적인 생각

다른 `docker registry` 서비스들(예를 들면 `Artifactory` 라던지, `AWS ECR` 이라던지..)을 사용해보진 않았지만, 잘 만들어진 프로젝트라고 생각된다.
업데이트도 자주 이루어 지고 있는 중이고 이슈도 등록하면 금방 확인해주는 듯 하다.
특히나 Graduated 된 상태이니 믿고 사용해도 좋을 정도라고 생각된다.
나는 사내에서 용도별로 여러개의 `Harbor` 를 설치하고 운용중인데, 설치 과정이 매우 쉬워서 새롭게 시작하려는 사람도 테스트를 해보기 용이할 듯 하다.

`docker registry` 를 사용하다보면 당연스럽게도 아쉬운 부분이 GUI에 관한 부분인데 `Harbor` 는 GUI로 많은 것들을 할 수 있어서 정말 편리하다.
이미지간 복제도 GUI로 할 수 있었고(나의 회사에서는 VPN으로 연결된 구간이 있었는데, `Harbor` 가 컨테이너 기반이라서 그런지 VPN을 타고 트래픽이 나가지 못했다.
그래서 결국은 그 구간만 스크립트를 짜서 이미지 복제를 하였다), 기본적인 이미지 삭제나 특정 기간을 지난 이미지 삭제
또는 가장 최근에 Pull된 N개의 이미지만 빼고 삭제하기 등 운용하기에 편리한 기능들도 있다.

내 생각엔 일단, 한 번 써보면 좋을 프로젝트인 듯 하다.

---
title: "Controllers Overview"
menuTitle: "Controllers Overview"
date:  2020-01-30T18:26:04+09:00
weight: 5
draft: false
tags: ["kubernetes", "concepts"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Contents

이 포스트에서는 Kubernetes의 Controller들에 대해서 알아보도록 하겠습니다.
가장 작은 단위인 Container부터, 상위 개념인 Deployment, StatefulSet까지 다루어 보도록 하겠습니다.

* Containers
* Pods
* ReplicaSets
* Deployments
* StatefulSets

## Monolithic vs. Microservice

우선 Monolithic과 Microservice에 대해서 짚고 넘어가도록 하겠습니다.

Monolithic의 개념은 하나의 큰 어플리케이션을 말합니다.
여러 사람이 개발을 하고 나서 하나의 큰 패키지로 빌드하고 이를 배포하죠.
간단한 서비스라면 문제가 발생하지는 않겠지만, 점점 코드의 수가 늘어나고 거대해질 수록 문제점이 생깁니다.
예를 들면 빌드시간이 오래걸린다던지, scale-out을 하기 힘들다던지 하는 문제가 있겠네요.

반면 Microservice는 하나의 큰 어플리케이션을 여러 조각으로 쪼갠 것을 의미합니다.
각 조각(microservice)는 자신의 역할이 있고, 이를 잘 수행하면 됩니다.
전체적인 관점에서 보면 어플리케이션은 microservice들간의 통신으로 행해진다고 보면 될 것 같습니다.

Monolithic에 비해 Microservice는 몇가지 장점들이 있습니다.

* **scale-out이 용이합니다.**  
  Monolithic에 비해 microservice의 단위는 작기 때문에 scale-out 하는데 시간도 오래 걸리지 않고, 간단합니다.
* **빠른 배포가 가능합니다.**  
  Monolithic에서는 사이즈가 커질수록 빌드하는 시간이 점점 길어진다고 설명했었습니다.
  이는 곧 요즘처럼 트렌드라던지 상황이 급변하는 상황에서 약점이 될 수 밖에 없습니다.
  반면 Microservice는 작은 조각이기 때문에 빌드하는 시간이 Monolithic에 비해 뛰어날 수밖에 없습니다.
  그리고 이를 배포하는 시간도 매우 줄어들게 되죠.
* **문제가 발생하였을 때 영향이 적습니다.**  
  쉽게 우리가 가장 잘 알고있는 게임중 하나인 LoL을 가지고 설명해 보도록 하겠습니다.
  만약 LoL을 플레이하고 싶은데, 상점에 에러가 있어서 이용하지 못한다면 어떻게 되나요?
  LoL이 Monolithic이었다면 게임 플레이가 막혀서 엄청난 원성을 샀을 것입니다.
  하지만 LoL또한 Microservice이기 때문에 문제가 발생한 곳만 이용하지 못할 뿐, 나머지 서비스는 정상적으로 이용이 가능합니다.
  따라서 복구하기도, 운영하기도 훨씬 쉽습니다.

이렇게 보면 무조건 Microservice만 해야하는 것처럼 보이기도 합니다.
하지만 세상일이 모두 그렇듯 여기에도 정답은 없습니다.
자신이 개발하고자 하는 어플리케이션의 특성을 잘 파악해서 한가지를 선택하고 개발하는 것이 좋은 방향이 될 것 같습니다.

## Conatiners

이제 본격적인 설명으로 넘어가보도록 하겠습니다.

Docker를 사용해 보았다면 container도 친숙한 개념이 될 것 같습니다.
container는 VirtualBox처럼 가상화를 하지만, OS까지 가상화하는 것이 아니라 host OS 위에서 커널을 공유하는 방식으로 가상화를 합니다.
어려운 말일수도 있지만 간단하게 말하자면 VirtualBox같은 hypervisor에 의한 가상화보다 훨씬 가벼운 방법으로 가상화를 할 수 있다고 생각하시면 됩니다.
여기서 가볍다는 의미는 빠르게 생성/삭제할 수 있고 용량도 작다는 의미입니다.

Kubernetes는 이런 Docker와 같은 container runtime 기반으로 동작합니다.
그리고 각 container는 어플리케이션 내에서 자신의 역할을 수행하는 것들입니다.
Docker를 통해 컨테이너를 동작시키는 것처럼, Kubernetes도 컨테이너를 Docker같은 container runtime의 힘을 빌려 동작시킵니다.

## Pods

Pod는 Kubernetes에서 어플리케이션을 관리하는 단위입니다.

하나의 Pod는 여러개의 Container로 구성될 수 있습니다.
즉, 너무나 밀접하게 동작하고 있는 여러 Container를 하나의 Pod로 묶어 함께 관리하는 것입니다.

이 때, 하나의 Pod 내에 존재하는 모든 Container들은 서로 `localhost`를 통해 통신할 수 있습니다.

Kubernetes에서 Pod는 `언제 죽어도 이상하지 않은 것`으로 취급됩니다.
동시에 `언제 생성되어도 이상하지 않은 것`, `어디에 떠있어도 이상하지 않은 것`이죠.
그 만큼 어플리케이션 개발자는 Pod를 구성할 때 하나의 Pod가 장애가 나는 경우에도 어플리케이션이 제대로 동작할 수 있도록 만들어야 합니다.
언제 없어졌다가 생성될지 모르니까요.

## ReplicaSet

ReplicaSet은 Pod를 생성해주는 녀석입니다.

단일 Pod를 그냥 생성해도 되지만 그렇게 할 경우 Kubernetes가 주는 여러 이점들, 예를 들어 auto healing, auto scaling, rolling update 등을 이용하지 못합니다.
때문에 Pod를 관리해주는 Controller가 필요하게 되는데 이것이 ReplicaSet입니다.

ReplicaSet은 자신이 관리해야하는 Pod의 template을 가지고 있습니다.
그리고 주기적으로 Kubernetes를 주시하며 내가 가지고 있는 template에 대한 Pod가 원하는 숫자만큼 잘 있는지 확인합니다.
부족하다면 Pod를 더 생성하고 너무 많으면 Pod를 삭제합니다.

ReplicaSet은 Pod를 `label`을 기준으로 관리합니다.
자신이 가지고 있는 `matchLabels`와 일치하는 Pod들이 자신과 관련된 Pod라고 인식하는 것이죠.
따라서 `label`을 운영중에 바꾸는 일은 웬만해선 피해야 합니다.
고아가 발생하여 어느 누구도 관리해주지 않는 Pod가 남게 될 수 있기 때문입니다.

또한 ReplicaSet은 자신이 생성한 파드들을 `<ReplicaSet의 이름>-<hash 값>`으로 생성합니다.
따라서 Pod의 이름만 보고도 어떤 ReplicaSet이 생성했는지 알 수 있게되죠.

## Deployments

위에서는 ReplicaSet에 대해서 이야기해 보았습니다.
Deployments는 ReplicaSet보다 상위 개념의 Controller입니다.

만약 Pod의 template을 수정하는 경우가 생긴다면 어떻게 해야 할까요?
예를 들어 cpu 할당량을 바꾼다던지, image를 다른 이미지로 변경한다던지 하는 경우가 발생한다면요.
ReplicaSet만 존재한다면 이런 상황에서 새로운 template을 가지고 ReplicaSet을 생성하고, 기존의 ReplicaSet을 삭제하거나 `replicas: 0`으로 변경하여 ReplicaSet만 남아있고 실제 Pod는 없도록 해야합니다.

그렇다면 새로 만든 template에 오류가 있어서 이전 버전으로 돌아가고 싶다면 어떻게 해야할까요?
이전에 작성했던 ReplicaSet의 `replicas`를 늘리고, 현재 올라가있던 ReplicaSet의 `replicas`를 줄이면 될 것 입니다.
하지만 이는 일일이 기억해야하는 크나큰 단점이 있겠죠.

때문에 ReplicaSet을 관리해주는 Deployment가 필요합니다.
Deployment는 ReplicaSet을 생성하고 이 ReplicaSet이 Pods를 생성하도록 만듭니다.
그리고 자신이 관리하는 ReplicaSet의 `labels`를 자신의 `matchLabels`로 일치시켜 구분합니다.

그렇다면 Pod의 template이 변경되는 상황엔 이번에는 어떻게 적용이 될까요?

Deployment는 새로운 ReplicaSet을 생성하고, 기존 ReplicaSet의 `replicas`를 `0`으로 줄입니다.
그러면서 자신이 생성했던 ReplicaSet들을 `revision`으로 관리하죠.
이렇게 되면 사용자는 `kubectl` 명령어만 가지고 이전 template을 가진 Pod로 변경할 수 있습니다.

또한 Deployments가 ReplicaSet을 생성할 때는 `<Deployment의 이름>-<hash 값>`으로 생성합니다.
위에서 ReplicaSet도 Pod를 생성할 때 `<ReplicaSet의 이름>-<hash 값>`이라고 설명했었습니다.
따라서 Deployments에 의해 만들어진 Pod들은 `<Deployment의 이름>-<ReplicaSet에 대한 hash 값>-<Pod에 대한 hash 값>`과 같은 형태를 띄게 됩니다.

## StatefulSets

StatefulSet은 좀 특이한 녀석입니다.

Pod는 `어디에 떠있어도 이상하지 않은 것`이라고 언급했었습니다.
그런데 StatefulSet은 그렇지 않습니다.
이름에 걸맞게 이전의 상태를 그대로 보존하고 있어야 합니다.
따라서 여러 제약사항들이 생기기도 합니다.

여기에서는 Pod와의 관계만 알아보고 넘어가도록 하겠습니다.

StatefulSets는 Deployments와는 다르게 ReplicaSet을 생성하지 않습니다.
대신 자신이 직접 Pod를 생성합니다.
이 때 Pod의 `label` 속성을 자신의 `matchLabels`과 일치시켜 자신이 관리하고 있는 Pod를 구분합니다.

또한 이름도 hash값을 사용하지 않고 `0`부터 시작하는 숫자를 사용합니다.
즉, `<StatefulSets의 이름>-<0부터 오름차순>`의 이름을 가지는 Pod를 생성합니다.

만약 생성해야하는 Pod의 template에 변화가 있다면 어떻게 해야할까요?

StatefulSets는 Deployments와 다르게 변경할 수 있는 부분에 제약이 있습니다.
이 경우 절대 StatefulSet을 update할 수 없습니다.
따라서 기존 StatefulSet을 삭제하고 다시 `kubectl apply`와 같은 명령어로 새로운 template을 가지고 생성해야합니다.

Rollback에도 제약사항이 있습니다.

Rollback시 StatefulSet은 Pod가 완전히 Ready상태가 되길 기다립니다.
그런데 만약 StatefulSet이 생성한 Pod가 `ImagePullbackOff`같은 에러에 빠지게 된다면, 또는 영원히 `rediness probe`에 의해 Ready 상태가 되지 않는다면 StatefulSet의 rollback은 멈춰버립니다.
이는 Known Issue로 수동으로 해당 Pod를 삭제하는 방법밖에 없습니다.

따라서 웬만하면 설계를 할 때 StatefulSet을 지양하는 것이 Kubernetes의 Design에 더욱 맞는 방향일 것입니다.

## 마치며..

이렇게 Container, Pods, ReplicaSets, Deployments, StatefulSets의 관계를 중심으로 알아보는 시간을 가졌습니다.
혹시나 잘못된 부분이 있다면 언제든지 댓글 또는 메일로 알려주시면 감사하겠습니다.

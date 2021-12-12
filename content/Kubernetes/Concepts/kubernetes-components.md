---
title: "Kubernetes Components"
date:  2021-02-09T17:29:15+09:00
weight: 10
draft: false
tags: ["kubernetes", "components"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

각 `Components` 에 대해 알아보자.

![Components](/images/Kubernetes/components.png)

## Control Plane Component

### ETCD

`partition tolerance`(분할 내성)보다 `consistency`(일관성)에 중점을 둔 db. `ETCD` 는 간단한 unstructured value를 저장하기에 좋다.

`consistency`를 중요하게 여기기 때문에 write의 순서를 엄격하게 규정하여 set value 시 atomic한 update를 제공한다.

client는 특정한 key namespace에 대해 subcription을 하여 변화를 감지할 수 있다.  따라서 어떤 `component`가 ETCD에 write를 할 경우 다른 `component`는 즉각적으로 그 변화에 대응할 수 있다.

### `kube-apiserver`

Kubernetes에서 ETCD와 통신하는 유일한 시스템이다. ETCD로의 접근 시도에 대해 필터링을 한다.

Kubernetes API를 노출하는 Control Plane Component 이다.

### Controller Manager

API Server와 통신을 하는 것.

Controller는 `ReplicaSet` 과 같은것으로, `ReplicaSet`을 예로 들면 `ReplicaSet` 의 resource를 watch하고 그 selector를 기반으로 `Pod` 를 watch한다. 그렇게 해서 `ReplicaSet` 의 desired state가 되도록 `Pod` 를 조작한다.

각 Controller는 논리적으로 개별 프로세스이지만 복잡성을 낮추기 위해 단일 바이너리로 컴파일 되고 단일 프로세스 내에서 실행된다.

- 노드 컨트롤러: 노드가 다운되었을 때 통지와 대응에 관한 책임을 가진다.
- 레플리케이션 컨트롤러: 시스템의 모든 레플리케이션 컨트롤러 오브젝트에 대해 알맞은 수의 파드들을 유지시켜 주는 책임을 가진다.
- 엔드포인트 컨트롤러: 엔드포인트 오브젝트를 채운다(즉, 서비스와 파드를 연결시킨다.)
- 서비스 어카운트 & 토큰 컨트롤러: 새로운 네임스페이스에 대한 기본 계정과 API 접근 토큰을 생성한다.

### `kube-scheduler`

1. Node에 할당되지 않은 Pod 검색 (unbound Pods)
2. Cluster의 현재 state를 확인 (memory에 있는 cache)
3. 여유 공간이 있고 다른 constraint를 만족하는 노드를 선택
4. 파드를 노드에 할당

## Node Plane Component

### `kubelet`

Node에 위치하게 되는 agent이다.

자신이 위치한 Node에 있는 `Pod` 를 watch하고 있어야 할 책임이 있어, 이러한 `Pod` 들이 Running 상태가 되도록 만들어야 한다. 이러한 `Pod` 에 변경점이 생기면 report를 한다.

### `kube-proxy`

Node에서 network rule 들을 관리한다. User space, `iptables`, `IPVS` 모드가 있으며, 모드에 따라 적절하게 `netfilter` 등을 조작한다. 서비스에 `Virtual IP` 를 할당하고, 이 곳으로 `SNAT` 과 `DNAT` 을 통해 패킷이 전달되도록 하는 역할이다.

`iptables` 의 rule을 생성하여 `Service` 로 전달되는 request가 적절한 `Pod` 로 라우팅 되도록 해주는 역할이다.

### Container Runtime

Container의 실행을 담당하는 소프트웨어

## Appendix

### links

- [https://blog.heptio.com/core-kubernetes-jazz-improv-over-orchestration-a7903ea92ca](https://blog.heptio.com/core-kubernetes-jazz-improv-over-orchestration-a7903ea92ca)
- [https://blog.2dal.com/2018/03/28/kubernetes-01-pod/](https://blog.2dal.com/2018/03/28/kubernetes-01-pod/)
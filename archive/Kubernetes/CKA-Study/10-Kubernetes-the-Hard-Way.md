---
title: "10 Kubernetes the Hard Way"
date:  2020-05-12T01:46:22+09:00
weight: 10
draft: false
tags: ["kubernetes", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Design a Kubernetes Cluster

* Purpose
  * Education
    * Minikube
    * Single node cluster with kubeadm/GCP/AWS
  * Development & Testing
    * Multi-node cluster with a Single Master and Multiple workers
    * Setup using kubeadm tool or quick provision on GCP or AWS or AKS
  * Hosting Production Applications 
    * HA Multi node cluster with multiple master nodes
    * 최대 5000노드
    * 클러스터 내에 최대 약 150000 파드
    * 최대 300000개의 컨테이너
    * 노드당 최대 약 100개의 파드
* Cloud or OnPrem
  * on-prem에는 `kubeadm`을 사용하자.
  * GKE for GCP
  * Kops for AWS
  * AKS for Azure
* Storage
  * High Performance - SSD 사용
  * Multiple Concurrent connections - Network based storage
  * Persistent shared volumes for shared access across multiple PODs
  * Node를 disk type에 따라 label을 주입하기
  * Node selector를 사용하여 특정 disk type을 사용하는 노드로 application 할당하기
* Nodes
  * Virtual or Physical Machines
  * 최소 4개의 노드
  * Master vs Worker Nodes
    * Master node도 workload를 호스트할 수 있다.
  * Linux X86_64 Architecture
* Master Nodes
  * 대규모의 Cluster에서는 Master Nodes와 ETCD cluster를 분리해야한다.

## Choosing Kubernetes Infrastructure

* Windows로 호스팅을 하려면 바이너리 파일이 따로 없기 때문에 VM을 이용해서 리눅스를 띄워야 한다.
* 간단하게 로컬로 클러스터를 만들고자 한다면 Minikube를 사용할 수 있다.
  * Single Node
* `kubeadm`으로도 가능하다.
  * Single/Multi Node 가능
* Turnkey Solutions
  * VM을 Provision 해야함.
  * VM을 Configure 해야함.
  * Script를 통해 Cluster를 배포함.
  * 스스로 VM을 관리해야함.
  * AWS에서의 KOPS
* Hosted Solutions
  * Kubernetes-As-A-Service
  * Provider가 VM을 Provision함.
  * Provider가 Kubernetes를 설치함.
  * Provider가 VM을 관리함.
  * GKE
* OpenShift는 on-prem kubernetes platform중 하나이다.
  * 쿠버네티스 위에 설치되어있는 서비스.
  * 쿠버네티스 관리를 위한 추가적인 툴과 GUI를 제공함.
  * CI/CD 파이프라인 제공
* Cloud Foundry Container Runtime은 BOSH라는 오픈소스를 통해 HA cluster를 관리함.

## Configure High Availability

* Master Node가 삭제되면 어떡하나?
  * 일단 계속 동작은 함.
  * 그러다가 파드가 크래시나기 시작하면 문제가 발생.
  * 파드를 재시작해줄 수 없음.
* 따라서 마스터노드를 여러대 두어야 한다.
* Signle Point of Failure를 없애기 위함.
* 마스터노드를 두개 두려면 컴포넌트들 역시 둘 다 배포되어야 한다.
  * ETCD, API Server, Controller Manager, Scheduler
* `API Server`는 Active, Active 모드로 떠있을 수 있다.
  * 기존에는 API server의 6443 포트로 `kubectl` 이 접근했었다.
  * 이를 LoadBalancer를 통해서 트래픽을 분기해주는 것이 필요하다.
  * `NGINX`, `HA proxy`등이 이용가능하다.
* `Scheduler`, `Controller Manager`의 경우
  * 클러스터의 상태를 보고 행동을 취하는 것들이다.
  * 계속해서 파드의 상태를 체크한다.
  * 동시에 동작한다면 실제로 생성해야 하는 파드보다 더 많은 파드를 생성해버릴 수 있다.
  * 따라서 Active/Standby Mode로 동작해야한다.
  * leader election process가 필요하다.
  * `kube-controller-manager` 에서 `--leader-elect` 를 `true` 로 설정해야한다.
  * 이들은 `kube-controller-manager endpoint` 에 lock을 걸로 lease를 한다.
  * 둘다 2초마다 leader가 되고자 한다.
  * 하나가 down되면 두번째 것이 up이 된다.
* ETCD는 쿠버네티스 마스터 노드 안에 위치할 수 있고 이는 Stacked Topology라고 한다.
  * 이렇게 구성했을 때에는 하나가 죽었을 때 Redundancy가 제대로 되지 않는다.
  * 다른 방법은  ETCD를 Control plan node와 분리시키는 것이다.
* ETCD를 다른 노드에 두는 것을 External ETCD Topology라고 한다.
  * 이는 control plane node가 죽더라도 ETCD cluster와 데이터에 영향을 주지 않아 덜 위험하다.
  * 외부 노드가 필요하기 때문에 서버가 더 필요하고 셋업이 더 어렵다.
  * API Server가 유일하게 ETCD를 바라보고 있고, API Server에 ETCD의 주소를 적을 수 있기 때문에 API Server가 제대로 바라보도록 설정해야 한다.

## ETCD in HA

* ETCD는 distributed reliable key-value store이다.
* distributed라는 의미는 동일한 copy를 가진 database를 가지고 있다는 것을 의미한다.
  * ETCD는 어떤 copy도 동시에 접근이 가능함을 보장한다.
* READ는 문제가 없지만 WRITE는 상황이 다르다.
* ETCD는 하나의 instance만 write를 처리할 수 있다.
  * 내부적으로 Leader/Follower를 정의한다. 
  * Follower로가 write 요청을 받으면 이를 Leader로 포워딩하여 WRITE를 처리한다.
* Leader Election은 RAFT 알고리즘을 통해서 한다.
* Quorum 이상의 숫자만큼 write가 되면 정상적으로 쓰기가 되었다고 판단한다.
  * N/2 + 1
  * 따라서 2개의 인스턴스만 있으면 fault tolerance가 제대로 보장되지 않는다.
  * 짝수로 클러스터를 구성하면 네트워크가 나뉘었을 때 클러스터가 망가질 수 있다.
  * 따라서 홀수가 추천되고 5개가 가장 적절하다.

## TLS Bootstrap worker node
 
* `kubelet`의 client는 `kube-apiserver`이다.
* Bootstrap Token을 생성하고 이를 group `system:bootstrappers`에 넣는다.
* `system:node-bootstraper`를 `system:bootstrappers` 그룹에 할당한다.
* `system:certificates.k8s.io:certificatesigningrequests:nodeclient`를 `system:bootstrappers` 그룹에 넣는다.
* `systemLcertificates.k8s.io:certificatessiningrequests:selfnodeclient`를 `system:nodes`에 넣는다.
* kube documentation에 있다.
* 이렇게 하면 워커는 자동으로 TLS certificates를 생성하고 expire 시 renew가 가능하다.
* `kubelet`에 `--rotate-certificates=true`를 넣으면 된다.
* `server certificates`는 보안상의 이유로 수동으로 approve해야 한다.


---
title: "06 Cluster Maintenance"
date:  2020-04-24T13:09:30+09:00
weight: 6
draft: false
tags: ["kubernetes", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## OS Upgrades

* Node가 Down된 상태에서 5분이 지나면 pod는 terminate가 된다.
  * Dead로 간주
* `ReplicaSet`으로 관리되면 다른곳에 파드를 띄워준다.
* `kube-controller-manager`에 `pod-eviction-timeout`이 기본적으로 5분이 설정되어 있다.
* 다시 online으로 오게되면 blank node로 뜨게 된다.
* `ReplicaSet`으로 관리되지 않았던 파드는 삭제되고 재생성되지 않는다.
* 따라서 `kubectl drain node`를 통해 파드를 이주시키고 `cordon`을 통해 스케줄되지 않도록 한다.
* 그 다음에 노드를 down시키고 살리면 된다.
* 산 뒤에도 이전에 처리한 `cordon`이 남아있게 되는데 이를 삭제하기 위해선 `kubectl uncordon`을 해야한다.
* 자동으로 이주되었던 파드가 돌아오지 않는다.
* `cordon`은 `unschedulable`로 처리하는 작업이다.


## Kubernetes Software Versions

* Release Number는 3가지로 구분
  * MAJOR, MINOR, PATCH
  * MINOR 버전은 Feature, Functionalities 위주
  * PATCH는 Bug Fixes 위주
* stable한 버전(MINOR의 업그레이드)는 매달 일어남.
* alpha 버전은 default로 disable되어 있는 feature. 버그 등이 있을 수 있음
* beta 버전은 잘 테스트 되었고 default로 enable되어 있는 feature. stable로 통합됨.
* release package에는 모든 controller component가 패키징되어있음.(동일한 버전으로)
* ETCD cluster나 CoreDNS같은 controller component가 아닌 것은 포함되어있지 않음.
  * supported version을 명시해둠.

## Cluster Upgrade Process

* 웬만하면 Controller Component는 동일한 버전을 가지고 있어야 한다.
* `kube-apiserver`가 `1.x` 버전이면 `controller-manager`와 `kube-scheduler`는 `1.x` ~ `1.x-1` 버전이 될 수 있다.
  * `kubelet`과 `kube-proxy`는 `1.x-2` ~ `1.x`
  * `kubectl`은 `1.x-1` ~ `1.x+1`
* kubernetes는 최근 3버전만 support한다.
* 업그레이드 시 MINOR 버전 하나씩 업그레이드 하는 것을 권장한다.
* Cloud provider로 관리되고 있으면 클릭만 해서 업그레이드가 가능하다.
* `kubeadm`과 같은 툴을 사용할 경우 tool에 내장된 업그레이드 플랜을 사용하면 된다.
* scratch로 구성했을 경우 수작업으로 업그레이드 해야한다.
* 강의에서는 `kubeadm` 사용.
* 클러스터 업그레이드는 두가지 major step으로 나뉨.
  * 마스터 노드 업그레이드
    * 업그레이드 시 Control plane component들 (`apiserver`, `scheduler`, `controller manager`)는 잠시 끊긴다.
    * 그렇다고 해서 워커노드에 영향이 있다는 것을 의미하지는 않는다.
    * 마스터 노드가 다운이 되기 떄문에 클러스터에 접속이 불가능하다.
    * 마스터 노드가 업그레이드 되더라도 워커노드들의 controller component와의 통신이 지원된다.(버전 하나차이)
  * 워커 노드 업그레이드
    * 한번에 업그레이드 하기
      * 유저가 접속할 수 없음.
      * 다운타임 발생
    * 하나씩 업그레이드 하기
      * 다운타임 없음.
    * 새 버전의 노드 새로 추가
      * public cloud 사용할 떄 좋음.
      * 하나씩 추가.
* kubeadm은 upgrade 명령어가 있음.
* `kubeadm upgrade plan`
  * `kubeadm`은 `kubelet`의 업그레이드를 하지 않음.
  * 따라서 스스로 업그레이드 해야함.
  * 또한 `kubeadm`의 업그레이드를 먼저 해야함.
  * `apt-get upgrade` 명령어시 다른 패키지도 업그레이드가 되기 때문에 `apt-get install` 명령어로 단일 패키지만 업그레이드 시킨다.
  * `apt-get install -y kubeadm=<version>`
  * `kubeadm upgrade apply <version>`
  * 이후에도 여전히 `kubectl get nodes`로 조회한 것은 이전 버전이 나온다.
  * 이는 `kubelet`의 버전을 표시하기 때문이다.
  * 설치 방법에 따라 `kubelet`이 설치 되었을수도, 아닐수도 있다.
  * scratch 방식으로 설치했을 떄에는 `kubelet`을 설치하지 않는다.
  * `apt-get install -y kubelet=<version>`
  * `systemctl restart kubelet`
  * `kubectl get nodes`시 업그레이드된 버전이 보임.
  * 워커노드의 업그레이드
    * 워커노드를 업그레이드 하려면 `drain`을 통해 노드를 비워야 한다.
    * `cordon`을 통해 `unschedulable`로 변경
    * `apt-get install -y kubeadm=<version>`
    * `apt-get install -y kubelet=<version>`
    * `kubeadm upgrade node config --kubelet-version <version>`
    * `systemctl restart kubelet`
    * 이를 각 노드를 순회하며 실행한다.
* `apt-mark kubelet kubectl kubeadm`을 통해 자동업그레이드가 안되도록 설정한다.



## Backup and Restore Methods

* Backup Candidates
  * Resource Configuration
  * ETCD Cluster
  * Persistent Volumes
* Resource Configuration
  * `Declarative`방식으로 사용하여 저장할 것을 권장.
  * `yaml`파일로 구성하는 것.
  * 이런 파일을 github등으로 저장할 것.
  * `kube-apiserver`로부터 모든 리소스를 가져와서 백업할 수도 있음.
  * `kubectl get all --all-namespace -o yaml > all-deploy-services.yaml`
  * 그러나 몇가지 리소스는 빠져있음.
  * `VELERO`라는 툴로 모든 리소스를 저장할 수 있음.
* ETCD Cluster
  * cluster의 state를 저장한다.
  * master node에 배포된다.
  * `--data-dir=/var/lib/etcd`의 형태로 모든 데이터를 저장하는 path를 지정한다.
  * Backup
    * ETCD는 `etcdctl`에서 `snapshot` command를 제공한다.
    * backup하기 위해서는 `service kube-apiserver stop`을 해야한다.
      * ETCD cluster를 restart를 하게 되는데 `kube-apiserver`가 이에 대해 의존성을 가지고 있기 때문이다.
  * Restore
    * `etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup ... --initial-cluster-token etcd-cluster-1 ...`
    * 이 정보들은 original file에서 가져와야 한다.
    * 여기서 설정한 `--data-dir`과 `--initial-cluster-token`으로 기존의 `etcd.service`를 채워야 한다.
    * `systemctl daemon-reload`를 통해 설정을 다시 읽는다
    * `systemctl etcd restart`
    * `service kube-apiserver start`
* ETCD cluster에 접근할 수 없으면 `kube-apiserver`를 통한 백업이 가장 나은 방법이다.
* 두 방법 모두 장/단점이 있다.

## Working with ETCDCTL

* 실습에서 ETCD는 master 노드에서 static pod로 v3가 배포되어 있다.
* backup과 restore를 위해서는 `ETCDCTL_API=3`을 설정해야 한다.
* 환경변수로 추가하면 된다.
* `TLS` 기반으로 만들어진 ETCD Cluster이기 떄문에 `--cacert`, `--cert`, `--endpoints=[127.0.0.1:2379]`, `--key`는 필수이다.

## Tips

* 실제 시험 환경에서는 어떤것이 문제인지 알 수 없으니 `kubectl describe pod`를 통해 디버깅을 해야한다.
* <https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster/>
* <https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md/>
* <https://www.youtube.com/watch?v=qRPNuT080Hk/>

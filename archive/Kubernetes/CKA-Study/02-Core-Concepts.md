---
title: "02 Core Concepts"
date:  2020-04-14T00:42:25+09:00
weight: 2
draft: false
tags: ["kubernetes", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---
* Controller는 Kubernetes의 brain과 같다.

## ReplicaSets

* Pod가 죽게 되면 사용자가 접근을 할 수 없게 된다.
* 따라서 여러개의 파드를 띄워 하나가 죽어도 나머지가 동작하도록 해야한다.
* Replication Controller는 여러개의 파드를 띄울 수 있도록 도와준다.
* 이를 High Availability라고 한다.
* 하나의 파드만 관리한다고 해서 쓸모없는게 아니라 이는 하나가 죽으면 다시 하나를 실행시키는 방식으로 동작한다.
* 로드가 늘어나면 파드를 늘릴 수 있다.
* Replica Controller와 Replica Set은 비슷하지만 다르다.
  * Replica Controller는 Replica Set으로 대체되었다.
* Replica Controller
  * `apiVersion=v1`
  * `spec.template`에 파드의 스펙을 정의한다.
    * 파드를 정의할때의 yaml에서 `apiVersion`과 `kind`같은것만 빼고 나머지 정의들을 `spec.template`에 적으면 사용가능하다.
* Replica Set
  * `apiVersion=apps/v1`
  * `selector`를 작성해야한다. -> Replica Controller과의 차이점.
  * 어떤 파드를 컨트롤하는지 적어야한다.
  * label을 적어서 apply시 생성했던 파드가 아니더라도 관리할 수 있도록 한다.

## Deployments

* roll back, rolling upgrade
* 각 파드는 Replica Set으로 관리된다.
  * 이를 Deployment가 감싼다.
  * rolling upgrade, roll back, scale out, pause, resume 등을 사용할 수 있음
* Definition
  * ReplicaSet에서 Deployment로만 변경하면 된다.
  * ReplicaSet과 Deployment는 크게 다르지 않다.
* commands
  
## Certification Tips

* `kubectl run`을 이용하여 yaml template을 생성하라.
* Create an NGINX pod
  * `kubectl run --generator=run-pod/v1 nginx --image=nginx`
* Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
  * `kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml`
* Create a deployment
  * `kubectl create deployment --image=nginx nginx`
* Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
  * `kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml`
  * Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment
  * 즉, deployment를 create할 때는 먼저 생성해놓고 replicas를 조정해야한다. (`--replicas` 옵션이 없음)

## Namespaces

* 각 네임스페이스에는 그 안에서 소비할 수 있는 리소스들이 있다.
* `default` 네임스페이스는 쿠버네티스가 처음 만들어질 때 생성되는 것.
* 쿠버네티스는 네트워킹이나 DNS와 같은 내부적인 목적으로 파드와 서비스를 만든다.
* 이러한 것들을 유저가 사용하는 공간과 다르게 두어 실수로 삭제하지 않도록 만들어준다.
  * `kube-system`이다.
* `kube-public`은 모든 유저가 사용할 수 있는 리소스들이 있다.
* 작은 공간에서 사용할 경우 `default`만 써도 되지만 큰 기업으로 가면 네임스페이스를 고려해야 한다.
* 클러스터 내에서 `dev`와 `prod`를 다른 네임스페이스를 두고 만들수도 있다.
* 각 네임스페이스는 누가 무엇을 할 수 있는지에 관한 정책을 가지고 있다.
* 또한 네임스페이스별로 리소스 쿼터를 설정하여 최소한의 서비스를 할 수 있도록 한다.
* 네임스페이스에 내에 있는 리소스끼리는 이름만 가지고 접근이 가능하다.
* 다른 네임스페이스에 있는 것에 접근하려면 네임스페이스를 알려주어야 한다.
  * `servicename.namespace.svc.cluster.local`
* 이러한 형태로 dns가 설정되기 때문.
* `cluster.local`: domain
* `svc`: subdomain, service
* yaml이 특정 네임스페이스에만 뜰 수 있도록 하고싶다면 이를 manifest에 옮기면 된다.
* 기본 네임스페이스를 변경하고 싶으면 
  `kubectl config set-context $(kubectl config current-context) --namespace=dev`
  와 같은 방식으로 조정
* 리소스 쿼터는 `kind=ResourceQuota`로 설정하면 된다.


## Service

* 내 외부의 다양한 컴포넌트와 통신을 가능하게 해준다.
* 어플리케이션을 loose-coupling하게 해준다.
* 클러스터 내부의 파드와 통신하는 방법
  * ssh로 클러스터 노드에 접속 후 curl을 통해 파드의 아이피로 직접 통신 가능
  * ssh 없이 통신을 하기 위해서는 노드 안에서 포워딩을 도와주는 무언가가 필요
    * 서비스는 노드의 포트를 listen하고 이를 파드의 포트로 포워딩한다.
    * 이 방식은 `NodePort`라고 알려진 것.
  * `ClusterIP`
    * 클러스터 내부에 virtual IP를 만들어 서로 통신하게 만듬.
  * `LoadBalancer`
    * cloud provider 내부에서 서비스에 대한 로드밸런싱을 사용하게 해줌.
* `[30008]node - service [80] - [80]pod` 형태일 때
  * `targetPort`로 파드가 떠있는 80포트를 지정.
    * 서비스가 요청을 포워딩할 곳.
  * `port`는 서비스 자체의 포트.
    * 서비스 관점에서의 포트
    * 노드 내에서 virtual server처럼 작동.
    * 자신의 IP를 가지고 있고, 이를 cluster ip라고 부름.
  * `nodePort`는 노드에서 외부로 expose하는 포트
  * 기본적으로 노드포트 범위는 30000~32767
* `spec`에는 `type`과 `port`를 작성.
* 포트에서 필수 필드는 `port` 하나.
  * targetPort는 안적으면 port와 동일
  * nodePort는 안적으면 범위내에서 랜덤 생성
* `label`과 `selector`로 파드를 연결한다.
* 서비스와 멀티 파드가 연결되어 있으면 랜덤 알고리즘으로 로드밸런싱을 한다.
* 모든 클러스터의 노드로 노드포트를 expose한다.

## Service Cluster IP

* service들간에 통신을 하는 올바른 방법은 서비스의 아이피를 이용하는 것이다.
  * 파드의 아이피는 변경될 수 있기 때문에.
* 랜덤으로 로드를 분배한다.
* 이를 클러스터 아이피라고 한다.
* `targetPort`는 파드가 expose하는 포트
* `port`는 서비스가 사용하는 포트

## Certification Tips

* 파일을 직접 만드는 것은 어렵기 때문에 imperative command를 사용해서 파일템플릿을 생성하는 것이 좋다.
* `--dry-run`과 `-o yaml`을 자주 사용하면 좋다.
* 파드
  * `kubectl run --generator=run-pod/v1 nginx --image=nginx`
  * `kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml`
* Deployment
  * `kubectl create deployment --image=nginx nginx`
  * `kubectl run --generator=deployment/v1beta1 nginx-image=nginx --dry-run --replicas=4 -o yaml`
* Important
  * `kubectl create deployment`는 `--replicas` 옵션이 없다.
    * `kubectl scale` 명령어로 크기를 조정해야한다.
  * `kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml`
  * `kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml`
* Service
  * `kubectl expose pod redis --port 6379 --name redis-service --dry-run -o yaml`
    * 자동으로 파드와 매칭되는 라벨을 사용하게 됨.
  * `kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml`
    * `app=redis`라는 셀렉터가 있다고 가정함. 따라서 배포 전 selectordp rhdp
  * `kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml`
    * 수동으로 nodeport 설정 넣어줘야 함.
  * `kubectl create service nodePort nginx --tcp=80:80 --node-port=30080 --dry-run all`
    * 셀렉터 지정 필요
* `kubectl expose`를 더 추천.

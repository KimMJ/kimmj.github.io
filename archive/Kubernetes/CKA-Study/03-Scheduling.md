---
title: "03 Scheduling"
date:  2020-04-15T18:55:03+09:00
weight: 3
draft: false
tags: ["kubernetes", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Manual Scheduling

* how scheduling works
  * `podSpec`에 `nodeName` 필드를 채워넣으면 해당 노드로 파드가 뜬다. default로는 비워져있음.
  * 스케줄링은 알고리즘에 의해 파드를 띄울 노드를 선택하고 나면 `nodeName` 필드를 채운다.
  * 스케줄러가 없으면 파드가 계속해서 pending 상태에 있게 된다.
  * 따라서 스케줄러가 없으면 단순히 `nodeName`을 채우면 될 것이다.
  * runtime에 `nodeName`을 변경할 수 있는데, 이는 `kind=Binding` object를 binding API로 POST 요청을 보내는 방식으로 가능하다.
  * binding definition에 `target.name=<NodeName>`으로 설정하면 된다.
  * 이를 JSON 형식으로 보내면 된다.
    * 따라서 yaml을 json으로 변경해야한다.

## Labels and Selectors

* 각 리소스에 대해 label로 속성을 부여하고 selector로 선택할 수 있다.
* `metadata.labels`에 key-value 형태로 제공
* ReplicaSet을 예로 들 때, `spec.template.metadata.labels`에 있는 정보는 파드에 대한 label이다.
  * `metadata.labels`에 있는 레이블은 ReplicaSet에 대한 label이다.
  * `spec.selector.matchLabels`은 생성할 파드와 연결해주는 것이다.
* label와 selector는 object를 그룹화하고 선택할 때 사용된다.
  * annotations는 inflammatory purpose를 위해 기록되는 것이다.
  * integration purpose로도 사용됨.

## Taints and Tolerations

* 사람이 벌레를 퇴치하는 것에 비유
  * 벌레가 싫어하는 스프레이를 사람에게 뿌리면 사람은 taint 처리가 됨.
  * 벌레는 이를 견디지 못해서 사람에게 갈 수 없다. (intolerant)
  * 만약 이 스프레이에 내성이 있는 벌레가 있다면 사람에게 다가갈 수 있을 것이다. (tolerant)
* 즉, 사람에게 taint 처리를 하고 벌레에게 해당 taint에 대해 tolerant 속성을 주면 벌레가 사람에게 다가갈 수 있다.
* Taints: `kubectl taint nodes node-name key=value:taint-effect`
  * `taint-effect`에는 `NoSchedule`,`PreferNoSchedule`,`NoExecute`가 있다.
  * `kubectl taint nodes node1 app=blue:NoSchedule`
* podSpec에서 다음과 같이 설정
  
  ```yaml
  spec:
    tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
  ```

* taint, toleration 설정을 했다고 해서 파드가 항상 taint쪽으로 가는 것은 아니다.
* toleration이 없는 파드들이 taint 처리된 노드로 뜨지 못할 뿐이다.
* 파드를 특정 노드로 할당하고 싶다면 이는 node affinity를 사용해야 한다.
* `kubectl describe node node-name | grep Taint`

## Node Selectors

* 특정한 노드로 파드가 뜨게 하는 방법에는 여러개가 있다.
  * `nodeSelector` 사용하기
    * `nodeSelector`는 `key=value`를 사용하고, 이는 노드에 할당된 label이다.
    * 제약사항: equal 상황에서만 사용할 수 있음

## Node Affinity

* `spec.affinity.nodeAffinity`에 지정
  * `requiredDuringSchedulingIgnoreDuringExecution`
    * `nodeSelectorTerms`
      * `matchExpressions`
        * `In`을 사용하면 key값에 대해 복수개의 value중 만족하는 것이 있는 경우에 적용할 수 있다.
        * `NotIn`을 사용하여 key=value를 만족하지 않는 것에 대해 적용할 수 있다.
        * `Exists`를 사용하여 key값이 있는지 확인할 수 있다. value가 필요 없음.
* `requiredDuringSchedulingIgnoreDuringExecution`과 `preferredDuringSchedulingIgnoreDuringExecution`이 있음.
* `required`는 스케쥴링 시 없으면 기다림. `preferred`는 없으면 딴데 띄움.
* `DuringExecution`의 경우 `Ignore`만 있으며 노드의 label이 삭제되더라도 evict 하지 않음.

## Taints and Tolerations vs Node Affinity

* Taints & Tolerations만 설정할 경우 뜨기 희망하지 않는 다른 노드에 뜰 수도 있다.
* Node Affinity만 설정하면 다른 파드가 나의 노드에 뜰 수도 있다.
* 따라서 둘 다 사용해야 우리가 원하는 어떤 파드만 어떤 노드에 뜰 수 있도록 하는 상황을 이용할 수 있을 것이다.

## Resource Requirements and Limits

* 노드에 자원이 충요하지 않으면 pending이 된다.
* Resource Request: Minimum Requirement
* 1 CPU라는 의미는 1vCPU와 같다.
  * AWS에서는 1vCPU, GCP와 Azure에서는 1Core, 또는 1 Hyperthread.
  * Mi 처럼 i가 들어가는 것들은 2^10 단위, M만 들어가는 것은 10^3 단위
* 도커는 원래 노드의 cpu를 모두 사용할 수 있을 정도로 cpu 사용량에 제한이 없다.
* default로 쿠버네티스는 컨테이너에 대해 1vCPU 제약을 둔다.
* 메모리 또한 같다. 512Mi가 default
* vCPU의 경우 쿠버네티스가 CPU 사용량에 제한을 둘 수 있다. 하지만 Memory의 경우 그럴 수 없기 떄문에 초과하면 죽인다.
* 기본으로 Request와 Limit을 설정하는 것들은 `LimitRange`를 설정하면 된다.

## Edit POD and Deployments

* 이미 있는 파드에 대해 다음의 필드 이외에 수정이 불가능.
  * spec.containers[*].image
  * spec.initContainers[*].image
  * spec.activeDeadlineSeconds
  * spec.tolerations
* Deployment 안에 있는 파드스펙은 수정이 모두 수정이 가능하며 자동으로 파드를 삭제하고 새로 띄운다.

## DaemonSets

* DaemonSets은 ReplicaSet과 유사.
* 각 노드당 하나의 파드를 띄울 수 있도록 한다.
* 클러스터의 모든 노드에 최소 하나의 파드가 떠있는 것을 보장한다.
* 대표적으로 kube-proxy가 있다.
* ReplicaSet과 작성방법이 유사.
* 작동 원리
  * 파드를 생성하면서 각 노드에 해당하는 nodeName을 설정한다.
  * 1.12까지 이런식으로 사용되었다.
  * 1.12부터 default schedular와 node affinity를 사용하도록 변경되었다. 

## Static Pods

* `kube-apiserver`, `kube-scheduler`, `controller`, `ETCD` cluster가 없다면?
* master 조차 없다면?
* 하나의 worker node만 있을 때 kubelet이 할 수 있는 것이 있을까?
* `kubelet`은 노드를 독립적으로 관리할 수 있다.
* `kubelet`이 아는것은 파드를 생성하는 것 뿐이다.
* `kube-apiserver` 없이 `kubelet`에게 어떻게 podSpec을 전달할 수 있을까?
* 노드의 `/etc/kubernetes/manifests`에 파드 yaml을 넣으면 된다.
  * `kubelet`은 이 디렉토리를 주기적으로 감지하여 파드를 생성한다.
  * 또한 파드가 살아있는지 확인한다.
  * 파드가 죽으면 재시동한다.
  * 파일에 변경사항이 있으면 쿠버네티스는 파드를 다시 생성한다.
  * 디렉토리에서 파일을 삭제하면 파드도 삭제된다.
  * 즉, API Server에 의해 관리되지 않는 파드이다.
  * 이를 Static POD라고 한다.
  * 이 방법으로 파드만 생성해야한다.
  * replicaset이나 deplpoyment, service는 생성할 수 없음.
  * `kubelet`은 파드만 관리할 수 있어서 파드만 생성가능하다.
  * `--pod-manifest-path`로 `kubelet`을 시작할 때 인자로 넣어주어야 한다.
  * `--config` 옵션으로 `yaml` 파일을 넣을 수 있다.
    * 그 안에서 `staticPodPath`를 통해 디렉토리를 넣을 수 있다.
* 다른 워커/마스터가 있더라도 static pod 사용 가능.
  * `kubectl`로도 볼 수 있음.
  * `kubelet`은 클러스터일 때 static pod에 대한 것을 `kube-apiserver`에도 미러링한다.
  * 단, `kube-apiserver`는 read only이다.
  * 수정 또는 삭제는 불가능하며 노드의 manifest folder를 수정해야만 한다.
  * 파드의 이름은 뒤에 자동으로 노드이름이 추가된다.
* static pod는 kuberentes control plane에 의존하지 않음.
* 따라서 control plane 그 자체를 배포할 때 사용한다.
* control plane들을 pod manifest path에 넣으면 `kubelet`이 알아서 생성해준다.
* 이것이 `kubeadm` 툴이 클러스터를 생성하는 방법이다.
* DaemonSet은 클러스터의 모든 노드에 파드를 띄우는 방법
  * control plane의 개입이 있음.
* Static pod는 `kubelet`이 자체생성하는 것

## Multiple Schedulers

* 쿠버네티스는 여러개의 스케줄러를 동시에 가지고 있을 수 있다.
* 파드나 Deployment를 생성할 때 쿠버네티스에게 특정 스케줄러를 사용하도록 지정할 수 있다.
* `kube-scheduler` binary를 다운로드하고 이를 옵션을 통해 서비스로서 동작하도록 실행한다.
* `kube-scheduler`는 스케줄러 이름을 결정하는 것이고 `default-scheduler`가 default이다.
* `kubeadm`은 `kube-scheduler`를 파드 형태로 배포한다.
  * manifest 폴더에서 확인 가능.
  * 여기서 파일을 복사하고 `--scheduler-name`을 내 스케줄러의 이름으로 변경하면 됨.
* multimaster 상황에서 HA 구성이 되어있을 경우 `--lock-object-name`을 통해 leader-election을 조정한다?
* `podSpec`에서 `shedulerName`에 scheduler의 이름을 명시하면 해당 스케줄러를 사용한다.
* `kubectl get events`를 통해 어떤 scheduler가 사용되었는지 알 수 있다.
* scheduler의 로그는 스케줄러 파드의 로그 확인한다.

## Tips

* ```bash
  cd /etc/systemd/system/kubelet.service.d/
  cat 10-kubeadm.conf
  ```

  여기에서 KUBELET_CONFIG_ARGS를 확인할 수 있다.

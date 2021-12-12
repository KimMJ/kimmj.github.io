---
title: "05 Application Lifecycle Management"
date:  2020-04-19T17:02:09+09:00
weight: 5
draft: false
tags: ["kubernetes", "cks"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Rolling Updates and Rollbacks

* `kubectl rollout status deployment/myapp-deployment`로 롤아웃 상태 확인 가능
* `kubectl rollout history deployment/myapp-deployment`로 히스토리 확인 가능
* `Recreate`: old를 모두 죽인 뒤 new를 생성
* `Rolling Update`: old를 하나씩 죽이고 하나씩 new를 생성
* `kubectl set image deployment myapp-deployment nginx=nginx:1.9.1`로 이미지 수정 가능
* `StrategyType`이 `Recreate`면 old의 replica가 모두 0으로 줄고난 뒤 new의 replica를 늘린다.
* 반면 `RollingUpdate`일 경우 old를 하나씩 죽이고 new를 하나씩 늘린다.
* `Rollback`을 할때는 반대로 동작한다.
* `kubectl run`을 통해 파드를 생성하면 `deployment`가 생성된다.

## Commands

* CKA에는 굳이 필요 없을 수 있다..
* container는 OS를 구동하려고 만들어진 것이 아니라 특정한 태스크를 실행하기 위해 만들어졌다.
  * 앱이 crash가 나면 컨테이너는 종료된다.
* Dockerfile을 보면 `CMD`로 어떤 프로세스를 시작할지 결정한다.
* 도커는 기본적으로 컨테이너를 시작할 때 터미널 연결을 하지 않는다. 따라서 `CMD`가 bash일 경우 종료된다.
* `CMD`에 입력할 때에는 executable command와 parameter를 하나의 리스트안에 담으면 안된다.
  * `CMD["sleep", "5"]` vs `CMD["sleep 5"]`
  * `CMD`는 overwrite되는 값이다.
* `ENTRYPOINT`는 `CMD`와 미슷하지만, 도커 실행시 뒤에 적는 추가적인 글자들은 entrypoint의 parameter가 되어 들어간다.
  * appending을 하지 않으면? `sleep`만 전달된다.
  * 따라서 default를 넣고 싶다면 `ENTRYPOINT`와 `CMD`를 함께 쓴다.
  * 아예 덮어쓰고 싶다면 `docker run --entrypoint ...` 형식으로 entrypoint를 지정한다.

## Commands and Arguments

* 파드에 arguments를 넣고 싶으면 `pod.spec.containers[].args`에 넣으면 된다.
* Dockerfile에서 `CMD`는 default parameter이다.
* `args`는 Dockerfile에서 `CMD`를 변경한다.
* `ENTRYPOINT`를 수정하고 싶다면 `command`를 수정하면 된다.

## Configure Environment Variables in Applications

* 환경변수를 넣으려면 `env`에 넣으면 되고 이는 array이다.
* key-value pair로 넣을수도 있지만 ConfigMap이나 Secrets에서 value를 가져올 수도 있다.
  * `valueFrom`으로 가져온다.


## Configuring ConfigMaps in Applications

* `ConfigMap`은 key-value pair data를 저장하는 리소스이다.
* 파드가 생성되면 ConfigMap 안에 있는 정보들이 environment로 들어가서 파드 내에서 사용할 수 있게 된다.
* 사용법은 먼저 생성하고 이를 파드 내에 inject하는 것이다.
* 간단 생성: `kubectl create configmap <config-name> --from-literal=<key>=<value>`
* 파일에서: `kubectl create configmap <config-name> --from-file=<path-to-file>`
* 환경변수 전체를 가져올때 사용가능, 또는 단일 value만을 가져올 수도 있음
* 아니면 volumes에서 configMap에 있는 데이터를 통해 volume mount도 가능.

## Configure Secrets in Applications

* app 내에서 사용하는 값들을 `configMap`으로 관리할 수 있다.
* 여기서 공개하기 어려운 값들이 있을 수 있는데 이를 `Secret`으로 관리한다.
* 이게 Environment Value로 들어간다.
* 사용법: 생성하고 파드에 inject
* `kubectl create secret generic <secret-name> --from-literal=<key>=<value>`
* `kubectl create -f `로도 생성 가능.
  * key-value pair
  * 그러나 그냥 저장하면 보안 취약.
  * encode할 것.
  * `kubectl describe secrets`로 조회되지 않음.
  * `kubectl get secrets -o yaml`으로 조회해야 함.
  * hash value로 나옴. (`base64`)
* 환경변수, 단일 ENV, Volume으로 마운트도 가능.
* 각 Key값에 해당하는 파일이 생성되며 그 내용이 value값이다.
* 그러나 secret은 그 자체로 보안이 뛰어난 것이 아니다.
* 보호하는 방법
  * secret은 vcs로 관리하지 않기
  * ETCD에 암호화해서 저장하기
* 참조: 
  * <https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/>
  * <https://blog.outsider.ne.kr/1266/>

## Multi Container PODs

* micro service로 변경하면서 작은 단위로 코드를 관리하고 배포할 수 있게 되었다.
* scale up, scale down을 각 마이크로 서비스 단위로 할 수 있다.
* 그 중에서 두 pair가 반드시 함께해야하는 경우가 있을 수 있다.
* 개발은 따로 하지만 항상 짝을 이뤄야 하는 경우 multi container를 사용하면 된다.
* 같은 lifecycle을 가짐.
* localhost로 통신 가능.
* 같은 storage volume에 접근.
* 따라서 volume sharing이나 파드간 서비스를 설정하지 않아도 된다.

## Multi-container PODs Design Patterns

* logging같은 것을 하기 위해 sidecar pattern으로 multicontainer 사용하기
* adpater 패턴
* ambassador 패턴
* 이 내용은 CKAD에서.

## initContainers

* 파드 내의 컨테이너들은 함께 죽고 산다.
* 이 중 하나라도 fail이 되면 파드는 재시작된다.
* 파드가 생성되고나서 한번만 실행하고 싶은 동작들은 initContainers로 관리할 수 있다.
* 다른 컨테이너처럼 `initContainers` 안에서 설정이 가능하다.
* 여러개의 initContainer가 있으면 순서대로 실행하고, 순서대로 성공해야한다.
* 하나의 initContainer라도 실패하게 되면 파드를 계속해서 재시작한다.

## Tips

* readiness, liveness는 CKAD에서

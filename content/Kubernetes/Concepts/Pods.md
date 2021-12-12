---
title: "Pods"
menuTitle: "Pods"
date:  2020-02-03T14:03:50+09:00
weight: 10
draft: false
tags: ["kubernetes", "pod"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Pod Overview

### Pod의 이해

Pod는 Kubernetes에서 가장 작은 배포 오브젝트이며 쿠버네티스에서 관리하는 최소 관리 단위입니다.
Pod는 cluster 안에서 실행중인 어떤 프로세스를 의미합니다.
application container, 스토리지 리소스, 유일한 network ip, container가 어떻게 실행할지를 캡슐화한 것입니다.

각각의 Pod는 주어진 application에서 단일 인스턴스를 수행합니다.
즉, 한가지 역할을 맡고 있다고 생각하시면 됩니다..
따라서 application을 수직확장하고 싶다면 각 인스턴스에 대해 여러 Pod를 생성하면 된다.
그러면 동일한 역할을 하는 Pod가 늘어나니, 병렬적으로 처리가 가능할 것입니다.

Pod는 서비스 중에서 서로 연관성이 높은 프로세스를 지원하기 위해 디자인되었습니다.
container는 리소스와 의존성들을 공유하고, 서로 통신하며, 언제/어떻게 종료하는지에 대해 서로 조정합니다.
Pod 내의 container는 Networking과 Storage를 공유할 수 있습니다.

Pod 안에는 둘 이상의 container가 있을 수 있습니다.
이 때 Pod 내에 여러 container를 두는 것은 container가 정말 강하게 결합될 때 입니다.
예를 들어, 하나의 container는 web server로 shared volume에서 파일을 가져와 호스팅하는 역할을 하고, 나머지 하나의 side-car container는 외부에서 file을 pulling하여 shared volume에 올리는 역할을 하는 것입니다.
이처럼 둘간의 결합성이 큰 경우(여기서는 shared volume일 것입니다) 동일한 Pod에 위치할 수 있습니다.

init container는 실제 app container가 시작하기 전에 먼저 작업을 하는 container입니다.

#### Networking
각 Pod 단위로 네트워크 IP 주소를 할당받습니다.
그 내부의 container끼리는 localhost로 통신하게 되고 외부와의 통신을 위해선 공유중인 네트워크 리소스를 어떻게 분배할 지 합의 및 조정해야합니다.
예를 들어 Pod 단위로 생각해보면 하나의 IP를 가지게 되고 expose할 port들을 가지게 될 것입니다.
IP는 상관 없지만 port의 경우 특정 container로 binding이 되어야 하기 때문에 각 container는 동일한 port를 expose할 수 없습니다.
Pod 관점에서 보면 어느 container로 전달해주어야 하는지 모르기 때문이죠.

#### Storage
Pod는 공유하는 저장공간을 volumes로 지정합니다.
이렇게 하면 하나의 container만 재시동되는 경우에도 데이터를 보존할 수 있습니다.

### Pod로 작업하기

Pod를 사용할 때에는 `kubectl create pods`처럼 controller 없이 Pod를 생성하는 것은 좋은 생각이 아닙니다.
이렇게 할 경우 Kubernetes의 장점들을 충분히 활용할 수 없습니다.
특히 self-healing을 하지 못하기 때문에 Pod가 떠있던 노드에 장애가 발생하면 Pod는 영영 복구되지 않을 수 있습니다.
반면 controller로 관리할 경우 self-healing을 지원하여 노드에 장애가 발생해도 다른 노드에 해당 Pod를 띄워서 계속하여 서비스를 할 수 있습니다.

Pod의 재시작과 container의 재시작을 혼동하면 안됩니다.
Pod는 그 자체만으로 동작하지 않습니다.
오히려 Pod는 삭제되기 전까지 계속 유지가 되는, container가 동작하는 환경이라고 보면 됩니다.
Pod를 VM을 사용하는 상황과 비유를 해보자면 Pod는 VM에서 하나의 VM에 실행하는 application들처럼 하나의 논리적 호스트에서 container들을 실행하는 개념입니다.

Pod 내의 container들은 IP주소와 port space를 공유합니다.
그리고 서로 localhost로 통신할 수 있습니다.
반면에 다른 Pod에 있는 container와는 Pod에 할당된 IP를 사용하여 통신할 수 있습니다.

Pod는 container처럼 임시적인 자원이기 때문에 삭제시 reschedule이 아닌 새로운 동일한 spec의 Pod를 새로운 UID(Unique ID)로 생성합니다.
즉, Pod가 삭제되면 그 안에 있는 내용들을 잃어버리게 됩니다.
VM에 빗대자면 VM을 생성하는 template을 가지고 새로운 VM을 생성하며, 삭제할 경우 해당 VM을 완전히 삭제한다는 개념입니다.
이 때 volume도 Pod가 삭제되면 삭제됩니다.

## Pod Lifecycle

Pod의 `status` 필드는 PodStatus의 `phase` 필드입니다.
Pod의 phase는 간단하게 Pod가 위치한 lifecycle의 상위 개념에서의 요약정보입니다.

`phase` 의 value들에는 다음과 같은 것들이 있습니다.

* `Pending` : Pod가 kubernetes 시스템에 의해 받아들여졌지만 하나 이상의 container image가 생성되지 않은 상태.
* `Running` : Pod가 node에 바운드되고 모든 container들이 생성됨. 최소한 하나의 container가 실행 중이거나 시작 또는 재시작 중.
* `Succeeded` : 모든 container가 성공적으로 종료되었고, 재시작되지 않음.
* `Failed` : 모든 container가 정료되었고, 최소 하나의 container가 failure 상태.
* `Unknown` : 어떤 이유로 인해 Pod의 state를 얻어낼 수 없음. 보통 Pod의 호스트와 통신이 안되는 문제.

### Container probe

`kubelet`이 container을 진단할 때 사용하는 것입니다.
container에서 구현된 Handler를 호출하여 이러한 진단을 수행합니다.

크게 3가지 probe가 있습니다.

* `livenessProbe`: container가 실행 중인지 나타냄. liveness probe가 실패하면 `kubelet`은 container를 죽이고, 이 container는 restart policy를 실행한다.
* `redinessProbe`: container가 service requests를 받을 준비가 되었는지 나타냄. readiness probe가 실패할 경우 endpoint controller는 Pod의 IP 주소를 Pod와 매칭되는 Service들의 endpoints에서 삭제한다. initial delay 이전의 default 값은 `Failure`이다.
* `startupProbe`: container 내부의 application이 시작되었는지를 나타냄. 다른 probe들은 startup probe가 성공할 때까지 비활성화 상태.

#### `livenessProbe`를 사용해야하는 상황

`livenessProbe`는 container가 제대로 동작하지 않은 경우 제대로 동작할 수 있을 때까지 재시동하는 목적으로 사용합니다.
따라서 container가 제대로 동작하지 않을 때 실행하는 프로세스가 이미 있다면 굳이 사용하지 않아도 됩니다.
원래 목적대로 재시동을 하고 싶다면 `livenessProbe`를 지정하고 `restartPolicy`를 설정합니다.
 
#### `readinessProbe`를 사용해야하는 상황

Pod에 request를 보내면 트래픽은 그 내부의 container로 전달됩니다.
이 때, 해당 container가 제대로 동작을 하지 않는다면, 파드에 request를 보냈을 때 비정상적인 응답을 할 것입니다.

따라서 Pod가 제대로 응답을 보내줄 수 있는 상황에만 해당 Pod로 트래픽을 전달하고 싶다면, `readinessProbe`를 사용합니다.
Kubernetes는 `readinessProbe`가 실패하면 Service와 연결된 Endpoint에서 해당 Pod를 삭제합니다.
그러면 Service로 흐른 트래픽은 `redinessProbe`가 실패한 Pod로 흐르지 않게 됩니다.

단순히 삭제시 트래픽이 안흐르도록 하고 싶다면 굳이 할 필요는 없습니다.
알아서 삭제시 Service와 연결된 Pod의 Endpoint를 삭제하기 때문입니다.

#### `startupProbe`를 사용해야하는 상황

 `startupProbe`는 위의 두 probe들과는 약간 다른 성격을 가졌습니다.
 `livenessProbe`와 함께 사용이 되는데요, container가 `initialDelaySeconds + failureThreshold × periodSeconds`만큼의 시간이 지난 후에 정상동작을 할 경우(container가 작업을 시작하기까지 시간이 오래 걸리는 경우) `livenessProbe`에 의해 fail이 발생하고, 재시작 되는것을 막아 deadlock 상태를 방지해줍니다.

### Restart policy

PodSpec에서 `restartPolicy` 필드에는 `Always`, `OnFailure`, `Never`를 사용할 수 있습니다.
그 중 default는 `Always`입니다.

`restartPolicy`는 Pod 내의 모든 container에 적용됩니다.
또한 `restartPolicy`는 exponential back-off delay로 재시작됩니다.
즉, 10초, 20초, 40초로 계속해서 일정수준까지 delay가 늘어납니다.
성공적으로 실행되고 나서 10분이 지나면 해당 delay는 초기화됩니다.

### Pod lifetime

일반적으로 Pod는 사람 또는 컨트롤러가 명백하게 이를 지우지 않는 이상 유지됩니다.
control plane은 Pod의 총 개수가 지정된 threshold를 초과하면(node마다 정해져 있습니다) 종료된 Pod들(`Succeeded` 또는 `Failed`)을 삭제합니다.

컨트롤러는 3가지 타입이 있습니다.

* `Job`: batch computations처럼 종료될것으로 예상되는 Pod입니다.
* `ReplicationController`, `ReplicaSet`, `Deployment`: 종료되지 않을 것으로 예상되는 Pod입니다.
* `DaemonSet`: 머신마다 하나씩 동작해야하는 Pod입니다.

## Init Container

Init container는 app image에서 사용할 수 없거나 사용하지 않는 setup script나 utility들을 포함할 수 있습니다.
실제 app container가 시작되기 전에 먼저 필요한 작업들을 수행하는데 사용됩니다.

Pod Specification에서 `containers` 배열과 같은 개위로 작성하면 Init container를 사용할 수 있습니다.

Init container는 completion이 되기 위해 실행됩니다.
따라서 complete상태가 되면 재시작되지 않습니다.
completion을 위해 실행되므로 당연하게도 `readinessProbe`는 사용할 수 없습니다.

init container는 여러개를 정의했을 경우 `kubelet`은 이를 순서대로 실행됩니다.
그리고 각 init container는 다음 init container가 실행되기 전에 반드시 성공적으로 종료되어야 합니다.

init container가 실패하면 성공할때까지 재시작합니다.
하지만 Pod의 `restartPolicy`가 `Never`이면 init container도 재시작하지 않습니다.

init container는 app container가 사용할 수 있는 대부분의 필드를 그대로 사용할 수 있습니다.
일반적인 container와의 차이점은 resource에 대해 다르게 관리된다는 것입니다.
자세한 내용은 공식 홈페이지의 docs를 확인하시기 바랍니다.

### Init container 사용하기

* Init container는 app image에는 없는 utility나 custom code를 포함할 수 있습니다.
* Init container는 동일한 Pod 내에 있는 app container와는 다른 filesystem view를 가질 수 있습니다.
  따라서 app container는 접근할 수 없는 Secret을 가지고 동작할 수 있습니다.
* Init container가 성공할 때까지 Pod의 app container들은 생성되지 않습니다.
* App container보다 안전하게 utility, custom code를 실행시킬 수 있습니다. 따라서 보안 취약점을 줄일 수 있습니다.

메인 app container를 실행할 때 필요한 configuration file에 필요한 value들을 주입할 때 init container를 사용할 수 있습니다.

### Detailed behavior

Pod가 시작되는 동안 network와 volume들이 초기화 된 후 init container가 순서대로 실행되게 됩니다.
각 container는 반드시 다음 container가 실행되기 전까지 성공적으로 종료되어야 합니다.

Init container에 대한 spec 변경은 container image에 대한것만 가능하다.
그리고 Init container는 idempotent[^1]가 성립해야합니다.

Init container가 실패 시 계속해서 재시작 되는 것을 막으려면 Pod에 `activeDeadlineSeconds`와 Container에 `livenessProbe`를 설정하면 막을 수 있습니다.

### Resource 다루는 법

* 모든 init container에 대해 가장 높은 resource request나 limit은 `effective init request/limit`이라고 정의합니다.
* Pod의 `effective request/limit`은 다음보다 커야합니다.
  * 모든 app container의 resource에 대한 request/limit의 합
  * resource에 대한 `effective init request/lmit`
* `effective request/limits`를 기준으로 스케쥴링합니다.
  즉, init container의 resource는 Pod의 life 동안 사용되지 않음을 의미합니다.
* Pod의 `effective QoS tier`에서 QoS tier는 init container와 app container의 QoS tier와 같습니다.

### init container가 재시작되는 경우

* user가 pod specification을 업데이트 하여 init container의 이미지가 변경되었을 경우입니다.
  App container image의 변화는 app container만 재시작시킵니다.
* Pod infrastructure container가 재시작 되었을 때 Init container가 실행됩니다.
* `restartPolicy`가 `Always`로 설정이 되어있는 상태에서 Pod가 재시작 되었을 때 init container가 이전 완료 상태를 저장한 것이 만료되거나 없을 경우 재시작될 수 있습니다.

## Disruptions

Pod는 원래 누군가가(사람 또는 컨트롤러) 지우지 않는다면, 또는 피할 수 없는 하드웨어, 소프트웨어적인 에러가 아니라면 삭제되지 않습니다.

여기서 `unavoidable`인 경우를 involuntary disruptions라고 부릅니다.
involuntary disruption에는 다음과 같은 것들이 있습니다.

* hardware failure
* cluster administrator가 실수로 VM을 삭제
* cloud provider나 hypervisor의 장애로 VM이 삭제됨
* kernel panic
* cluster network partition에 의해 node가 cluster에서 사라짐
* 노드가 out-of-resource여서 pod의 eviction이 실행됨

voluntary disruption은 application이나 cluster administrator에 의해 시작된 동작들입니다.
voluntary disruption에는 다음과 같은 것들이 있습니다.

* 해당 Pod를 관리하고 있던 delployment나 다른 controller의 삭제
* deployment의 Pod template update가 재시작을 유발함
* 직접적으로 Pod를 삭제

Cluster Administrator는 다음이 disruption을 유발할 수 있습니다.

* Upgrade를 위한 Draining Node
* cluster를 scale down 하기 위해 Draining Node
* 특정 노드에 띄워야 하는 요구사항 때문에 기존에 있던 해당 노드에서 Pod를 제거

### Dealing with Disruptions

* Pod에게 충분한 양의 resource 할당하기
* 고가용성을 원할경우 application을 복제하기
* application을 rack또는 zone에 분배하기

### How Disruption Budgets Work

`PodDisruptionBudget(PDB)`를 각 application에 설정할 수 있습니다.
이는 **voluntary disruption** 상황에서 동시에 down될 수 있는 pod의 갯수를 제한합니다.

`PodDisruptionBudget(PDB)`를 사용하려면 Cluster Manager는 Eviction API를 통해서 Pod를 삭제해야합니다.
즉, 직접 Pod나 Deployment를 삭제하게 되면 PDB를 사용하지 못하게 됩니다.
Eviction API를 사용하는 예시에는 `kubectl drain`가 있습니다.

`PodDisruptionBudget(PDB)`는 involuntary disruption 상황에서는 작동하지 않습니다.
하지만 몇개가 종료되는지는 기록하여 budget에 추가합니다.

Rolloing upgrade 때문에 Pod가 삭제되거나 사용 불가능한 상태일 때에도 `PodDisruptionBudget(PDB)`는 이를 카운트하지만 PDB때문에 제한되지는 않습니다.
application의 업데이트 동안 발생한 장애처리는 controller spec에서 정의내린대로 실행합니다.

[^1]: 멱등법칙. 여러번 실행하더라도 동일한 결과를 냄.

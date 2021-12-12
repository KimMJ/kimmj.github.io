---
title: "Kubernetes Service는 어떻게 iptables 설정이 되는가"
date:  2021-02-10T16:18:17+09:00
weight: 10
draft: false
tags: ["kubernetes", "service", "iptables"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`kube-proxy` 는 `daemonset` 으로 각각의 노드에 모두 떠있다. 역할은 kubernetes에서의 `service` 가 가지고 있는 `Virtual IP` 로 트래픽을 전달할 수 있도록 적절한 조작을 해주는 것이다. 기본적으로 3가지 모드가 있으나, 일반적으로는 `iptables` 모드를 많이 사용한다.

- User space
- `iptables`

    Linux Kernel 에서의 `netfilter` 를 사용하여 kubernetes 서비스에 대한 라우팅을 설정하는 것이다. 이 모드가 default 옵션이다. 여러개의 `pod` 로 트래픽을 load balancing 할 때 `unweighted round-robin scheduling` 을 사용한다.

- `IPVS`(IP Virtual Server)

    `netfilter` framework 에 속해있는 것으로, `IPVS` 는 `least connection`이나 `shortest expected delay`와 같은 layer-4 load balancing algorithm을 지원한다. 

kubernetes에서 `Service` 가 만들어지면 `kube-proxy` 는 `netfilter` chain을 설정하여 connection이 node의 kernel에 의해 바로 pod-container의 endpoint까지 가도록 한다.

## Service로의 통신

![service-flow.png](/images/Kubernetes/service-flow.png)

1. `KUBE-MARK-MASQ` 는 masquerade를 이용하여 `Serivce` 로 이동하는 패킷이 cluster의 외부에서 들어온 패킷임을 표시한다. 이렇게 마킹이 된 패킷은 `POSTROUTING` 룰에서 Source IP를 해당 노드의 IP로 `SNAT` 을 할 때 사용된다.
2. `KUBE-SVC-XXX` 는 `Service` 로 가는 모든 트래픽에 대해 적용이 되며, 각 `Serivce` 의 `Endpoint` 로 가는 rule을 정의한다. 어디로 갈지는 랜덤으로 보낸다.
    1. `KUBE-SEP-XXX`

        `KUBE-MARK-MASQ` 를 사용하여 필요할 경우 `SNAT` 을 해야한다고 표시한다.

        그 다음 `DNAT` 을 통해 `Endpoint` 로 목적지를 변경한다.

3. `KUBE-MARK-DROP`

    앞선 rule 들에 의해 `DNAT` 이 되지 않은 패킷들에 대해서 `netfilter` 에 마킹을 한다. 이 패킷들은 `KUBE-FIREWALL` chain 에 의해 버려진다.

이렇게 할 경우 `Endpoint` 가 되는 `Pod` 가 각각의 노드에 있을 때, 어떤 곳을 더 선호한다던지 그런 규칙이 없다. 다만, `externalTrafficPolicy: Local` 을 설정하게 되면 요청을 받은 Node에 실제로 `Endpoint` 로 사용될 `Pod` 가 없게되면 connection이 refuse 된다.

## NodePort로의 통신

`NodePort` 를 사용하게 되면 `kube-proxy` 는 `--service-node-port-range` 에 의해서 설정된 값들 중 임의로 하나를 선택하여 열리게 된다. 이 때, Kubernetes가 아닌 다른 프로세스에 의해 점유되는 것을 막기 위하여 이 포트를 bind하고 listen하게  된다. 만약 `NodePort` 를 사용하기 위해 포트를 bind하기 전 다른 프로세스가 bind를 해버린다면, `NodePort` 의 할당은 에러로그를 출력할 것이다.

만약 `kube-proxy` 가 죽어있는 상태에서(`bind` 가 풀리게 될 것이다) `NodePort` 와 동일한 포트를 다른 프로세스에서 `bind`, `listen` 을 하게되면 어떻게 될까? 이 경우에도 마찬가지로 Kubernetes의 Endpoint로 패킷이 전달된다. 이는 `iptables` 의 `nat` table에서 `PREROUTING` chain 때문이다. 

### PREROUTING chain

`PREROUTING` chain은 Linux kernel의 networking stack에서의 가장 먼저 발현되는 rule이다.

`kube-proxy` 가 생성한 `PREROUTING` chain은 패킷이 현재 노드의 local socket으로 가야하는지, 아니면 파드로 포워딩해야하는지 결정해준다. 따라서 `NODE_IP:NODE_PORT` 로의 요청은 해당 노드에서 해당 파드가 아닌 다른 프로세스가 그 port를 사용하고 있다고 하더라도, 항상 파드로 갈 수 있게 된다.

![nodeport-flow](/images/Kubernetes/nodeport-flow.png)

`PREROUTING` 을 통해 NodePort로 사용중인지 확인하고, 파드로 보낼지 아니면 다른 프로세스로 보낼지 결정한다.

## Reference

### link

- [https://ronaknathani.com/blog/2020/07/kubernetes-nodeport-and-iptables-rules/](https://ronaknathani.com/blog/2020/07/kubernetes-nodeport-and-iptables-rules/)
- [https://letslearn24x7.blogspot.com/2020/10/lets-learn-about-kube-proxy-in-iptables.html](https://letslearn24x7.blogspot.com/2020/10/lets-learn-about-kube-proxy-in-iptables.html)
- [https://kubernetes.io/ko/docs/tutorials/services/source-ip/](https://kubernetes.io/ko/docs/tutorials/services/source-ip/)

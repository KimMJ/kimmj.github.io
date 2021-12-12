---
title: "09 Networking"
date:  2020-05-04T18:12:04+09:00
weight: 9
draft: false
tags: ["kubernetes", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Switching Routing

* Switching
  * 컴퓨터 두대를 스위치를 통해 연결하는 방법.
  * 연결된 스위치의 인터페이스에 ip를 할당한다.
  * 동일한 네트워크로만 트래픽을 보내준다.
* Routing
  * 라우터는 스위치로 연결된 두 네트워크 대역을 연결해준다.
  * 각 네트워크 대역에 대해서 아이피를 동시에 가지고 있다.
* Gateway
  * 어떤 아이피 대역에 대해서 어떤 라우터를 통해서 가게 할지 결정하는 것
  * 외부 인터넷과 연결하고 싶다면 먼저 인터넷을 라우터에 연결하고, 이 곳으로 route 설정을 하면 된다.
  * default gateway는 `Destination=0.0.0.0`인 것.
  * gateway가 `0.0.0.0`이라면 라우터가 필요없다는 의미.
* 기본적으로 `/proc/sys/net/ipv4/ip_forward`가 `0`으로 설정되어 자신에게 오는 트래픽이 아닌 경우 포워딩하지 않는다.

## DNS

* `/etc/hosts`에 domain name을 넣을 수 있다.
* DNS server에 domain name을 정리하고, 각 컴퓨터에서 이를 조회하도록 한다.
* `/etc/resolv.conf`에 nameserver를 넣어서 서버에 있는 dns를 사용할 수 있다.
* domain name lookup할 때 local 파일부터 찾고 그 다음에 nameserver에서 검색한다.
* DNS server에 `Forward All to 8.8.8.8`로 public internet의 DNS를 default로 참조하도록 할 수 있다.
* `apps.google.com`으로 조회를 하면 `root DNS`, `.com DNS`, `Google DNS`의 순서로 조회하여 IP를 가져오고, 이를 캐싱하게 된다.
* `/etc/resolv.conf`에 `search`를 `mycompany.com`으로 등록하게 되면 `ping web`을 입력했을 때 `web.mycompany.com`으로 검색한다.
* Record Types
  * A: IPv4
  * AAAA: IPv6
  * CNAME: name alias (name to name mapping)

## Network Namespaces

* 쿠버네티스는 네임스페이스를 기준으로 나누어져 있다.
* 네임스페이스 안에서는 서로의 프로세스가 무엇인지 볼 수 있다.
* 컨테이너에서 실행한 것은 호스트에서 프로세스 확인이 가능하다.
  * process ID는 컨테이너 내부와 바깥이 서로 다르다.
* 컨테이너가 생성되면 네트워크 네임스페이스를 호스트에서 생성한다.
  * 컨테이너는 그 안에서 Routing Table과 ARP Table을 가진다.
* 리눅스 호스트에서 network namespace를 생성하려면 `ip netns add red`와 같이 입력하면 된다.
  * `ip netns`로 리스트 확인 가능.
* `ip link`를 입력하면 interface를 볼 수 있는데 이를 네임스페이스 아래에서 실행하고자 한다면 다음과 같이 하면 된다.
  * `ip netns exec red ip link`
  * 또는 `ip -n red link`
* 나머지도 비슷하다.
  * `ip netns exec red arp`
  * `ip netns exec red route`
* 네트워크 네임스페이스 두개를 연결하려면 파이프를 연결하면 된다(가상의 랜선이라고 생각하면 된다.)
  * `ip link add veth-red type veth peer name veth-blue`
  * `ip link set veth-red netns red`
  * `ip link set veth-blue netns blue`
  * `ip -n red addr add 192.168.15.1 dev veth-red`
  * `ip -n blue addr add 192.168.15.2 dev veth-blue`
  * `ip -n red link set veth-red up`
  * `ip -n blue link set veth-blue up`
  * `ip netns exec red ping 182.168.15.2`
* virtual switch를 통해 네트워크 네임스페이스를 연결할 수 있다.
  * `linux bridge`, `open vswitch`가 있다.
  * `ip link add v-net-0 type bridge`
  * `ip link set dev v-net-0 up`
  * `ip -n red link del veth-red` <- 위에서 생성한 연결을 끊는 것.
  * `ip link add veth-red type veth peer name veth-red-br`
  * `ip link add veth-blue type veth peer name veth-blue -br`
  * `ip link set veth-red netns red`
  * `ip link set veth-red-br master v-net-0`
  * `ip link set veth-blue netns blue`
  * `ip link set veth-blue-br master v-net-0`
  * `ip -n red addr add 192.168.15.1 dev veth-red`
  * `ip -n blue addr add 192.168.15.2 dev veth-blue`
  * `ip -n red link set veth-red up`
  * `ip -n blued link set veth-blue up`
  * 이를 호스트의 네트워크와 연결할 수 있다.
    * virtual switch(bridge)는 호스트의 자원.
    * `ip addr add 192.168.15.5/24 dev v-net-0`을 추가하면 연결이 가능
  * 외부 인터넷과 연결하려면 호스트의 `eth0`와 연결을 해야한다.
  * `gateway`를 추가.
  * `ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5`
    * 이 상태에서는 ping이 되지 않음.
    * private ip를 달고 나가기 때문에 외부에서는 인식하지 못하는 IP.
    * 따라서 NAT를 해야한다.
    * `iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE`
* 외부에서 network namespace로 들어오려면?
  * 2가지 해법이 있음.
    * 직접 ip 할당하고 라우팅하기
    * `iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT`

## Docker Networking

* `docker run`에서 `--network none`을 하면 컨테이너는 어떤 곳과도 통신할 수 없다.
* `--network host`를 사용하여 port를 열면 host의 port가 열린다.
* `--network bridge`는 default로 도커 컨테이너끼리 통신이 가능하다.
* `docker`는 실행시 default로 brdige를 생성한다.
* `docker network ls`를 통해 확인 가능.
  * `ip link`를 통해 `docker0`를 볼 수 있다.
    * `ip link add docker0 type bridge`를 통해 생성한 것과 동일.
  * `ip addr`을 통해 보면 ip를 확인할 수 있다. (brdige의 ip)
* `docker run`을 하면 도커는 네트워크 네임스페이스를 생성한다.
  * `ip netns`를 보면 확인이 가능하다.
  * `docker inspect <container id>`를 통해서 어떤 네트워크 네임스페이스인지 확인이 가능하다.
* host에서 `ip link`를 하면 네임스페이스와 bridge(`docker0`)를 연결한 것이 보인다.
  * `ip -n <network namespace> link`를 하면 컨테이너에서 brdige에 연결한 것이 보인다.
  * `ip -n <network namespace> addr`을 하면 컨테이너의 아이피를 확인할 수 있다.
* 컨테이너의 port를 호스트의 port와 연결하려면 `-p 8080:80`과 같은 옵션을 사용하면 된다.
  * 이는 `nat`를 사용하여 할 수 있다. 
  * `ip tables -t nat -A DOCKER -j DNAT --dport 8080 --to-destination 172.17.0.3:80`
  * `iptables -nvL -t nat`을 통해 확인 가능.

## CNI

* bridge program은 특정한 task를 쭉 실행해준다.
* `bridge add <cid> <namespace>`를 통해 할 수 있다.
  * 쿠버네티스에서도 이 방식을 사용한다.
* container runtime간에 제대로 동작을 하는지 확인하기 위한 표준이 생겼다.
  * `bridge`도 CNI의 plugin 중 하나이다.
* CNI 표준
  * Container Runtime은 반드시 network namespace를 생성해야한다.
  * container가 연결되어야 하는 network를 확인해야한다.
  * container runtime은 컨테이너가 생성될 때 network plugin(brdige)를 호출해야한다.
  * container runtime은 컨테이너가 삭제될 때 network plugin(brdige)를 호출해야한다.
  * JSON 형식으로 network configuration을 할 수 있어야 한다.
* Plugin 표준
  * add/del/check에 대한 command line arguments를 제공해야한다.
  * contianer id, network 등에 대한 파라미터를 제공해야한다.
  * 파드에 할당되는 IP를 관리해야한다.
  * 특정한 형식으로 결과를 리턴해야한다.
* Plugin에는 다음과 같은 것들이 있다.
  * bridge
  * vlan
  * ipvlan
  * macvlan
  * windows
  * dhcp
  * host-local
* 그러나 docker는 CNI 표준을 지키지 않는다.
  * 자신만의 독자적인 Container Network Model을 사용한다.
  * 따라서 `docker run --network=cni-brdige nginx`와 같이 사용할 수는 없다.
  * 하지만 그렇다고 해서 docker에서 아주 못사용하는 것은 아니다.
    * `docker run --network=none nginx`처럼 우선 생성한 뒤 수동으로 plugin을 실행하면 된다.
    * 이 방식을 쿠버네티스에서 사용한다.

## Cluster Netwrking

* 쿠버네티스는 마스터, 워커 노드가 있고, 이들이 네트워크로 연결이 되어야 한다.
* 이들은 서로 `hostname`과 MAC address가 달라야 한다.
* 마스터 노드는 특정한 포트를 열어야 한다.
  * `kube-apiserver`에서 사용하기 위한 6443 포트
  * `kube-scheduler`에서 사용하기 위한 10251 포트
  * `kube-controller-manager`에서 사용하기 위한 10252 포트
  * `ETCD`에서 사용하기 위한 2379, multi-master일 경우 2380도 열어야 한다.
* 또한 워커 노드는 `kubelet`을 위한 10250 포트를 열어야 한다.
  * 그리고 service expose를 위한 30000-32767까지의 포트를 열 수 있어야 한다.

## Pod Networking

* pod layer에서의 네트워킹이 중요하다.
* 파드는 수백개가 생성이 될 수 있는데 이들끼리 서로 통신이 되어야 하고, 아이피가 할당되어야 한다.
* 쿠버네티스는 built-in solution으로 이를 해결하지 않는다.
* 다음이 구현되어야 한다.
  * 모든 파드는 IP address를 가지고 있다.
  * 모든 파드는 같은 노드 안에서의 파드와 통신할 수 있어야 한다.
  * 모든 파드는 다른 노드에 있는 파드와 NAT가 아닌 방법으로 통신할 수 있어야 한다.
* 이를 구현한 네트워킹 모델들이 있다. (flannel, calico 등)
* 강의에서는 이전에 배운 네트워크 네임스페이스를 기반으로 어떻게 동작하는지를 알아볼 것이다.
  * 각 노드들은 `192.168.1.0` 대역을 가지고 있다고 가정한다.
  * 각각의 노드에 대해서 bridge를 생성한다.
  * 그 다음 bridge를 up 한다.
  * 각 노드에 대해서 `10.244.1.0/24`, `10.244.2.0/24`, `10.244.3.0/24`의 네트워크 대역을 가지도록 한다.
  * 그 뒤 컨테이너가 생성될때마다 특정 스크립트를 생성해서 아이피를 할당하고 노드 내부 bridge와 연결한다.
  * 그 다음 각 노드의 파드끼리 route 설정을 넣어준다.
* 이런 것들을 CNI를 통해서 해결한다.
* `kubelet`은 각 노드에 있는 컨테이너의 생성/삭제를 관리한다.
  * 따라서 컨테이너가 생성될 때 `kubelet`을 실행할 때 사용된 arguments에 있는 CNI를 확인하고, script를 실행한다.

## CNI in kubernetes

* CNI plugin은 각 `kubelet`을 실행할 대 arguments로 등록할 수 있다.
* `--cni-bin-dir`에서 설정한 디렉토리에는 CNI의 binary 파일들이 있다.
* `--cni-conf`에는 어떤 CNI를 사용할지 알파벳 순서로 골라서 사용한다.

## CNI weave

* 파드에서 파드로 가는 트래픽은 노드 바깥의 router로 나간 뒤, 알맞은 노드로 들어가서 파드에 전달된다.
  * 그러나 이는 수백개의 노드가 각각 수백개의 파드를 가지고 있는 상황에서는 제대로 작동할 수 없다.
  * 라우팅 테이블이 모든것을 담을 수 없다.
* weave는 각각의 노드에 agent를 두고, 서로 통신하게 하여 서로의 위치를 확인한다.
  * agent는 다른 노드의 파드로 가는 트래픽을 가로채서 어디로 보낼지 확인한 뒤 트래픽을 감싼다.
  * 해당 트래픽은 다른 노드로 전달된 뒤에 agent가 다시 가로채서 트래픽을 확인하고 원하는 파드로 전달한다.

## IP Address Management - Weave

* 어떻게 노드의 Virtual Brdige가 IP를 할당하는지에 대해 알아볼 것이다.
* network plugin에 의존하여 CNI는 IP를 할당한다.
  * 쿠버네티스는 할당한 IP의 관리에는 영향을 주지 않는다.
* `/etc/cni/net.d/net-script.conf`에는 사용하는 CNI에 대한 정보가 적혀있다(`IPAM`).
* `weaveworks`는 `10.32.0.0/12`로 IP 대역을 파드에 할당하려 하고 각 노드에 `10.32.0.1`, `10.38.0.1`, `10.44.0.0`으로 IP 대역을 나누어 노드에 할당한다.
* customization이 가능하다.

## Service Networking

* 서비스가 생성되었을 때에도 노드의 위치에 상관없이 모든 파드에서 접근이 가능해야한다.
* 서비스는 특정 노드에 bound되는 것이 아니라 노드 전역에 걸쳐 생성된다.
* ClusterIP는 클러스터 내부에서만 연결이 된다.
  * DB처럼 클러스터 내부에서만 통신하려는 목적이면 ClusterIP를 사용하면 된다.
* 외부로 노출하기 위해서는 NodePort를 사용한다.
  * IP를 가지는것 뿐만 아니라 노드에서 포트를 할당받기도 한다.
* service는 단순히 virtual object로 어딘가에 생성되는 것이 아니다.
  * 쿠버네티스에 serivce를 생성하면 미리 설정된 범위에서 IP 주소를 할당받는다.
  * 할당받은 IP 주소로 forwarding rule이 생성된다.
  * 단순히 IP로 forwarding하지 않고 IP:port로 forwarding한다
* `userspace`, `iptables`, `ipvs` 모드로 `kube-proxy`가 설정될 수 있으며 기본은 `iptables`이다.
* service의 ip range는 `kube-api-server`에서 default로 `10.0.0.0/24`로 설정되어있다.
* pod ip range와 service ip range는 겹치면 안된다.
* `kube-proxy`의 로그를 보면 어떻게 iptables를 조작했는지 볼 수 있다.

## Cluster DNS

* 쿠버네티스는 설치시 자신만의 DNS 서버를 가진다.
* 서비스가 생성이 될때, hostname과 IP 주스를 해당 서비스에 맞게 등록한다.
* 파드 내에서는 해당 domain을 통해 접근할 수 있다.
* 동일한 네임스페이스에서는 `service name`만 가지고 접근할 수 있다.
* 다른 네임스페이스에 있는 경우 `<service name>.<namespace>`로 접근해야한다.
* `apps` namespace 안의 `web-service`가 있는 경우
  * `hostname`: `web-service`
  * `namespace`: `apps`
  * `type`: `svc`
  * `root`: `cluster.local`
* pod에 대해서도 dns에 등록이 된다. 그러나 IP 주소를 `10-244-2-5`와 같이 dash를 통해 구분할 뿐이다.
  * `type`: `pod`

## CoreDNS in Kubernetes

* 여러 서비스 및 파드가 생성/삭제될 때 domain을 정리하기 위해 중앙의 DNS 서버를 사용한다.
* 파드 내에서는 `/etc/resolv.conf`에서 nameserver로 등록한다.
* 새로운 파드가 생기면 DNS 서버에 엔트리를 추가한다.
* 1.12 이전에는 `kube-dns`를 사용했지만 이후로는 `CoreDNS`를 권장한다.
* `CoreDNS`는 `kube-system` namespace에 파드로 배포되어있다.
  * `Deployment`로 두개가 동작중이다.
* `CoreDNS`는 `Corefile`이라는 파일을 이용한다.
  * 여기에 `kubernetes`라는 plugin 구역을 보면 된다.
  * root domain이 적혀있다.
  * `pods insecure` 부분은 파드에 대한 record를 관리할지를 결정한다.
    * default로 disable이다.
  * `proxy` 부분은 dns에 등록되지 않은 것들을 `/etc/resolv.conf`에서 검색하도록 설정되어있다.
  * 이 파일은 `configmap`으로 전달된다.
* 파드에서 DNS 서버에 접근할 떄는 service사용한다.
  * 기본적으로 `CoreDNS`도 `kube-dns` 이름으로 사용한다.
  * 이는 파드가 생성될 때 자동으로 IP를 기입한다. (`/etc/resolv.conf`)
  * `kubelet`이 설정한다.
    * option으로 설정되어있음.
  * `/etc/resolv.conf`에 `search` 항목으로 `default.svc.cluster.local`, `svc.cluster.local`, `cluster.local`이 설정되어 있어서 간단하게 이름만 입력해도 근이 가능한 것이다.
    * 그러나 pod는 정의되어있지 않아서 FQDN으로 접근해야한다.

## Ingress

* service와 ingress의 차이점은 무엇인가?
* 일반적으로 NodePort로 expose하면 30000 이상의 port(ex: 38080)를 사용하게 된다.
  * 유저가 80포트를 통해 접속하도록(http) 하려면 proxy server를 둬서 트래픽을 80에서 38080으로 전달해야 한다.
  * LoadBalancer를 사용할 때도 이와 비슷하다.
* Ingress는 유저가 하나의 external accessble URL을 통해 클러스터내의 서비스에 접속할 수 있게 해준다.
  * https설정도 할 수 있다.
  * ingress 또한 expose해야한다.
  * NodePort 또는 LoadBalancer.
  * 그러나 서비스가 늘어나도 한번만 수행하면 된다.
* ingress controller, ingress resources로 ingress 설정을 한다.
  * default로 설치되어있지 않음.
  * GCE와 Nginx는 쿠버네티스 프로젝트에서 관리한다.
* nginx는 configmap으로 nginx의 설정을 한다.
  * ServiceAccount를 통해 적절한 권한을 줘야 한다.
* ingress resource는 url에 따라 다른 파드로 트래픽을 전달할 수 있다.
* ingress에 backend로 serviceName과 servicePort를 지정하여 트래픽을 전달할 수 있다.
  * 해당 ingress로 온 트래픽은 모두 serviceName으로 전달된다.
* 각각의 rule을 지정하여 원하는 서비스에 트래픽을 전달할 수 있다.
  * host를 따로 지정하지 않으면 모든 트래픽에 대해 매칭이 된다.

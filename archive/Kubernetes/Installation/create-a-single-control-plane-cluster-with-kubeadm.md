---
title: "Create a Single Control Plane Cluster With Kubeadm"
menuTitle: "Create a Single Control Plane Cluster With Kubeadm"
date:  2020-01-18T22:27:39+09:00
weight: 3
draft: false
tags: ["kubeadm", "instll"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

이 문서에서는 Master 노드 한대로 클러스터를 구성하는 방법에 대해 알아보도록 하겠습니다.

먼저, 파드 네트워크에 사용할 add-on을 선정합니다.
그런뒤 `kubeadm init`을 할 때 필요로 하는 사항이 있는지 확인해야 합니다.

저는 `Calico`를 사용할 것입니다.
`Calico`는 `kubeadm init`에서 `--pod-network-cidr=192.168.0.0/16`을 해주거나, 나중에 `calico.yml` 파일에서 적절하게 수정해주어야 한다고 합니다.
저는 Pod Network에 사용될 IP 대역을 10.1.0.0/16 대역을 사용하고자 합니다.
그러기 위해 `kubeadm init`을 `--pod-network-cidr=10.1.0.0/16` 옵션을 통해 실행할 것입니다.

```bash
kubeadm init --pod-network-cidr=10.1.0.0/16
```

몇분 후 설치가 완료될 것입니다.
그러면 아래에 `kubeadm join` 이라면서 어떻게 다른 node들을 join 시키는지 설명이 되어 있습니다.

클러스터를 사용하려면 다음과 같은 명령어를 통해서 `kubectl`에서 접근이 가능하도록 해야합니다.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

그 다음엔 각 worker node로 접속하여 설치시 나왔던 `kubeadm join` 명령어를 복사하여 입력합니다.
그런 뒤 master node로 접속하여 `kubectl get nodes`를 입력하고 결과를 확인합니다.
성공했을 경우 node들이 보일 것입니다.

이제 `calico`를 설치하도록 합니다.

```bash
curl -o calico.yaml https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

이후 `calico.yaml`에서 `CALICO_IPV4POOL_CIDR` 부분을 수정해줍니다.

```yaml
            - name: CALICO_IPV4POOL_CIDR
              value: "10.1.0.0/16"
```

#### Reference
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/  
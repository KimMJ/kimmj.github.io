---
title: "Install Kubeadm"
menuTitle: "Install Kubeadm"
date:  2020-01-17T00:38:41+09:00
weight: 2
draft: false
tags: ["install", "kubeadm"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`kubeadm`은 Kubernetes cluster의 설정들을 관리하는 툴입니다.

## Prerequisites

먼저, 몇가지 전제사항이 있습니다.

1. 모든 노드의 MAC 주소와 product_uuid가 달라야 합니다.
   `ifconfig -a`와 `sudo cat /sys/class/dmi/id/product_uuid`를 통해 알 수 있습니다.
2. network adapter를 확인합니다.
   하나 이상의 network adapter가 있고, Kubernetes component들이 default route로 통신이 불가능하다면
   IP route를 설정하여 Kubernetes cluster 주소가 적절한 adapter를 통해 이동할 수 있도록 해주는 것이 좋습니다.
3. `iptables`를 사용하는지 확인
   Ubuntu 19.04 이후버전부터는 `nftables`라는 것을 사용한다고 합니다.
   그러나 이는 `kube-proxy`와 호환이 잘 안되기 때문에 `iptables`를 사용해야한다고 하고 있습니다.

   다음과 같이 설정할 수 있습니다.

   ```bash
    sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
    sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
    sudo update-alternatives --set arptables /usr/sbin/arptables-legacy
    sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy
   ```
4. 필수 포트 확인

   **Control-plane node(s)**
   
   | Protocol | Direction | Port Range | Purpose                 | Used By                   |
   |----------|-----------|------------|-------------------------|---------------------------|
   | TCP      | Inbound   | 6443*      | Kubernetes API server   | All                       |
   | TCP      | Inbound   | 2379-2380  | etcd server client API  | kube-apiserver, etcd      |
   | TCP      | Inbound   | 10250      | Kubelet API             | Self, Control plane       |
   | TCP      | Inbound   | 10251      | kube-scheduler          | Self                      |
   | TCP      | Inbound   | 10252      | kube-controller-manager | Self                      |

    **Worker node(s)**

   | Protocol | Direction | Port Range  | Purpose               | Used By                 |
   |----------|-----------|-------------|-----------------------|-------------------------|
   | TCP      | Inbound   | 10250       | Kubelet API           | Self, Control plane     |
   | TCP      | Inbound   | 30000-32767 | NodePort Services**   | All                     |

   여기서 `*` 표시가 있는 것은 수정이 가능한 사항이라고 합니다. (API server의 port, NodePort Service로 열 수 있는 port의 범위)

5. container runtime 설치
   여기에서는 `Docker`를 사용할 것입니다.
   `Docker` 홈페이지에는 script를 통해 최신 버전의 `Docker`를 다운로드 받는 방법을 제공합니다.

   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   ```

6. `kubeadm`, `kubelet`, `kubectl` 설치
   Kubernetes에 필요한 세가지 툴을 설치하도록 하겠습니다.
   간단하게 설명드리자면 `kubeadm`은 Kubernetes를 관리하는 툴이라고 생각하면 되고 `kubelet`은 명령을 이행하는 툴이라고 생각하면 됩니다.
   `kubectl`은 Kubernetes cluster와 통신하는 툴입니다.

   ```bash
   sudo apt-get update && sudo apt-get install -y apt-transport-https curl
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
   deb https://apt.kubernetes.io/ kubernetes-xenial main
   EOF
   sudo apt-get update
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

   마지막에 `apt-mark`를 통해 `kubelet`, `kubeadm`, `kubectl`의 버전을 고정시켰습니다.

이렇게 `kubelet`, `kubeadm`, `kubectl`을 설치하였으면 마지막으로 swap 영역을 제거할 것입니다.
Kubernetes는 swap 영역이 없는 것이 필수 항목입니다.
따라서 다음의 명령어로 swap 영역을 삭제합니다.

```bash
sudo swapoff -a
```

이는 현재 세션에서만 동작하고 재부팅시 해제됩니다.
따라서 `/etc/fstab`을 열고 swap 부분을 주석처리합니다.

```yaml
#/swapfile                                 none            swap    sw              0       0
```

#### Reference

https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/calico-network-policy/  
https://kubernetes.io/docs/concepts/cluster-administration/networking/  
https://docs.projectcalico.org/v3.11/getting-started/kubernetes/  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
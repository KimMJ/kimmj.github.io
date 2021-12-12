---
title: "Overview"
menuTitle: "Overview"
date:  2020-01-14T01:42:53+09:00
weight: 1
draft: false
tags: ["kubernetes", "install", "overview"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

저의 vm으로 구성한 클러스터를 설명드리고자 합니다.

* Cloud Provider: Kubernetes on-prem (4 VMs)
  * 1 for master (4GB Mem, 2 CPU)
  * 3 for worker (each 8GM Mem, 4 CPU)
* Kubernetes 1.17.0
* Storage Class: Ceph
* OS: Ubuntu 18.04.2 Server
* Internal Network: VirtualBox Host-Only Ethernet Adapter (192.168.x.0/24)
* External Network: Bridge to adapter (192.168.y.0/24)

한번에 쳐야하는 명령어가 많기 때문에, `tmux`를 사용해서 여러개의 pane을 생성하고 각각에 대해 `ssh`로 접속하였습니다.

![tmux](/images/Kubernetes/Install/tmux.png)

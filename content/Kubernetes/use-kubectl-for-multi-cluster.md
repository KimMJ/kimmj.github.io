---
title: "kubectl로 multi cluster에 접속하기"
date:  2020-02-23T15:30:33+09:00
weight: 10
draft: true
tags: ["kubectl", "kubernetes", "multi-cluster"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`kubectl`은 kubernetes cluster에 접속하여 명령어를 보내고, 그 결과를 받아오는 프로그램입니다.
이게 없으면 cluster에 명령을 내릴수도, 어떤 서비스도 deploy할 수가 없습니다.

보통 kubernetes master node에 접속할 수 있다면 ssh로 접속하여 `kubectl`을 사용합니다.
그러나 로컬에 있는 yaml을 가지고 deploy하려면 해당 yaml을 서버로 전송한 뒤 deploy하는 번거로움이 발생할 수 있습니다.

여기서는 하나의 로컬 컴퓨터에서 여러 kubernetes cluster로 접속하는 방법에 대해 알아볼 것입니다.

## `kubeconfig` 파일

`kubeconfig` 파일은 kubernetes의 cluster에 관한 정보를 기입하는 파일입니다.
`kubectl`은 여기서 cluster에 대한 접속정보를 가져오고, 이를 토대로 명령어를 전송하게 됩니다.

파일은 보통 `~/.kube` 폴더 내에 있으며 `config` 파일이 우리가 흔히 말하는 `kubeconfig` 파일입니다.
즉, `~/.kube/config` 파일을 열어보면 내용을 볼 수 있습니다.

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: C:\Users\USER\.minikube\ca.crt
    server: https://192.168.99.102:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: C:\Users\USER\.minikube\client.crt
    client-key: C:\Users\USER\.minikube\client.key
```

여기서 중요한 정보는 server IP, current-context

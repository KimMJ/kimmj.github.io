---
title: "Choose Cloud Providers"
menuTitle: "Choose Cloud Providers"
date:  2020-01-19T00:32:21+09:00
weight: 3
draft: false
tags: ["install", "spinnaker"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

Spinnaker를 배포할 환경을 설정해 주어야 합니다.
여기에서는 제가 구축한 local kubernetes cluster를 사용할 것입니다.

먼저 2가지가 필요합니다.
1. `kubeconfig` 파일
   `kubeconfig` 파일은 일반적으로 `~$HOME/.kube/config` 파일을 의미합니다.
   저는 local kubernetes cluster로 이동하여 해당 파일을 `halyard`를 위한 vm으로 복사하였습니다.
2. `kubectl` CLI 툴

이제 `hal config` 명령어를 통해 kubernetes cluster를 추가합니다.

```bash
hal config provider kubernetes enable

CONTEXT=$(kubectl config current-context)

hal config provider kubernetes account add wonderland \
    --provider-version v2 \
    --context $CONTEXT

hal config features edit --artifacts true
```
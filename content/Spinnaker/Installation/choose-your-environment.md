---
title: "Choose Your Environment"
menuTitle: "Choose Your Environment"
date:  2020-01-19T00:42:56+09:00
weight: 4
draft: false
tags: ["install", "spinnaker"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

Spinnaker를 배포하는 방법에는 3가지가 있습니다.
Kubernetes 환경에 배포하기, local debian으로 배포하기, local git으로 배포하기가 있습니다.

여기에서는 Kubernetes 환경에 배포하기를 진행할 것입니다.

```bash
ACCOUNT=wonderland
hal config deploy edit --type distributed --account-name $ACCOUNT
```

위와같이 설정하면 됩니다.
`ACCOUNT`는 kubernetes cluster를 추가할 때 사용했던 이름을 사용하면 됩니다.

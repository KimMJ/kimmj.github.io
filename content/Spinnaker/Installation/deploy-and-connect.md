---
title: "Deploy and Connect"
menuTitle: "Deploy and Connect"
date:  2020-01-19T01:20:12+09:00
weight: 6
draft: false
tags: ["spinnaker", "install"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

드디어 마지막 절차입니다.

먼저 어떤 버전을 설치할지 확인후 설정합니다.

```bash
hal version list
```

작성 기준으로 최신 버전이 `1.17.6`이므로 이를 설정합니다.

```bash
hal config version edit --version 1.17.6
```

`halyard`를 NodePort로 노출시키기 위해 `api`와 `ui`에 base url을 부여합니다.

```bash
hal config security ui edit --override-base-url http://192.168.8.22:30100
hal config security api edit --override-base-url http://192.168.8.22:30200
```

이제 본격적으로 deploy를 하도록 합니다.

```bash
hal deploy apply
```

그 후 Spinnaker를 NodePort로 서비스합니다.

```bash
kubectl patch svc spin-deck -n spinnaker --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'
kubectl patch svc spin-gate -n spinnaker --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'

kubectl patch svc spin-deck -n spinnaker --type='json' -p '[{"op":"replace","path":"/spec/ports/0/nodePort","value": 30100}]'
kubectl patch svc spin-gate -n spinnaker --type='json' -p '[{"op":"replace","path":"/spec/ports/0/nodePort","value": 30200}]'
```

이제 Spinnaker로 접속하여 확인합니다.
url은 http://<IP>:30100 입니다.

![Spinnaker](/images/Spinnaker/Install/spinnaker.png)

여기까지 했으면 Spinnaker를 Kubernetes에서 사용할 수 있습니다.
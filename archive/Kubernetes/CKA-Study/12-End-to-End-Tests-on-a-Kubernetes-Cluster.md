---
title: "12 End to End Tests on a Kubernetes Cluster"
date:  2020-06-15T19:39:06+09:00
weight: 12
draft: false
tags: ["kubernetse", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## End to End Tests

* Test - Manual
  * `kubectl get nodes`
  * `kubectl get pods -n kube-system`이 잘 되는지
  * `service kube-apiserver status`
  * `service kube-controller-manager status`
  * `service kube-scheduler status`
  * `service kubelet status`
  * `service kube-proxy status`
  * `kubectl run nginx`
  * `kubectl expose deployment nginx --port=80 --type=NodePort`
* Kubernetes에는 `test-infra`라는 프로젝트가 있다.
  * 약 1000개의 테스트를 해줌.
* `sonobuoy`

## End to End Tests - Run and Anlyze

* `go get`으로 소스를 받고 `skeleton`으로 하여 op-prem을 테스트할 수 있다.
* 그 다음 테스트 명령어를 입력하여 테스트를 할 수 있고 12시간정도 걸린다.

## Smoke test

* 일일히 입력하여 테스트하는 방법.

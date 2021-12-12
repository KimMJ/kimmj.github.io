---
title: "04 Logging and Monitoring"
date:  2020-04-18T21:47:08+09:00
weight: 4
draft: false
tags: ["kubernetes", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Monitor Cluster Componets

* 쿠버네티스에서 자체 제공하는 것은 없으나 다음과 같은 것들이 있다.
  * Metrics Server
  * Prometheus
  * Elastic Stack
  * DataDog
  * Dynatrace
* metric server는 각 쿠버네티스의 노드와 파드의 메트릭을 모아서 메모리에 저장한다.
* metric server는 유일한 in-memory monitoring solution이다.
  * 데이터를 저장하지 않아서 이전 자료를 보지 못한다.
* `kubelet`은 `cAdvisor`를 포함한다.
  * 파드로부터 퍼포먼스 메트릭을 수집하여 메트릭 서버로 전송한다.
* metric server가 설치되면 `kubectl top node`, `kubectl top pods`를 사용하여 메트릭을 볼 수 있다.

## Managing Application Logs

* 도커에서 stdout으로 로깅을 하는 상황
  * `docker logs` 명령으로 로그를 볼 수 있음.
* 파드에 여러개의 컨테이너가 있으면 하나를 지정해야 로그를 볼 수 있다.

---
title: "[번역] 쿠버네티스에서의 Port, TargetPort, NodePort"
date:  2020-03-15T23:13:37+09:00
weight: 10
draft: false
tags: ["kubernetes", "service", "targetport"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

원문: <https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ports-targetport-nodeport-service.html>

쿠버네티스의 port declaration 필드에는 여러가지가 있다.
각 type에 대해 빠르게 살펴보고 YAML에서 각각 어떤 의미를 가지고 있는지 알아보도록 하자.

## Pod `ports` list

`pod.spec.containers[].ports`로 정의된 이 배열은 container가 노출하고 있는 포트의 리스트를 나타낸다.
이 리스트를 꼭 작성해야할 필요는 없다.
리스트가 비어있다고 하더라도 container가 포트를 listening하고 있는 한 여전히 네트워크 접속이 가능하다.
이는 단순히 쿠버네티스에게 추가적인 정보를 줄 뿐이다.

> List of ports to expose from the container. Exposing a port here gives the system additional information about the network connections a container uses, but is primarily informational. Not specifying a port here DOES NOT prevent that port from being exposed. Any port which is listening on the default "0.0.0.0" address inside a container will be accessible from the network. Cannot be updated. - Kubernets API Docs

## Service `ports` list

서비스의 `service.spec.ports` 리스트는 서비스 포트로 요청받은 것을 파드의 어느 포트로 포워딩할 지 설정하는 것이다.
클러스터 외부에서 노드의 IP 주소와 서비스의 `nodePort`로 요청이 되면 서비스의 `port`로 포워딩되고, 파드에서 `targetPor`로 들어온다.

<script src="https://gist.github.com/matthewpalmer/9d9180c4980befe6b06babe98577eab9.js"></script>

### `nodePort`

이는 서비스가 쿠버네티스 클러스터 외부에서 노드의 IP 주소와 이 속성에 정의된 포트로 보일수 있도록 한다.
이 때, 서비스는 `type: NodePort`로 지정해야 한다
이 필드는 정의되어 있지 않을 경우 쿠버네티스가 자동으로 할당한다.

### `port`

서비스를 클러스터 안에서 지정된 포트를 통해 내부적으로 노출한다.
즉, 서비스는 이 포트에 대해서 보일 수 있게 되며 이 포트로 보내진 요청은 서비스에 의해 선택된 파드로 전달된다.

### `targetPort`

이 포트는 파드로 전달되는 요청이 도달하는 포트이다.
서비스가 동작하기 위해서는 어플리케이션이 이 포트에 대해 네트워크 요청을 listening을 하고 있어야 한다.

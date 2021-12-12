---
title: "08 Storage"
date:  2020-05-04T02:34:12+09:00
weight: 8
draft: false
tags: ["kubernetes", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Introduction to Docker Storage

* docker에는 두가지 컨셉의 stroage가 있다.
  * storage drivers
  * volume drivers

## Storage in Docker

* file system
  * `/var/lib/docker`에 docker가 사용하는 파일들이 있다.
* docker는 layered architecture라서 caching을 사용할 수 있다.
* 이렇게 해서 이미지를 실행시키면 해당 이미지에 사용된 layer들은 `Read Only`이다.
* 컨테이너 상에서 실행하는 것들은 container layer로, `Read Write`이다.
* `Read Only`상의 파일을 수정하면 도커는 자동으로 이를 복사해서 수정한다.
* 여기서 영구적으로 보관하고 싶은 것들이 있다면, volume을 사용해야한다.
* `/var/lib/docker/volumes`를 만드록 `docker run -v <volume>:<path>`의 형태로 사용한다.
  * `absolue path`를 통해서도 mount가 가능하다.
* strorage driver는 layer를 관리한다. (mount)

## Volume Driver Plugins in Docker

* volume은 volume drivers가 할당한다.
* default = local (`/var/lib/docker/volumes`)

## Container Storage Interface (CSI)

* 쿠버네티스에서 스토리지를 확장하는 방법.
* 쿠버네티스에서만 사용하는 규격이 아니다.

## Volumes

* 파드에 volume을 할당해서 container의 파일을 저장할 수 있다.
* 원하는 stroage type을 사용해서 특정 스토리지 서비스를 이용할 수 있다.

## Persistent Volumes

* 파드마다 volume을 할당하는 것은 매우 힘든 일이다.
* 좀 더 나은 방법: `Persistent Volumes` 사용하기
  * admin이 `pv`를 생성하고, `pod`는 필요시에 이를 가져다가 사용하기
  * `persistent volume claim`을 통해 요청한다.

## Persistent Volume Claims

* admin은 `Persistent Volume`을 생성하고, user는 `Persistent Volume Claim`을 생성하여 PV를 사용한다.
* PVC가 생성되면 쿠버네티스는 Bind 과정을 통해 알맞은 PV를 선택한다.
* 모든 PVC는 하나의 PV에 할당된다.
* label기반으로 bind도 할 수 있다.
* PVC를 했는데 bind될 것이 없으면 Pending 상태가 된다.
* default로 `persistentVolumeReclaimPolicy=Retain`이라서 PVC가 삭제되어도 PV의 데이터는 남아있다.
  * Reuse되지 않는다.
  * `Delete`로 설정하면 자동으로 삭제된다.
  * `Recycle`로 설정하면 다시 사용할 수 있지만 권장하지 않는다.

## Tips

* Storage Class나 StatefulSets에 관한 것은 CKAD의 범위.
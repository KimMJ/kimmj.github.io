---
title: "Create Vm With Ansible Libvirt"
date: 2020-01-08T01:52:47+09:00
tags: ["ansible", "libvirt"]
weight: 10
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

Ansible은 어떠한 프로세스를 자동화 할 때 사용할 수 있는 툴입니다.
그리고 libvirt는 linux 환경에서 qemu를 이용하여 VM을 생성할 때 사용하는 python 모듈입니다.

이 두가지를 합하여 Ansible을 통해 VM을 생성하는 방법에 대해 알아보도록 하겠습니다.

## ansible-role-libvirt-vm

참조 Github : [https://github.com/stackhpc/ansible-role-libvirt-vm](https://github.com/stackhpc/ansible-role-libvirt-vm)

위의 Github 프로젝트는 libvirt를 ansible에서 사용할 수 있도록 만든 오픈소스입니다.
이를 이용하여 ansible-playbook을 통해 VM을 생성해 볼 것입니다.

이를 로컬에 clone 합니다.

```bash
git clone https://github.com/stackhpc/ansible-role-libvirt-vm
```

## 테스트 환경

저는 Ubuntu 18.04.3 Desktop을 사용하고 있습니다.
그리고 설치에 사용될 iso는 제 포스트에서 작성한 적이 있었던 `preseed.cfg`를 이용한 자동 설치 이미지입니다.
따라서 이미지를 넣고 부팅만 하면 실행할 수 있습니다.

## play.yaml

<script src="https://gist.github.com/KimMJ/00c08cf5d11770ac08f17419f1a2fae9.js"></script>

저는 이러한 `play.yaml` 파일을 사용하였습니다.

여기서 `cdrom`을 사용하였는데, 이미지는 `baked-ubuntu.iso`를 사용하였습니다.

또한 장비들에 대한 설정을 xml로 추가적으로 하고싶어서 `xml_file`을 설정해 주었습니다.

`xml_file`또한 업로드 해두었습니다.

<script src="https://gist.github.com/KimMJ/0e74dea0e988b9072d26628fa69f0ee1.js"></script>

네트워크는 설정을 빼놓을 경우 설치중에 확인창이 발생하여 기본적으로 NAT를 사용하도록 하였습니다.
이는 필요에 따라 변경을 해야 합니다.
또한 `enable_vnc`의 경우 virt-manager를 통해 상황의 경과를 확인하고 싶어서 추가하였습니다.

위의 파일들을 workspace에 두시면 됩니다.

## Test

이렇게까지 한 뒤 `play.yaml`이 있는 위치에서 시작합니다.

그러면 `ansible-playbook`은 `ansible-role-libvirt-vm`이라는 role을 해당 위치에서 검색하고, 실행이 될 것입니다.

```bash
ansible-playbook play.yaml
```

실행 중 `sudo` 권한이 필요하다고 할 수도 있습니다. 이럴 경우 `sudo su`로 잠시 로그인 후 `exit`로 빠져나오시면 에러가 발생하지 않습니다.

## 확인

`virt-manager`를 통해 GUI 환경에서 실제로 잘 되고 있는지 확인할 수 있습니다.

```bash
virt-manager
```

`preseed.cfg`를 사용한 이미지라면 30초 후 설치 언어가 자동으로 영어로 설정이 되면서 계속해서 설치가 진행될 것입니다.

## 마치며

vm을 생성하는 일이 잦다면, 이 또한 굉장히 귀찮은 일이 아닐 수 없습니다. 
소규모가 아닌 대규모로의 확장성을 생각한다면 당연히 자동화를 하는 것이 올바른 접근이라고 생각합니다.

VM 설치 자동화의 방법이 여러가지가 있을 것이고 이 방법 또한 그 여러가지 방법 중 하나입니다.

더 좋은, 더 편한 방법이 있다면 알려주시면 감사하겠습니다.
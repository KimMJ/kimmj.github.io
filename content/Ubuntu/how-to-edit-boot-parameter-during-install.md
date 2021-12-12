---
title: "Ubuntu 설치 시 Boot Parameter를 수정하기"
date: 2020-01-08T01:52:12+09:00
tags: ["ubuntu", "install", "boot parameter"]
weight: 10
pre: "<i class='fas fa-minus'></i>&nbsp;"
---


Ubuntu 설치할 때 boot parameter가 필요한 상황이 간혹 발생할 수 있습니다.

특히 저의 경우, `preseed.cfg`를 수정하기 위해 인스톨러가 질의하는 것이 `preseed.cfg`의 어떤것과 대응이 되는지를 보기 위해 `DEBCONF_DEBUG=5`라는 옵션을 boot parameter로 주어야 했습니다.
이 때 사용할 수 있는 방법을 소개드립니다.

먼저 평소와 같이 ubuntu를 설치하기 위해 설치 이미지를 삽입합니다.
그 다음에는 언어를 선택하시면, 다음으로 넘어가기 전에 메뉴가 뜹니다.

이 상태에서 `F6`을 누르시면 옵션을 선택할 수 있고, 이 때 `ESC`키를 누르면 boot parameter가 하단에 보일 것입니다.
여기서 원하는 boot parameter를 입력하면 됩니다.

이 때, 위아래 방향키를 누르게 되면 입력했던 내용이 사라지게 됩니다.
따라서 미리 맨 위 install ubuntu에 커서를 올리고 수정하시기 바랍니다.

## install 시에 설정으로 넣어버리기

`preseed.cfg`로 미리 질문에 대한 답을 다 정할 수 있었듯이, boot parameter 또한 미리 설정할 수 있습니다.
해당 파일은 iso 파일을 압축해제 하였을 때, `/isolinux/txt.cfg` 파일 내에 있습니다.

```bash
grep -ri 'initrd' .
```

이렇게 검색해 보았을 때 `quiet ---`이라고 적힌 것들이 있는데, `---` 뒤에가 boot parameter로 쓰이는 것들입니다.

vim으로 `/isolinux/txt.cfg` 파일을 열고 원하는 설정을 기입하면 됩니다.

이렇게 원하는 boot parameter를 적었다면, 다시 `md5sum`을 통해 체크섬을 만들어주어야 합니다. 
이에 대한 내용은 앞선 [포스트]({% post_url 2020-01-05-unattended-ubuntu %})에서도 확인할 수 있으니 `bakeIsoImage.sh` 스크립트를 참조하여 `md5sum`을 하고 iso 파일을 만들면 됩니다.
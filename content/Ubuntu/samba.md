---
title: "Samba를 통한 디스크 공유"
date: 2020-11-28T00:42:06+09:00
draft: false
weight: 10
tags: ["samba"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`Samba`는 리눅스에 있는 폴더를 윈도우와 공유하기 위해 사용한다.
최초에 NFS로 구성했으나 NFS는 다음과 같은 문제점이 있었다.

1. 폴더 전체를 `rwx` 권한을 주게 만들거나 `nobody:nogroup`으로 모든 권한을 폴더에 줘야 했다.
2. `perforce`와 연동하려고 구성한 것이었는데 1번의 문제로 인해 자꾸 충돌이 발생했다.

이때문에 다른 대안을 찾던 중 `samba`를 알게 되었고, 이를 내 컴퓨터에도 적용해보았다.

## Samba 구성

### samba 설치

```sh
apt install samba -y
```

### samba 설정

`/etc/samba/smb.conf` 파일을 열어 수정한다.

```sh
[workspace] # 여기서 workspace는 접근할 때 사용되는 이름
comment = workspace path
path = /path
read only = no
ritable = yes
browseable = yes
guest ok = yes
available = yes
public = yes
valid users = root
```

### user 추가

실제 `passwd`에 등록된 계정으로 설정해야 한다.

```sh
$ smbpasswd -a root
New SMB password:
Retype new SMB password:
Added user root.
$ service smbd restart
```

### 네트워크 드라이브 연결

주소: \\<서버 IP>\workspace

이런식으로 네트워크 드라이브를 연결하면 될 것이다.
`workspace`는 위에서 `/etc/samba/smb.conf`에서 사용한 이름으로 대체될 수 있다.

---
title: "Netplan으로 static IP 할당받기"
date: 2020-01-11T01:12:57+09:00
draft: false
weight: 10
tags: ["ubuntu-18.04", "netplan", "static-ip"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## 유선 static IP 할당

다음과 같이 static-IP-netplan.yaml을 작성합니다.

```yaml
network:
  version: 2
  ethernets:
    enp3s0:
      dhcp4: no
      dhcp6: no
      addresses: [ 192.168.1.26/24 ]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [ 8.8.8.8, 8.8.4.4 ]
```

하나씩 살펴보도록 하겠습니다.

1. `ethernetes`: 유선랜 설정입니다.
2. `enp3s0` 설정을 사용할 랜카드입니다.
3. `dhcp4`, `dhcp6`: dynamic으로 IP를 할당받는 dhcp를 disable한 것입니다.
4. `addresses`: 사용할 IP 및 CIDR입니다.
5. `gateway4`: IP가 사용하는 gateway입니다.
6. `nameservers`: dns 주소입니다. `8.8.8.8`과 `8.8.4.4`를 사용합니다.

## WIFI static IP 할당

```yaml
  wifis:
    wlp2s0:
      dhcp4: no
      dhcp6: no
      addresses: [ 192.168.8.26/24 ]
      gateway4: 192.168.8.30
      nameservers:
        addresses: [ 8.8.8.8, 8.8.4.4 ]
      access-points:
        "my-SSID":
          password: "my-password"
```

하나씩 살펴보도록 하겠습니다.

1. `wifis`: wifi에 대한 설정입니다.
2. `wlp2s0`: 무선 랜카드입니다.
3. `access-points`: 사용할 wifi에 대한 설정입니다.
4. `"my-SSID"`: wifi의 SSID 즉, wifi 이름입니다.
5. `password`: 해당 wifi의 비밀번호를 적으면 됩니다.

만약 ubuntu server를 사용한다면, wifi 관련 패키지는 초기에 설치되지 않습니다.
따라서 `wpasupplicant`를 설치해주어야 합니다.

```bash
sudo apt install wpasupplicant
```

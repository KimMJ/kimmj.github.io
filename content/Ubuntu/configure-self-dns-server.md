---
title: "Configure Self Dns Server"
menuTitle: "Configure Self Dns Server"
date:  2020-03-03T02:06:15+09:00
weight: 10
draft: true
tags: [""]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Steps

### install bind

```bash
sudo apt-get install -y bind9
```

### `/etc/resolv.conf` 수정

```yaml
name server 192.168.8.24
```
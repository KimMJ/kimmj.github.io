---
title: "열려있는 포트 확인하기"
date:  2020-02-24T13:38:04+09:00
weight: 10
draft: false
tags: ["linux", "port", "network"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## 열려있는 포트 확인하기

```bash
# 방법 1
lsof -i -nP | grep LISTEN | awk '{print $(NF-1)" " $1}' | sort -u
# 방법 2
netstat -tnlp
```

## 열려있는 포트 확인하기 + 관련된 프로세스 이름 확인하기

```bash
netstat -tnlp | grep -v 127.0.0.1 | sed 's/:::/0 /g' | sed 's/[:\/]/ /g' | awk '{print $5"\t"$10}' | sort -ug
```

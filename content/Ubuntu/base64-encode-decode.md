---
title: "Ubuntu에서 Base64로 인코딩, 디코딩하기"
date:  2020-02-27T20:44:42+09:00
weight: 10
draft: false
tags: ["ubuntu", "base64"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Encode

### echo로 입력하기

```bash
$ echo "password" | base64
cGFzc3dvcmQK
```

### Control+D를 누를때까지 입력하기

```bash
$ base64
admin
password
^D # Control+D
# result
YWRtaW4KcGFzc3dvcmQK
```

## Decode

### echo로 입력하기

```bash
$ echo "cGFzc3dvcmQK" | base64 --decode
password
```

### Control+D를 누를때까지 입력하기

```bash
$ base64 --decode
YWRtaW4KcGFzc3dvcmQK
^D # Control+D
# result
admin
password
```
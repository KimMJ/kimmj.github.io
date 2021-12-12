---
title: "Editor(vi)가 없을 때 파일 수정하기"
date:  2020-02-26T17:59:29+09:00
weight: 10
draft: false
tags: ["ubuntu", "file", "editor"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## echo로 파일 내용을 입력하는 방법

### `>`로 파일 덮어쓰기

```bash
$ cat file
asdfasdfasdf
$ echo "asdf" > file
$ cat file
asdf
```

### `>>`로 파일에 이어쓰기

```bash
$ cat file
asdf
$ echo "asdf" >> file
$ cat file
asdf
asdf
```

## cat으로 파일 입력하는 방법

### `>`로 파일 덮어쓰기

```bash
$ cat file
asdf
$ cat > file
aaaa
bbbb
^D # Command+D
$ cat file
aaaa
bbbb
```

### `>>`로 파일에 이어쓰기

```bash
$ cat file
asdf
$ cat >> file
aaaa
bbbb
^D # Command+D
$ cat file
asdf
aaaa
bbbb
```

### <<EOF로 EOF을 입력하면 입력 완료하기

```bash
$ cat file
asdf
$ cat <<EOF > file
aaaa
bbbb
EOF
$ cat file
asdf
aaaa
bbbb
```

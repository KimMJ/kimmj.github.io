---
title: "Tmux"
menuTitle: "Tmux"
date:  2020-01-23T21:04:29+09:00
weight: 10
draft: false
tags: ["tmux", "ubuntu"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## `tmux`란?

`tmux`는 하나의 화면에서 여러개의 터미널을 키고싶을 때 사용하는 프로그램으로, ubuntu를 설치하면 기본적으로 설치되는 프로그램입니다.

다음과 같은 구조를 가집니다.

```text
tmux  
├── session  
│   ├── windows  
│   │   ├── pane  
│   │   └── pane  
│   └── windows  
│       ├── pane  
│       └── pane  
└── session  
    ├── windows  
    │   ├── pane  
    │   └── pane  
    └── windows  
        ├── pane  
        └── pane  
```

## session 사용법

먼저 가장 큰 단위인 session을 다루는 방법부터 시작해보도록 하겠습니다.

### session 생성

```bash
tmux
```

위처럼 `tmux`를 생성할 수 있습니다.
이 경우 `tmux` session의 이름은 0부터 차례로 증가하는 숫자로 정의됩니다.

여기에 session의 이름을 사용자가 정의할 수도 있습니다.

```text
tmux new -s <my-session>
```

이렇게 이름을 지어놓으면 용도에 따른 session을 구분할 때 좋습니다.

### session에 접속하기

이번에는 이미 생성된 session에 접속하는 방법입니다.

```text
tmux a
tmux a -t <my-session>
tmux attach
tmux attach -t <my-session>
```

`-t` 옵션을 줘서 이름을 지정할 수도 있고, (임의로 생성된 숫자 또한 마찬가지입니다.) 
옵션없이 실행할 경우 가장 최근 열린 session으로 접속합니다.

### session 확인하기

session어떤 것들이 있는지 보려면 `ls`를 이용하면 됩니다.

```text
tmux ls
```

또는 이미 session에 들어간 상태에서, session들의 리스트를 봄과 동시에 미리보기 화면으로 어떤 작업중이었는지도 볼 수 있습니다.

```text
[prefix] s
```

여기서 [prefix]는 일반적으로 `Ctrl+b`를 의미하며, 사용자에 의해 변경될 수 있습니다. 

### session에서 빠져나오기

session을 빠져나오는 방법에는 두가지가 있습니다.

1. 완전히 session을 로그아웃하여 session 삭제하기
2. session이 계속 돌아가는 상태에서 빠져나오기

1번의 경우는 모든 pane에서 log out을 하면 되므로 생략하도록 하겠습니다.
두번째 session이 계속 돌아가는 상태에서 빠져나오는 방법은 다음과 같습니다.

```text
[prefix] d
```

### session 죽이기

`tmux` 바깥에서 session을 없애려면 들어가서 로그아웃을 통해 끄는 방법도 있을테지만,
`kill-session`이라는 명령어를 통해서도 session을 없앨 수 있습니다.

```text
tmux kill-session -t <my-session>
```

### session 이름 바꾸기

session에 들어가 있는 상태에서 현재 session의 이름을 변경할 수 있습니다.

```text
[prefix] $
```

그러면 상태표시줄에서 session의 이름을 정해줄 수 있습니다.

## windows 사용법

windows는 session내의 tab과 같은 존재입니다.
session은 큰 사용 목적으로 묶어준다면 windows는 그에따라 tab으로 관리할 필요가 있을 때 사용하면 좋습니다.
물론 session과 pane만 가지고 사용해도 되지만, 그보다 더 유연하게 하려한다면 windows도 알아두는 것이 좋습니다.

### windows의 생성

windows를 관리하고자 한다면, 우선 session에 들어가 있는 상태여야 합니다.

```text
[prefix] c
```

위처럼 windows를 생성하고 나면 아래 tmux 상태표시줄에 `0:bash- 1:bash*`와 같은 형태로 windows가 추가됨을 볼 수 있습니다.
여기서 `*`는 현재 사용중인 windows를 의미합니다.

### windows 움직이기

먼저 기본적으로 앞, 뒤로 움직이는 방법입니다.

```text
[prefix] n # next window
[prefix] p # previous window
```

session에서 미리보기를 사용하여 순회했던 것처럼, windows도 list들을 미리보기형식으로 순회할 수 있습니다.

```text
[prefix] w
```

### windows 이름 변경하기

특정 window에 들어가있는 상태에서 해당 window의 이름을 변경할 수 있습니다.

```text
[prefix] ,
```

### windows 죽이기

현재 window를 죽이려면 로그아웃을 하는 방법도 있지만, 강제로 죽이는 방법도 존재합니다.

```text
[prefix] &
```

## panes 다루기

pane이란 `tmux`의 화면을 분할하는 단위입니다.
`tmux`를 사용하는 가장 큰 이유라고 볼 수 있습니다.

### pane 분할하기

```text
[prefix] % # vertical split
[prefix] " # horizontal split
```

위와같은 방법으로 pane을 생성할 수 있습니다.

### pane 이동하기

먼저 기본적으로 방향키를 이용하여 움직일 수 있습니다.

```text
[prefix] <방향키>
```

일정시간이 지나면 pane내에서의 방향키 입력으로 전환되어 pane을 움직일 수 없으므로 빠르게 움직여줍니다.

### pane 위치 바꾸기

pane들을 rotate하는 방법입니다.

```text
[prefix] ctrl+o # 시계방향으로 회전
[prefix] alt+o # 반시계방향으로 회전
```

또는 하나를 이동할 수도 있습니다.

```text
[prefix] { # move the current pane left
[prefix] } # move the current pane right
```

### pane 크게 보기

pane을 여러개 쓰다가 하나만 크게 보고싶은 경우가 있을 수 있습니다.

```text
[prefix] z
```

돌아가는 방법 또한 같은 명령어를 통해 할 수 있습니다.

```text
[prefix] z
```

### pane을 새로운 window로 분할하기

특정 텍스트를 마우스로 복사하거나 여러가지 상황에서 새로운 window로 분할하는 것이 편한 경우가 있습니다.

```text
[prefix] !
```

### 모든 pane에서 동시에 입력하기

`tmux`를 통해서 여러개의 서버에 `ssh` 접속을 한 뒤, 동시에 같은 입력을 하게 할 수도 있습니다.

```text
[prefix] :set synchronize-panes yes (on)
[prefix] :set synchronize-panes no (off)
```

## Tips

`tmux`에는 기본 내장된 layout이 있습니다.
이를 통해서 pane들을 resize하지 않고 기본 형식에 맞게 쉽게 변경이 가능합니다.
그 중 가장 자주 사용할 수 있는 것은 다음 두가지입니다.

```text
[prefix] alt+1 # 수직
[prefix] alt+2 # 수평
```

## Reference

* https://gist.github.com/MohamedAlaa/2961058
* https://gist.github.com/andreyvit/2921703
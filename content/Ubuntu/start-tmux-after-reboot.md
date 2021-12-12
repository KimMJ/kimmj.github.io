---
title: "reboot 후에 tmux를 실행시켜 원하는 작업을 하기"
date:  2020-02-29T14:00:40+09:00
weight: 10
draft: false
tags: ["tmux", "reboot", "ubuntu"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`tmux`는 terminal을 한 창에 여러개 띄울 때 사용하는 프로그램입니다.

이 프로그램의 특징은 detach 모드로 들어가면, 어디서든 terminal에 접속하여 해당 session에 접속했을 때, 그 화면 그대로를 가져올 수 있다는 것입니다.

즉, 원격 접속을 통해 서버에 접속했을 때 작업을 돌려놓고 detach모드로 들어가면 나의 session을 꺼도 실제 서버에서는 해당 작업이 계속해서 돌아가고 있다는 것입니다.
퇴근하기 전 시간이 오래걸리는 작업을 돌려놓고 가야할 때 유용하게 사용할 수 있습니다.

저의 경우는 제 로컬 컴퓨터에서 `hugo`를 통해 사이트를 생성하여 블로그를 편집할 때마다 즉시 그 결과를 보고 있습니다.
이 떄 사용되는 명령어는 간단하지만, 매번 컴퓨터를 켤 때마다 이를 수행해주어야 한다는 것은 여간 귀찮은 일이 아닙니다.
따라서 다음과 같은 프로세스로 `tmux`를 생성하여 제가 하는 작업을 수행하도록 설정할 것입니다.

## overview

1. `tmux` session을 이름을 지정하여 생성하고, daemon으로 돌린다.
2. 해당 session에 직접 접속하지 않고 명령어를 전달한다.
3. 1, 2의 작업을 스크립트로 만들고 재부팅 시 실행하도록 한다.

## 절차

### sh script 작성

다음과 같은 스크립트를 작성합니다.
(`tmux-on-reboot.sh`)

```zsh
#!/bin/zsh

SESSIONNAME="script"
tmux has-session -t $SESSIONNAME 2> /dev/null

if [ $? != 0 ] 
  then
    tmux new-session -s $SESSIONNAME -n script -d "bin/zsh"
    tmux send-keys -t $SESSIONNAME "cd /home/wanderlust/Ibiza" C-m
    tmux send-keys -t $SESSIONNAME "hugo server --bind 0.0.0.0 --port 8000 --disableFastRender" C-m
fi
```

여기서 `C-m`은 enter 명령을 주기 위함입니다.

그 다음 해당 파일에 실행권한을 줍니다.

```bash
chmod +x tmux-on-reboot.sh
```

### 재부팅 시 실행하도록 설정

이를 위해서는 `crontab`을 사용할 것입니다.

`crontab`을 사용하는 방법은 `crontab -e`를 통해 root 권한으로 명령어를 실행하는 방법과 `/etc/crontab`을 수정하여 원하는 유저를 부여하는 방법이 있습니다.
여기서는 제가 사용할 계정을 가지고 생성하는 것을 해보도록 하겠습니다.

```bash
vi /etc/crontab
```

vim이 열리면 가장 아랫줄에 다음과 같이 추가합니다.

```crontab
@reboot wanderlust /home/wanderlust/scripts/tmux-on-reboot.sh
```

저장 후 재부팅하여 실행되는지 확인합니다.

## Reference

* <https://superuser.com/a/440082?>
---
title: "oh-my-zsh에서 home key와 end key가 안될 때 해결방법"
date:  2020-02-29T02:27:25+09:00
weight: 10
draft: false
tags: ["oh-my-zsh", "zsh"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

oh-my-zsh을 설치하고 원격접속이나 로컬환경에서 터미널에 접속했을 때 `home key`와 `end key`가 먹히지 않는 경우가 있습니다.

이런 경우에 사용하는 terminal에서 `home key`와 `end key`를 눌러 실제 어떤 값이 전달되는지 확인한 후, 이를 `beginning-of-line`, `end-of-line`으로 설정하면 해결할 수 있습니다.

## 해결법

1. `home key`가 되지 않는 terminal에 접속합니다.
2. `Control+V`를 누릅니다.
3. 문제가 되는 `home key`를 누릅니다.
4. terminal에 뜬 문자를 기록합니다.
5. `~/.zshrc`에 다음과 같이 추가합니다.
   여기서 `case`에 관한 부분은 상황에 따라 넣지 않거나 변경해야 합니다.
   ```zshrc
   case $TERM in (xterm*)
   bindkey '^[[H' beginning-of-line
   bindkey '^[[F' end-of-line
   esac
   ```
6. `source ~/.zshrc`를 하거나 새로운 session을 열어서 확인합니다.


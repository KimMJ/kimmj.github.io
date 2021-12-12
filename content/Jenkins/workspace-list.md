---
title: "Workspace@2를 변경하기 - Workspace List 설정 변경"
date:  2020-04-16T16:23:22+09:00
weight: 10
draft: false
tags: ["jenkins", "workspace"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

운용하는 노드에 executor가 2개 이상이라면, concurrent build 옵션을 disable했다고 하더라도
zombie process가 있을 경우 `workspace@2`처럼 `@` 캐릭터가 들어간 workspace를 사용할 수 있다.

workspace 안에 특정한 파일을 넣고 사용하는 경우라면 `@2`가 생기면 안될 것이다.
그러나 이는 그렇게 좋은 방법은 아닌것 같으며 이런 경우에는 github 등에 스크립트같은 파일을 옮겨놓고
`git pull`이나 `git` 관련 플러그인을 통해 다운로드 한 뒤 사용하는 것이 더 좋은 방법인 것 같다.

하지만 나의 경우 `Perforce`를 사용하고 있었느데 `p4 sync`에서 문제가 발생했다.
`@`라는 문자가 없어야 한다는 것이었다.
사실 저 문자만 없다면, workspace의 폴더 이름이 무엇이든 상관이 없었기 때문에 `@`대신 다른 캐릭터를 사용하기로 결정했다.

`JAVA_OPTS= '-Dhudson.slaves.WorkspaceList=A'`를 추가하면 된다.
내 경우 `docker-compose`를 통해 jenkins를 설치했으므로 다음과 같이 작성했다.

```yaml
environments:
    JAVA_OPTS= '-Dhudson.slaves.WorkspaceList=_'
```

파일에서 `envieonments`의 정확한 위치는 때에 따라 다를 수 있으므로 검색을 통해 environments의 정확한 위치를 확인해보고 설정하면 될 것 이다.
위와 같이 설정하면 두번째 workspace는 `workspace_2`와 같이 생길 것이다.

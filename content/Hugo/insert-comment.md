---
title: "Hugo에 Comment 추가하기 (Utterance)"
menuTitle: "Hugo에 Comment 추가하기 (Utterance)"
date:  2020-01-24T20:56:47+09:00
weight: 10
draft: false
tags: ["hugo", "utterance"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## 댓글 서비스 선택

블로그를 운영하는데 관심을 가지기 시작하면서, 기본적으로 jekyll이나 hugo에는 댓글 기능이 없다는 것을 알게 되었습니다.
static site를 만드는데 사실 댓글을 지원한다는게 이상한 상황이긴 하지요.
그래도 서드파티의 지원을 받으면 댓글 기능이 가능해집니다.
여러 블로그들을 탐방하며 git page 기능을 사용하는 블로그들에도 댓글이 있는것을 항상 봐왔으니까요.

따라서 댓글을 어떻게 사용하는지 검색해보게 되었습니다.
대표적인 것이 `Disqus` 입니다.
실제로 많은 사이트들이 `Disqus`를 기반으로 댓글 기능을 사용합니다.

저는 이 hugo 기반 블로그를 만드는 데 큰 도움을 준 [https://ryan-han.com/post/etc/creating_static_blog/](https://ryan-han.com/post/etc/creating_static_blog/)를 보고 `Utterance`에 대해 접하게 되었으며
개발자에게는 너무나도 친숙한 깃허브 기반이라는 점이 끌려서 이 서비스를 선택하게 되었습니다.

## 적용 방법

적용하는 방법은 너무나도 쉽습니다.

0. 미리 댓글을 위한 레파지토리를 생각해둡니다(기존에 있는 레파지토리 혹은 댓글만을 위한 레파지토리). 
   저는 사이트를 렌더링해주는 `kimmj.github.io` 레파지토리로 선택하였습니다.
1. [https://utteranc.es/](https://utteranc.es/)에 접속합니다.
2. 중간쯤에 보이는 `utterances app` 하이퍼링크를 클릭합니다.
3. 해당 앱을 적절한 레파지토리에 설치합니다.
4. 설치한 레파지토리를 1번에서 접속한 사이트에 양식에 맞게 기입합니다.
5. `Blog Post ↔️ Issue Mapping` 섹션에서 적절한 것을 선택합니다. 저는 default 옵션을 사용했습니다.
6. `Issue Label`은 댓글로 생성된 이슈에 label을 달지, 단다면 어떤 label을 달지 선택하는 것입니다.
   저는 comment로 작성했습니다.
7. `Theme`을 선택합니다. 저는 default 옵션을 사용했습니다.
8. 하단에 생성된 코드를 복사합니다.
9. hugo에서 댓글이 보이게 될 위치에 8번에서 복사한 코드를 붙여넣습니다.
   저의 경우 `custom-comment.html`이라는 파일에 붙여넣기 했습니다.
10. 사이트에 제대로 뜨는지 확인합니다.

## 후기

이렇게 hugo에 댓글 기능을 추가하였습니다. 
생각보다 너무나도 간단했네요.
이 댓글 기능을 하자고 생각하고 나서 적용까지 5분정도 걸렸던 것 같습니다.
댓글도 너무나 친숙한 레이아웃이라 정감이 가는 것 같네요.

#### Reference 
* https://github.com/Integerous/TIL/blob/master/ETC/Hugo%2BGithub_Page.md
* https://utteranc.es/
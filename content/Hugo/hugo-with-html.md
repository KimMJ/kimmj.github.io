---
title: "HUGO로 HTML이 되지 않을 때 가능하게 하는 방법"
menuTitle: "hugo with html"
date: 2020-01-10T02:22:34+09:00
draft: false
tags: ["hugo", "html"]
weight: 10
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`Hugo`는 markdown을 기본적으로 사용하지만 html을 이용해서 좀 더 다양하게 커스터마이징이 가능한 장점도 가지고 있습니다.

하지만 저는 처음에 html 코드를 사용하게 되면 `<!-- raw HTML omitted -->`와 같은 줄로 대치가 되곤 했습니다.
구글링 결과 이는 `Hugo`의 버전이 0.60.0으로 되면서부터 기본적으로 disable 시켰기 때문입니다.

따라서 다음과 같이 조치를 하면 간단하게 해결이 가능합니다.

```toml
[markup.goldmark.renderer]
unsafe= true
```

위와 같은 설정을 `config.toml`에 추가하기만 하면 됩니다.
추가를 한 뒤 다시 확인해보면 정상적으로 html 코드가 적용된 모습을 볼 수 있습니다.
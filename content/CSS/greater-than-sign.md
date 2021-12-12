---
title: "Greater Than Sign"
menuTitle: "Greater Than Sign"
date:  2020-02-01T19:56:45+09:00
weight: 10
draft: false
tags: ["css"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## `>`의 의미

`>`는 [child-combinator](https://www.w3.org/TR/selectors/#child-combinators) 입니다.

다음의 예시를 통해 정확히 어떤 역할을 하는지 알아보도록 하겠습니다.

```html
<div>
    <p class="some_class">Some text here</p>     <!-- Selected [1] -->
    <blockquote>
        <p class="some_class">More text here</p> <!-- Not selected [2] -->
    </blockquote>
</div>
```

위와 같은 예시에서 `div > p.some_class`는 `div` 바로 밑에 있는 `p.some_class`만을 선택합니다.
`<blockquote>`로 감싸진 `p.some_class`는 선택되지 않습니다.

이와는 다르게 `space`만 사용하는 [descendant combinator](https://www.w3.org/TR/selectors-3/#descendant-combinators)는 두개의 `p.some_class` 모두를 선택합니다.

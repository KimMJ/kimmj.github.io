---
title: "background image 어둡게 하기"
date:  2020-03-10T23:40:25+09:00
weight: 10
draft: false
tags: ["css", "bgimg-darken"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

배경 이미지를 삽입했는데 사진이 너무 밝아 어둡게 필터처리를 넣고 싶은 경우가 있을 수 있습니다.
저의 경우 logo에 제 깃허브 프로필사진을 빼고 노을진 풍경을 넣었는데 사진이 너무 밝아 부자연스러운 느낌이 들었습니다.

이 때 검색 후 다음과 같이 조치를 하여 어두워지는 효과를 줄 수 있었습니다.

```css
#sidebar #header-wrapper {
    background-image: linear-gradient( rgba(0, 0, 0, 0.3), rgba(0, 0, 0, 0.3) ),url(../images/sunset_on_ibiza.jpg);
}
```

`linear-gradient` 속성은 선형으로 gradient를 적용하는 속성인데 이를 활용하여 검정색 필터를 넣었습니다.
필수 파라미터로 두개의 색깔이 필요하고, 이를 각각 검정색(`rgb(0, 0, 0)`)에 투명도를 `0.3`으로 설정한 `rgba(0, 0, 0, 0.3)`으로 설정하여 이미지가 전체적으로 어둡게 보이도록 설정한 것입니다.

여담으로, 파워포인트에서 사진을 어둡게 하여 텍스트를 강조하고자 할 때 많이 이용했던 검정색 반투명 박스를 이용하는 것과 비슷한 방법이라고 느껴지네요.
---
title: "Font Change"
menuTitle: "Font Change"
date:  2020-01-12T16:28:31+09:00
weight: 10
draft: false
tags: ["font", "hugo"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

`Ibiza` 프로젝트를 진행하는데 폰트가 마음에 들지 않았습니다.
따라서 저는 폰트를 변경하기로 마음먹었습니다.

먼저, 폰트 설정을 어디서 하는지 알아낼 필요가 있었습니다.

```bash
find . | grep font
```

결과를 보니, theme 폴더 안에 제가 사용하는 `hugo-theme-learn` 테마에서 `static/fonts/` 폴더에 폰트들을 저장해두고 있었습니다. 그렇다면 어느 파일에서 어떤 폰트를 사용한다고 설정할까요?

`hugo-theme-learn`폴더로 이동하여 어디에 사용되는지 확인해보았습니다.

```bash
grep -ri "font"
```

결과가 길게 나오는데요, 여기서 `static/css/theme.css` 안에 폰트에 대한 설정을 한 것이 보였습니다.
그 파일을 보니, `@font-face`라는 설정이 보이네요.
여기서 `Work Sans`라는 폰트를 불러오고 있었습니다.

이 폰트를 `Noto Sans CJK KR`이라는 폰트로 바꾸려고 합니다.
따라서 먼저 폰트를 다운로드 받아야 합니다.

다운로드 페이지 : https://www.google.com/get/noto/#sans-kore

여기에서 다운로드 버튼을 눌러 폰트를 다운받습니다.

```bash
curl -o noto-mono.zip https://noto-website-2.storage.googleapis.com/pkgs/NotoSansCJKkr-hinted.zip
```

이를 `my-custom-theme` 폴더 내의 `static/fonts` 폴더 안에다가 압축해제할 것입니다.

```bash
mv noto-mono.zip mj-custom-theme/static/fonts
unzip noto-mono.zip
rm noto-mono.zip README
```

그러고나서 `font-face` 설정을 바꾸어 보도록 하겠습니다.
처음에는 이 설정으로 폰트가 정말 바뀌는지 확인해보기 위해 현재 사용중인 폰트의 url 부분을 `Noto Mono` 폰트로 변경해보았습니다.

```css
@font-face {
    font-family: 'Work Sans';
    font-style: normal;
    font-weight: 500;
    src: 
        url("../fonts/NotoSansMonoCJKkr-Bold.otf?#iefix") format("embedded-opentype"), 
        url("../fonts/NotoSansMonoCJKkr-Bold.otf") format("woff"), 
        url("../fonts/Work_Sans_500.woff2") format("woff2"), 
        url("../fonts/Work_Sans_500.svg#WorkSans") format("svg"), 
        url("../fonts/Work_Sans_500.ttf") format("truetype");
}
```

하지만 예상과 다르게 변경되지 않았습니다.
확인해보니 이는 `font-weight`이라는 설정때문이었습니다.
`body`에 대한 `font-weight`은 300으로 설정이 되어있었고, 따라서 제가 설정한 폰트가 아닌 `font-weight: 300`인 폰트를 선택하게 된 것입니다.

다시한번 `body`쪽의 `font-weight`을 500으로 바꾸어 실험해보았습니다.

```css
body {
    font-family: "Work Sans", "Helvetica", "Tahoma", "Geneva", "Arial", sans-serif;
    font-weight: 500;
    line-height: 1.6;
    font-size: 18px !important;
}
```

그러자 제가 원하는 폰트로 변경이 된 것을 확인하였습니다.
다시 위로 돌아가서 `@font-face` 설정을 저의 폰트 이름으로 변경하고 제가 원래 하려던 폰트로 변경하였습니다. `font-weight: 300`으로 다시 돌려놓았고, 새로운 폰트를 `font-weight: 300`으로 주었습니다.

```css
@font-face {
    font-family: 'Noto Mono Sans CJK KR ';
    font-style: normal;
    font-weight: 300;
    src: 
        url("../fonts/NotoSansMonoCJKkr-Regular.eot?#iefix") format("embedded-opentype"), 
        url("../fonts/NotoSansMonoCJKkr-Regular.woff") format("woff"), 
        url("../fonts/NotoSansMonoCJKkr-Regular.woff2") format("woff2"), 
        url("../fonts/NotoSansMonoCJKkr-Regular.svg#NotoSansMonoCJKkr") format("svg"), 
        url("../fonts/NotoSansMonoCJKkr-Regular.ttf") format("truetype");
}
```

여기서 보시면 `format` 속성들이 많은 것을 볼 수 있습니다.
이는 브라우저별로 지원하는, 지원하지 않는 폰트들에 대해서 처리를 해주기 위한 것입니다.
저는 폰트를 다운로드 받았을 때 `otf` 포멧밖에 없었습니다. 따라서 다른 포멧들로 변경해 줄 필요가 있었습니다.

font converter : https://onlinefontconverter.com/

위 사이트에서 제가 필요한 `eot`, `woff`, `woff2`, `svg`, `ttf` 파일들로 변환후 저장했습니다.

이제 `body`의 css에서 `font-family` 맨 앞에 앞서 정의한 폰트를 지정해줍니다.

```css
body {
    font-family: "Noto Sans Mono CJK KR", "Work Sans", "Helvetica", "Tahoma", "Geneva", "Arial", sans-serif;
    font-weight: 300;
    line-height: 1.6;
    font-size: 18px !important;
}
```

확인해보니 제대로 적용이 되었네요.

#### before font change

![before font change](/images//Hugo/Ibiza/before-font-change.png)

#### after font change

![before font change](/images//Hugo/Ibiza/after-font-change.png)

#### Reference

* https://wit.nts-corp.com/2017/02/13/4258
* https://aboooks.tistory.com/153
* https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/webfont-optimization?hl=ko
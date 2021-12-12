---
title: "Hugo에 Google Analytics 적용하기"
date:  2020-03-09T23:19:18+09:00
weight: 10
draft: false
tags: ["hugo", "google-analytics"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

google analytics는 내 블로그를 사용하는 사람들이 얼마나 많은지, 어떤 정보를 보는지 확인할 수 있는 서비스입니다.
무료로 사용할 수 있고, 사용 방법도 어렵지 않기 때문에 search console과 더불어 사용하면 좋은 서비스로 보입니다.

그래서 저도 제 블로그에 google analytics를 설정할 수 있는지 찾아보던 중 다음과 같은 글을 확인하고, 이를 통해 설정할 수 있었습니다.

<https://discourse.gohugo.io/t/implementing-google-analytics-in-hugo/2671/2>

## 적용 방법

### `config.toml`의 수정

hugo에서는 config 파일을 사용하여 사용자의 설정정보를 보관합니다.
저는 `config.toml`을 사용하고 있었는데 여기에다가 다음과 같이 추가하면 됩니다.

```toml
googleAnalytics = "UA-123-45"
```

만약 yaml이라면 `googleAnalytics: "UA-123-45"` 정도로 설정하면 될 것 같습니다.

여기서 해당 코드를 github에 올릴 때 안전하게 올리고 싶으시다면, [git-secret](/git/git-secret)을 참조하시기 바랍니다.

### `header.html`에 추가

자신이 사용하는 theme에서 `layouts/partials`로 이동하면 header.html이라는 파일이 있을 것입니다.
제가 사용중인 `Learn` 테마에서는 `custom-header.html`이라는 파일로 사용자가 추가하기를 원하는 내용을 적을 수 있는 파일이 있었습니다.
따라서 저는 그 파일에 다음과 같이 추가해주었습니다.

```hugo
{{ template "_internal/google_analytics.html" . }}
```

또는 async로 이용하길 원한다면 다음과 같이 적으면 됩니다.

```hugo
{{ template "_internal/google_analytics.html" . }}
```

## 확인

google analytics는 하루에 한번 데이터를 전송합니다.
따라서 하루정도가 지난 후에 google analytics 페이지를 들어가면 통계자료를 열람할 수 있습니다.

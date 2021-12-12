---
title: "14 Other Topics"
date:  2020-06-15T21:19:17+09:00
weight: 14
draft: false
tags: ["kubernetes", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---


## Advanced Kubectl Commands

* JSON PATH를 사용하는 이유는 많은 정보들 속에서 특정한 데이터를 필터링하기 위함이다.
* `kubectl` command를 사용하면 `kube-apiserver`와 통신한다.
  * 이는 json 포멧으로 `kubectl`에게 응답하고, 이를 사람이 읽을 수 있게 보여주는 것이다.
  * `detail`을 보려면 `-o wide`를 주면 된다.
* JSON PATH를 사용하면 보고싶은 결과만 볼 수 있다.
* 이를 사용하려면 다음과 같은 절차가 필요하다.
  * `kubectl` command로 raw data를 볼 수 있어야 한다.
  * `-o json`을 통해 json output을 확인할 수 있어야 한다.
  * JSON PATH query를 작성한다.
  * `-o=jsonpath=`를 통해 query를 전달한다. 이 때 `'{ }'`로 감싸야 한다.
* 이 때 여러개의 query를 넣을 수도 있다.
* 중간에 `{"\n"}`을 넣어서 이름과 데이터를 구분할 수 있다.
* Loop을 사용하여 루프를 돌 수 있다.
  * `{range .items[*]}`를 사용하면 된다.
  * 마지막에 `{end}`를 넣는다.
  * 또한 `custom-columns`도 사용할 수 있다.
    * `-o=custom-columns=<COMLUMN NAME>:<JSON PATH>`
    * `-o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu`
* Sort도 사용할 수 있다.
  * `--sort-by=<query>`

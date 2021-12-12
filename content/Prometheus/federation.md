---
title: "Federation"
menuTitle: "Federation"
date:  2020-01-21T23:34:44+09:00
weight: 10
draft: false
tags: ["prometheus", "federation"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## What is Federation

영어 의미 그대로는 "연합"이라는 뜻입니다.
즉, Prometheus의 Federation은 여러개의 Prometheus에서 Metric을 가져와
계층구조를 만드는 것을 의미합니다.

![prometheus-federation-on-aws](/images/Prometheus/prometheus-federation-on-aws.png?width=70pc)

위의 그림에서 너무나도 잘 표현이 되어 있습니다.
그림에서 보시면 상위에 있는 Prometheus에서 하위의 Dev, Staging, Production쪽으로 화살표가 간 것을 볼 수 있습니다.
이는 아래에 있는 Prometheus가 `http(s)://<url>/federation`으로 보여주는 Metric들을
위쪽에 있는 Prometheus에서 scrape하기 때문입니다.

저의 상황을 설명해드리고 지나가도록 하겠습니다.
저는 Kubernetes Cluster가 Dev(Canary), Staging, Production과 비슷하게 3개 있었습니다.
여기서 Spinnaker를 통해 Dev에 새로운 이미지들을 배포할 것이고,
이에 대한 Metric을 Canary Analysis를 통해 분석하여 Dev로 배포된 이미지가
이전 Staging의 이미지와 어떻게 다른지 등을 점수화하여 Staging 서버에 배포를 할지 말지 결정하도록 해야하는 상황이었습니다.

이 때, Spinnaker의 설정상 Prometheus를 연동하고 나서 Canary Config를 설정할 때 하나의 Prometheus만 바라보도록 할 수 있었습니다.
따라서 여러대의 Promethus의 Metric을 비교하기 위해서는 여러대의 Prometheus가 가지고 있는 Metric을 상위개념의 Prometheus가 scrape하도록 해야했습니다.
어떻게 해야하는지 검색해본 결과 Federation이라는 기능이 있는 것을 알게 되었습니다.

## How to configure Prometheus Federation

### Prerequisites

우선 여러대의 Prometheus가 필요합니다.
그리고 이를 하나로 모아줄 또 다른 Prometheus가 필요합니다.

저의 경우 `docker-compose`를 통해 Prometheus를 구동하여 다른 서버의 Prometheus Metric을 가져오도록 설정하였습니다.
Install에 관해서는 다른 문서를 참고해 주시기 바랍니다.

### Configuring federation

다음은 공식 사이트에 나온 federation의 구성입니다.

```yaml
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'

    static_configs:
      - targets:
        - 'source-prometheus-1:9090'
        - 'source-prometheus-2:9090'
        - 'source-prometheus-3:9090'
```

#### `job_name`

job_name은 static_configs[0].targets에 적힌 Prometheus Metric에 어떤 `job="<job_name>"`을 줄지 결정하는 것입니다.
이를 통해 저는 Canary, Stage 서버를 구분하였습니다.

#### `scrape_interval`

얼마나 자주 Metric을 긁어올 지 결정하는 것입니다.

#### `metrics_path`

어떤 path에서 Metric을 가져오는지 설정합니다.
보통의 경우 federation으로 설정하면 됩니다.

#### `params`

실제로 `<PrometheusUrl>/federation`으로 접속해보면 아무것도 뜨지 않습니다.
`/federation`은 `param`로 매칭이 되는 결과만 리턴하며, 없을 경우 아무것도 리턴하지 않습니다.
따라서 이는 필수 필드이고, 원하는 `job`만 가져오게 하거나 `{job=~".+"}`와 같은 방법으로 모든 Metric을 가져오게도 할 수 있습니다.

#### `static_configs`

어떤 Prometheus에서 Metric을 가져올지 결정하는 것입니다.

### Validation

위와같이 설정을 한 뒤, Prometheus에 접속하여 Targets에 들어가 봅니다.
리스트에 설정한 `job_name`들이 떠있고, `UP`인 상태로 있으면 정상적으로 구성이 된 것입니다.

#### Reference

<https://prometheus.io/docs/prometheus/latest/federation/>

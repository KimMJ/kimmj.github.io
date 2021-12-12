---
title: "Canary Analysis"
menuTitle: "Canary Analysis"
date:  2020-01-21T01:08:00+09:00
weight: 10
draft: false
tags: ["canary", "canary-update", "spinnaker"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Spinnaker Canary Analysis

Spinnaker에는 Canary Analysis라는 자동 분석 도구가 있습니다.
`Kayenta`라는 micro service를 사용하는데, 이를 통해 자동으로 canary deploy가 괜찮은 버전인지를 확인해 줍니다.

그러나 이 툴은 Spinnaker에서 사용하기에 여간 어려운 것이 아닙니다.
제일 먼저 봉착하는 난관은 바로 "어떻게 Canary Analysis를 활성화 하는가?"입니다.

[이곳](https://www.spinnaker.io/setup/canary/)에 방법이 나와있지만, 사실 저도 엄청 많이 헤멨습니다.
저는 bare-metal 환경에서 Kubernetes cluster를 구축하였었고, aws나 azure, gcp는 사용하지 못하는 상황었습니다.
(물론 지금도 집에서 VM으로 로컬에 구성하였지만, cloud platform은 언제나 과금때문에 꺼려지게 됩니다.)

이런 상황에서 어떻게 Canary Analysis를 활성화했는지부터, 어떻게 이를 통해 Metric을 비교하는지까지 한번 알아보도록 하겠습니다.

### Prerequisites

첫번째로 metric service를 선택해야 합니다.
여러가지가 있을 수 있겠지만, 저는 로컬에서 사용할 수 있는 Prometheus를 사용할 것입니다.

두번째로 가져온 metric의 결과들을 저장해 놓을 storage service가 필요합니다.
저는 Spinnaker를 구성할 때 minio를 storage service로 구축을 했었으므로,
여기에서도 마찬가지로 minio를 통해 데이터를 저장할 것입니다.

### How to enable Canary Analysis

제일 먼저 `hal` command를 통해서 Canary Analysis를 활성화시켜야 합니다.

```bash
hal config canary enable
```

그다음엔 Prometheus를 canary analysis에 사용되도록 설정할 것입니다.

```bash
hal config canary prometheus enable
hal config canary prometheus account add my-prometheus --base-url http://192.168.8.22/9090
```

이처럼 Prometheus 콘솔창이 보이는 url을 입력하면 됩니다.
저는 `docker-compose`를 통해서 `9090` port로 expose 시켰으므로 위와같이 적어주었습니다.

그 다음에는 metric provider를 설정합니다.
앞서 말했듯이 여기서는 Prometheus를 사용할 것입니다.

```bash
hal config canary edit --default-metrics-store prometheus
hal config canary edit --default-metrics-account my-prometheus 
```

위의 두 설정으로 Prometheus가 default metric store로 선택되었습니다.
이는 물론 나중에 canary configuration에서 원하는 것으로 선택할 수도 있습니다.
또한 default account도 `my-prometheus`라는 이름으로 선택해 주었습니다.

이번엔 default storage account를 설정할 것입니다.
공식 docs에서는 `minio`에 관련된 설정방법이 잘 나와있지 않습니다.

하지만 [hal command](https://www.spinnaker.io/reference/halyard/commands/#hal-config-artifact-s3-account-edit)를 잘 보시면
어떻게 해야할지 감이 약간은 잡히실 것입니다.

`--api-endpoint`에는 `minio`의 url을 적고, `--aws-access-key-id`에는 `minio`의 ID였던 `minio`를 입력합니다.
그리고 `--aws-secret-access-key`는 `minio`의 PW인 `minio123`을 입력합니다.

```bash
hal config artifact s3 account add my-minio \
    --api-endpoint http://192.168.8.22:9000 \
    --aws-access-key-id minio \
    --aws-secret-access-key minio123
```

그 뒤에는 위에서 입력했던 `my-minio` storage account를 Canary Analysis에 사용하도록 설정하면 됩니다.

```bash
hal config canary edit --default-storage-account my-minio
```

### Validate

제대로 설정을 마쳤으면 pipeline의 config에서 Feature 탭에 Canary가 추가된 것을 볼 수 있습니다.
이를 활성화하면 본격적으로 Canary Analysis를 사용할 수 있습니다.

#### Reference
https://www.spinnaker.io/guides/user/canary/  
https://www.spinnaker.io/guides/user/canary/config/
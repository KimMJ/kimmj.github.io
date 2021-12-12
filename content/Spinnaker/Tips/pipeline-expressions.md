---
title: "Pipeline Expressions"
date: 2020-01-10T01:33:32+09:00
draft: false
weight: 10
tags: [ "spinnaker", "pipeline" ]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

Spinnaker는 배포를 자동화할 때 사용합니다.
그렇기 때문에 자동화를 위해선 다른 곳에서 사용된 값들을 가지고 와야할 필요성이 생기기도 합니다.

이 문서에서는 그럴 때 사용할 수 있는 pipeline function에 대해 알아보도록 하겠습니다.

## pipeline functions

### pipeline에서 다른 pipeline의 값들 불러오기

{{% notice note %}}
Note: Pipeline expression syntax is based on Spring Expression Language (SpEL).
{{% /notice %}}

위의 Note에도 적었듯이, Spinnaker는 SpEL을 기반으로 Expressions를 사용합니다.
SpEL에 대해 이미 잘 알고있다면 너무나도 좋겠지만, 저는 익숙하지가 않았기 때문에 많은 시행착오를 거쳐서 습득을 하게 되었습니다.

기본적으로 `${ expression }`의 형태를 가지게 됩니다.

여기서 한가지 기억해 두어야 할 것은 nested가 되지 않는다는 것입니다.
즉, `${ expression1 ${expression2} }`가 되지 않습니다.

### 언제 pipeline expression을 사용하나요

pipeline expression은 Spinnaker UI로는 해결할 수 없는 문제들을 해결하여줍니다.
예를 들어 특정 stage가 성공했는지의 여부에 따라 stage를 실행할지, 말지 결정하는 방법을 제공해 줍니다.
또는 가장 최근에 deploy된 pod를 알아낸다거나, spinnaker를 통한 canary analysis를 할 때 비교할 두가지 대상을 선택하기 위해 사용할 수도 있습니다.

Spinnaker는 모든 파이프라인을 `JSON` 형태로도 관리할 수 있기 때문에, UI에는 없는 값들도 입력할 수 있습니다.
이렇게 좀 더 유연한 방법으로 Spinnaker를 이용하고자 한다면 pipeline expression은 꼭 알아두어야 합니다.

### 원하는 값을 어떻게 찾나요

pipeline이 구동되고 나면, Details를 누르고 Source를 눌렀을 때 해당 pipeline의 실행결과가 json형태로 출력됩니다.
이를 `VS Code`나 다른 편집기를 이용하여, json으로 인식하게 한 뒤, 자동 들여쓰기를 하면 보기 좋게 만들어줍니다.

이를 통해서 어떤 값을 내가 사용할 지 확인하여 pipeline expression을 작성하면 됩니다.

### 내가 작성한 pipeline expression은 어떻게 테스트하나요

작성한 pipeline expression을 테스트하기 위해 파이프라인을 구동한다는 것은 끔직한 일입니다.
Spinnaker는 이를 테스트하기 위해 API endpoint를 제공합니다.
즉, 파이프라인을 다시 구동시키지 않고도 어떤 결과값이 나오는지 확인할 수 있다는 것을 의미합니다.

테스트 방법은 간단합니다. 다음과 같이 `curl`을 통해 endpoint로 테스트하면 됩니다.

```bash
PIPELINE_ID=[your_pipeline_id]
curl http://api.my.spinnaker/pipelines/$PIPELINE_ID/evaluateExpression \
       -H "Content-Type: text/plain" \
       --data '${ #stage("Deploy").status.toString() }'
```

여기서 `api.my.spinnaker`는 `Gate`의 Service를 보고 포트를 참조하여 작성하면 됩니다. 기본값은 `localhost:8084`입니다.
이렇게 하면 `Deploy`라는 stage가 성공했을 때 다음과 같은 결과를 볼 수 있습니다.

```json
{"result": "SUCCEEDED"}
```

Spinnaker가 expression을 통해 결과를 만들어내지 못한다면 다음과 같이 에러와 로그가 발생합니다.

```json
{
  "detail":
    {
      "{ #stage(\"Deploy\").status.toString() ":
        [
          {
            "description":
              "Failed to evaluate [expression] Expression
              [{ #stage( #root.execution, \"Deploy\").status.toString() ]
              @0: No ending suffix '}' for expression starting at character 0:
              { #stage( #root.execution, \"Deploy\").status.toString() ",
            "exceptionType":"org.springframework.expression.ParseException",
            "level":"ERROR",
            "timestamp":1531254890849
         }
        ]
   },
  "result":"${#stage(\"Deploy\").status.toString() "
}
```

## Reference

Spinnaker Docs: <https://www.spinnaker.io/guides/user/pipeline/expressions/>

---
title: "[번역] What Is Infrastructure as a Code? How It Works, Best Practices, Tutorials"
menuTitle: "[Translate] What Is Infrastructure as a Code"
date:  2020-02-09T20:06:17+09:00
weight: 10
draft: false
tags: ["IaC", "infrastructure-as-code"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

link: <https://stackify.com/what-is-infrastructure-as-code-how-it-works-best-practices-tutorials/>

과거에 IT infrastructure를 관리하는 것은 힘든 일이었다.
시스템 관리자는 수동으로 관리하고 어플리케이션을 구동시키기 위해 모든 하드웨어와 소프트웨어를 설정해야 했다.

하지만 최근 몇년간 급격하게 상황들이 바뀌었다.
cloud computing같은 트렌드가 디자인, 개발, IT infrastructure의 유지를 하는 방법을 혁명화하고 발전시켰다.

이러한 트렌드의 핵심 요소는 `infrastructure as code`이다.
여기에 대해 이야기 해보도록 하겠다.

## Defining Infrastructure as Code

infrastructure를 코드로 정의하는 것부터 시작해보도록 하자.
이것이 무엇을 의미하는지, 어떤 문제들을 해결하는지 배우게 될 것이다.

Wikipedia에서 IaC는 다음과 같이 정의되어 있다.

> Infrastructure as code is the process of managing and provisioning computer data centers through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

> Infrastructure as code는 물리적인 하드웨어 설정이나 상호작용을 하는 설정 툴을 이용하는 것이 아닌, 기계가 읽을 수 있는 파일로 정의하여 computer data center들을 관리하고 프로비저닝하는 프로세스이다.

이 정의는 나쁘지 않지만 약간 너무 장황하게 설명했다.
더 간단하게 해보자.

> Infrastructure as code (IaC)는 IT infrastructure를 설정 파일로 관리한다는 의미이다.

다음 질문은 이렇게 될 것이다.
"왜 그걸 쓰고싶어 하나요?"

## What Problem Does IaC Solve?

"what"에 대해 생각하기 전에 "why"에 먼저 집중해보자.
왜 IaC가 필요한가?
이것이 어떤 문제를 풀어주는가?

### The Pain of Managing IT Infrastructure

역사적으로 IT infrastructure를 관리하는 것은 수동 프로세스였다.
사람들은 물리적으로 서버를 위치시키고 이를 설정했다.
머신이 OS와 어플리케이션이 원하는 설정으로 되었을 때에, 어플리케이션을 배포할 수 있게 된다.
당연하게도 수동 프로세스는 자주 문제를 일으킨다.

첫번째 큰 문제는 **비용**이다.
네트워크 엔지니어에서부터 하드웨어 관리 기술자까지 많은 전문가를 고용해서 프로세스의 각 단계를 수행시키도록 해야한다.
이런 모든 사람에게 돈을 지불해야 하면서 또한 관리되어야 한다.
이는 관리 오버헤드를 야기하고 기업 내의 소통의 복잡성을 증대시킨다.
결과적으로? 돈이 사라진다.
또한 우린 아직 비용을 더 많이 늘리는 데이터센터를 관리하는 것과 빌딩에 대한 이야기를 하지 않았다.

그 다음 문제는 **확장성**과 **가용성**이다.
하지만 결국 모든 것은 속도 문제다.
수동으로 설정하는 것은 너무 느리고, 어플리케이션의 접속은 스파이크를 치고 있는 동안 시스템 관리자는 부하를 관리하기 위해 필사적으로 서버를 세팅할 것이다.
이는 가용성에도 무조건 영향을 미친다.
기업이 백업 서버나 심지어 데이터 센터가 없다면 어플리케이션은 장기간동안 이용불가능해질 것이다.

마지막으로 가장 중요한 문제는 inconsistency이다.
몇몇 사람들이 수동으로 설정을 배포한다면, 균열은 생길수밖에 없다.

## Cloud Computing: A Cure?

Cloud computing은 방금 읽었던 고통들을 완화시켜준다.
이는 데이터센터를 구축하고 유지보수하는 것과 그 비용으로부터 자유롭게 해준다.

그러나 Cloud computing은 만병통치약과는 거리가 멀다.
이것이 infrastructure를 빠르게 셋업하는데에는 도움을 주겠지만 (그래서 고가용성이나 확장성의 문제는 해결해 준다) 이는 inconsistency 이슈를 해결하지는 못한다.
설정을 수행하는 사람이 한명보다 많다면, 불균형이 생길 수 있다.

## Infrastructure as Code: The Missing Piece of the Puzzle

IaC 정의를 다시 한번 봐보자.

> Infrastructure as code (IaC)는 IT infrastructure를 설정 파일로 관리한다는 의미이다.

정의에서 가져온 핵심은 바로 다음과 같다.
IaC 이전에 IT 사람들은 infrastructure에 대해 수동으로 설정을 바꾸어야 했다.
아마 스크립트를 쓰거나 몇몇 작업은 자동화를 시켰겠지만, 그냥 그 정도였다.
IaC를 통해 infrastructure의 설정은 코드 파일의 형태를 띄게 되었다.
이는 단순한 텍스트이지만 수정하고 복사하고 분배하는 것이 쉽다.
당신은 다른 소스코드 파일들처럼 source control로 이를 관리할 수 있고 또 그래야 한다.

## Infrastructure as Code Benefits

이제까지 수동으로 infrastructure를 관리하는 것의 문제점들을 알아 보았다.
우리는 여기서 어떻게 cloud computing이 이런 몇가지 문제점을 해결해 주는지, 또 어떤것들은 해결해주지 않는지 알아보았다.
그리고 우리는 IaC가 퍼즐의 마지막 조각이라고 말하며 이 논의를 마쳤다.

이제 우리는 IaC solution을 사용할 때의 이점에 대해서 알아볼 것이다.

### Speed

IaC의 가장 중요한 이점은 스피드이다.
Infrastructure as Code는 script를 실행시켜 빠르게 완전한 infrastructure를 셋업할 수 있게 해준다.
이를 개발환경과 production 환경부터 staging, QA 등등까지 모든 환경에 대해서 할 수 있다.
IaC는 전체 소프트웨어 개발 라이프사이클을 효과적으로 만들어준다.

### Consistency

수동 프로세스는 실수를 불어일으키고 시간이 걸린다.
사람은 실수할 수 있다.
우리의 기억은 잘못될 수 있다.
소통은 어렵고 우리는 일반적으로 소통을 어려워한다.
여기서 읽었던 것 처럼 수동으로 인프라를 관리하는 것은 얼마나 열심히 하던지 간에 불균형을 일으키게 된다.
IaC는 믿을 수 있는 하나의 소스코드로 이 설정 파일을 관리하여 이를 해결해준다.
이런 방식으로 동일한 설정이 어떤 불균형도 없이 계속해서 배포됨을 보장한다.

### Accountability

이는 빠르고 쉬운 것이다.
IaC 설정 파일을 소스 코드 파일처럼 버전화 할 수 있기 때문에 설정들의 변경사항을 추적할 수 있다.
누가 이를 했고 언제 했는지 추측할 필요가 없다.

### More Efficiency During the Whole Software Development Cycle

infrastrucutre as code를 적용하면 infrastructure 아키텍쳐를 어려 단계로 배포할 수 있다.
이는 전체 소프트웨어 개발 라이프 사이클을 더욱 효율적으로 해주어 팀의 생산성을 더 끌어올릴 수 있다.

프로그래머들이 IaC를 사용하여 sandbox 환경을 생성하여 독립된 공간에서 안전하게 개발할 수 있게 할 수 있다.
같은 방식으로 테스트를 돌리기 위해 production 환경을 복사해야 하는 QA 전문가들도 사용할 수 있다.
결과적으로 배포 시에 infrastructure as code는 하나의 단계가 될 것이다.

### Lower Cost

IaC의 주된 장점 중 하나는 의심할 여지 없이 인프라 관리의 비용이 줄어든다는 것이다.
IaC를 통해 cloud computing을 하면 극적으로 비용을 줄일 수 있다.
이는 하드웨어에 많은 시간을 사용하지 않아도 됨을 의미하고 이를 관리할 사람을 고용하지 않아도 되며 저장할 물리적인 장소를 만들거나 대여하지 않아도 됨을 의미한다.
하지만 IaC는 기회비용이라 부르는 다른 미묘한 방식으로 비용을 더 줄여준다.

보았듯이 똑똑하고, 높은 급료를 받는 전문가가 자동화 할 수 있는 작업을 수행하는 것은 돈 낭비이다.
이제 모든 포커스는 기업에 더 가치있는 일에 맞춰져야 한다.
그리고 자동화 전략을 사용하면 편하다.
이를 사용하여 수동적이고 느리고 에러가 나기 쉬운 작업을 수행하는 것으로부터 엔지니어들을 해방시켜주고 더 중요한 일에 집중할 수 있게 한다.

## How Does IaC Work?

IaC 툴은 어떻게 작동하는지가 굉장히 다양하지만 이를 일반적으로 두가지 종류로 나눠볼 수 있다.
하나는 imperative approach를 따르는 것이고 다른 하나는 declarative approach를 따르는 것이다.
이 위의 카테고리를 프로그래밍 언어 패러다임과 엮을 수 있다면 완벽하다.

imperative approach는 순서를 제공하는 것이다.
이는 명령어나 지시사항의 순서를 정의하여 인프라가 최종적인 결과를 가지게 하는 것이다.

declarative approach는 완하는 결과를 정의하는 것이다.
명백하게 원하는 결과로 가는 단계들의 순서를 정의하지 않고, 어떻게 최종 결과가 보여야 하는지만 정의한다.

## Learn Some Best Practices

이제 IaC 전략의 모범사례를 확인해 볼 것이다.

* 코드가 하나의 믿을 수 있는 소스로부터 나온다.
  명시적으로 모든 인프라 설정을 설정 파일 안에 작성해야 한다.
  설정 파일은 인프라 관리 문제에 대해서 단 하나의 관리포인트이다.
* 모든 설정 파일에 대해 Version Control을 하라.
  이는 말할 필요도 없겠지만, 모든 코드는 source control이 되어야 한다.
* 인프라 스펙에 대한 약간의 문서화만 필요하다.
  이 포인트는 첫번째의 논리적인 결과이다.
  설정 파일이 단일 소스이므로, 문서화가 필요하지 않다.
  외부 문서는 실제 설정과 싱크가 잘 안맞을 수 있지만 설정 파일은 그럴 이유가 없다.
* 설정을 테스트하고 모니터한다. IaC는 코드로 모든 코드와 같이 테스트될 수 있다.
  따라서 가능하면 테스트해라.
  IaC에 대한 테스트와 모니터링을 하여 production에 어플리케이션을 배포하기 전에 서버에 문제가 있는지 확인할 수 있다.

## Resources At Your Disposal

다음은 IaC를 배우기 좋은 유용한 리소스들이다.

* [Wikipedia’s definition](https://en.wikipedia.org/wiki/Infrastructure_as_code)
* [Edureka’s Chef tutorial](https://www.edureka.co/blog/chef-tutorial/)
* [Ibexlabs’s.The Top 7 Infrastructure As Code Tools For Automation](https://www.ibexlabs.com/top-7-infrastructure-as-code-tools/)
* [TechnologyAdvice’s Puppet vs. Chef: Comparing Configuration Management Tools](https://technologyadvice.com/blog/information-technology/puppet-vs-chef/)

## Infrastructure as Code Saves You Time and Money

IaC는 DevOps 움직임의 중요한 부분이다.
cloud computing이 많은 수동 IT 고나리의 문제점을 해결하는 데 첫 번째 단계라고 본다면 IaC는 다음의 논리적인 단계가 될 것이다.
이는 cloud computing의 모든 가능성을 열어주고 개발자와 다른 전문가들로부터 수동적이고 에러가 발생하기 쉬운 업무를 없애준다.
또한 소프트웨어 개발 라이프사이클의 효율성을 늘리고 비용을 절감한다.
IaC와 함께 Retrace같은 툴을 쓰는것도 좋다.
Retrace는 코드 레벨의 Application Performance Manager 솔루션으로 전체 개발 라이프사이클에서 어플리케이션의 퍼포먼스를 관리하고 모니터하게 할 수 있다.
에러 추적이나 로그 관리, 어플리케이션 메트릭과 같은 또한 많은 다른 기능들을 가지고 있다.

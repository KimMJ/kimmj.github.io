---
title: "[번역]Python Project 구조화하기"
date:  2020-10-18T18:16:40+09:00
weight: 10
draft: true
tags: ["python"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

링크: <https://docs.python-guide.org/writing/structure/>

"구조"란 *프로젝트를 어떻게 그 목적에 잘 부합하도록 할지*에 대해 생각하고 결정한 것들을 의미한다.
우리는 어떻게 Python의 기능들을 잘 이용하여 코드를 clean하고 effective하게 할 수 있을지에 대해 생각해 봐야 한다.
실질적으로 "구조"는 logic과 dependency가 명확하며 filesystem 상에 폴더와 파일들이 잘 정리된 `clean code` 를 만드는 것을 의미한다.

어떤 function이 어떤 module로 들어가야 하는가?
프로젝트 상에서 data flow는 어떻게 흐르는가?
어떤 feature와 function이 함께 그룹지어져야 하고 분리되어야 하는가?
이러한 질문에 대해 답을 내리며 넓은 관점에서 우리의 완성된 제품이 보일 모습을 계획하는 것을 시작할 수 있을 것이다.

이 섹션에서 우리는 프로젝트에서 구조를 적용하는데 핵심적이라 할 수 있는 Python의 module과 import 시스템을 면밀히 들여다 볼 것이다.
그 다음 다양한 방면으로 확장성 있고 잘 테스트 된(`extendend and test reliably`) 코드를 빌드할 수 있는지 논의해 볼 것이다.

## Structure of the Repository

### It's Important

Code Style, API Design, Automation 과 같은 것들이 건강한 개발 사이클을 위해서 필수적인 것처럼 Repository 구조는 프로젝트의 `architecture` 의 핵심적인 부분이다.

잠재적인 사용자 또는 contributor가 우리의 repository 페이지에 들어왔을 때, 다음과 같은 것들을 확인할 것이다.

* 프로젝트의 이름
* 프로젝트의 설명
* 여러 O' 파일들 --> ?

그들이 스크롤을 fold 아래로 내렸을 때에야 우리 프로젝트의 `README` 가 보일 것이다.

만약 우리의 repository가 많은 dump 파일이나 복잡한 nested 디렉토리 구조라면 그들은 우리가 열심히 작성한 documentation을 한 줄 조차 안읽고 다른 곳을 찾게될지도 모른다.

> Dress for the job you want, not the job you have.

물론, 첫 인상이 전부는 아니다.
당신과 당신의 동료는 이 repository에 셀수 없이 많은 시간을 쏟아부을 것이고, 결국엔 구석구석 모든 것들이 친숙해질 것이다.
repository의 layout은 중요하다.

### Sample Repository

**tl;dr:** 이건 [Kenneth Reitz가 2013년에 제안](https://kenreitz.org/essays/repository-structure-and-python)한 것이다.

이 Repository는 [GitHub에서 이용 가능](https://github.com/kennethreitz/samplemod)하다.

```text
README.rst
LICENSE
setup.py
requirements.txt
sample/__init__.py
sample/core.py
sample/helpers.py
docs/conf.py
docs/index.rst
tests/test_basic.py
tests/test_advanced.py
```

세부사항을 확인해보자.

### 실제 Module

* Location: `./sample/` 또는 `./sample.py`
* Purpose: 관심 코드

module package는 repository의 핵심적인 부분이다.
이를 감춰두면 안된다.

```text
./sample/
```

만약 module이 하나의 파일이라면 repository의 root 안에 바로 넣어도 된다.

```text
./sample.py
```

이 경우 라이브러리는 다른 소스코드에서 사용되거나 python subdirectory에 포함되지 않는다.

### License

* Location: `./LICENSE`
* Purpose: 법적 고지

License는 단언컨대 소스 코드 자체와는 별개로 repository 내에서 가장 중요한 부분이다.
전체 license text와 copyright claim이 이 파일 안에 있어야 한다.

만약 어떤 license를 사용해야 하는지 정확하게 모르겠다면 [choosealicense.com](http://choosealicense.com/)을 이용해보면 좋다.

물론 license 없이 코드를 배포해도 되지만 이는 잠재적으로 우리의 코드를 사용할 수도 있는 사람들을 막는 꼴이 된다.

### Setup.py

* Location: `./setup.py`
* Purpose: 패키지와 배포 관리

module package가 repository의 root에 있다면 이 파일 또한 root에 있어야 한다.

### Requirements File

* Location: `./requirements.txt`
* Purpose: development dependency

[pip requirements file](https://pip.pypa.io/en/stable/user_guide/#requirements-files)은 repository의 root에 있어야 한다.
이 파일에는 project에 기여(testing, building, document 생성)를 할 때 필요한 dependency를 작성해야 한다.

프로젝트에 development dependency가 존재하지 않거나 `setup.py`를 통해서 개발 환경을 세팅하는 것을 선호한다면, 이 파일은 없어도 된다.

### Documentation

* Location: `./docs/`
* Purpose: package reference documentation

이 파일이 다른 곳에 있어야 할 이유가 없다.

### Test Suite

test를 작성하는 것에 대한 조언은 [Testing Your Code](https://docs.python-guide.org/writing/tests/)를 참조한다.

* Location: `./test_sample.py` 또는 `./tests`
* Purpose: package integration과 unit test

처음 시작할 때는 간단한 test suite들이 하나의 파일로 존재할 것이다.

```text
./test_sample.py
```

test suite들이 커지게 되면, text를 다음과 같이 directory로 옮길 수 있다.

```text
tests/test_basic.py
tests/test_advanced.py
```

이러한 test module들은 테스트 할 packeged module들을 import 해야 한다.
import 하는 방법에는 몇가지가 있다.

* package가 `site-packages` 안에 설치된다고 가정한다.
* 간단한(하지만 명백하게) path를 수정하여 package를 적절하게 resolve 할 수 있도록 한다.

나는 후자를 강력하게 권장한다.
개발자가 `setup.py develop` 을 통해 능동적으로 codebase를 변경해야 하고 codebase의 각 instance에 대해 독립된 환경을 구축하도록 하는것이 필요하다.

각 test에 context를 import하기 위해 `tests/context.py` 파일을 생성한다.

```python
import os
import sys
sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))

import sample
```

그면 각 test module 안에서는 다음과 같이 import 될 것이다.

```python
from .context import sample
```

이 방법은 설치 방법에 관계 없이 항상 원하는 대로 동작한다.

어떤 사람들은 module 안에 test를 넣어야 한다고 주장한다.
그러나 나는 동의하지 않는다.
이는 사용자로 하여금 복잡도를 증가시키게 한다.
많은 test suite들이 추가적인 dependency나 runtime context들을 필요로 하기 때문이다.

### Makefile

* Location: `./Makefile`
* Purpose: 일반적인 management task

대부분의 나의 프로젝트나 Pocoo 프로젝트를 보게 된다면 Makefile이 있는 것을 알게 될 것이다.
이 프로젝트들이 C로 작성된 것이 아닌데도 왜 그럴까?
간단히 말해 `make` 는 project에 관한 일반적인 작업들을 정의하는 데 매우 유용한 툴이기 때문이다.

#### Sample Makefile

```makefile
init:
    pip install -r requirements.txt

test:
    py.test tests

.PHONY: init test
```

다른 일반적인 관리 목적의 스크립트들(예를 들면 `manage.py`나 `fabfile.py`)이 이 repository의 root로 포함될 수도 있다.

### Regarding Django Applications

`Django` 1.4 이후로 새로운 `Django` application의 트렌드를 발견했다.
많은 개발자들이 새로운 bundled application template 때문에 repository를 잘 구조화 하지 못한다.

어떻게 하냐고?
그들은 아무것도 없는 repository에서 다음과 같이 입력한다.

```sh
$ django-admin.py startproject samplesite
```

결과적으로 repository는 다음과 같은 구조가 된다.

```text
README.rst
samplesite/manage.py
samplesite/samplesite/settings.py
samplesite/samplesite/wsgi.py
samplesite/samplesite/sampleapp/models.py
```

이렇게는 하지 마라.

반복되는 path는 tool과 개발자 모두를 혼란스럽게 한다.
불필요한 nesting은 누구에게도 도움되지 않는다(monolithic SVN repo의 향수를 느끼고자 하지 않는 한).

이렇게 하자.

```sh
$ django-admin.py startproject samplesite .
```

"`.`"이 있음을 주목하라.

결과는 다음과 같을 것이다.

```text
README.rst
manage.py
samplesite/settings.py
samplesite/wsgi.py
samplesite/sampleapp/models.py
```

## Code의 구조가 핵심이다

Python이 import를 하는 방법과 모듈을 다루는 방식 덕분에 Python 프로젝트를 구조화 하는 것은 상대적을 쉽다.
여기서 쉽다는 의미는 제약사항같은 것이 많지 않고 module importing model을 파악하기 쉽다는 의미이다.
그래서 우리는 우리의 프로젝트의 다른 부분들과 그것들의 상호작용을 만드는 순수한 architectural task만 하면 된다.

프로젝트를 구조화하기 쉽다는 것은 잘못하기 쉽다는 의미 또한 내포한다.
잘못된 구조를 가진 프로젝트는 다음과 같은 특징이 있다.

* 많고 난잡한 circular dependency가 존재한다:
  `furn.py` 파일 안 에 있는 `Table` 과 `Chair` 클래스가 `workers.py` 파일 안에 있는 `Carpenter` 를 import 하여 `table.isdoneby()` 와 같은 질문에 답을 내려야 하는데 만약 `Carpenter` 클래스가 `Table` 과 `Chair` 클래스를 import 하여 `carpenter.whatdo()` 와 같은 질문에 대답을 해야한다면, circular dependency를 가지고 있는 것이다.
  이러한 경우 method나 function 안에서 import statement를 작성하는 것과 같은 취약한 방법에 의존해야 한다.
* 숨겨진 coupling:
  각각 그리고 모든 `Table` 구현의 변화는 `Table` 과 관련없는 20개의 테스트를 통과하지 못하고 이는 `Carpenter` 의 코드에 문제를 발생시켜 수정사항을 적용할 때 각별한 주의가 필요하다.
  이는 `Carpenter` 코드에서 `Table` 에 대해 너무 많은 가정을 했다는 의미거나 또는 그 반대인 경우이다.
* global state나 context의 과도한 사용:
  명확하게 `(height, width, type, wood)` 와 같이 넘기는 것이 아니라 `Table` 과 `Carpenter` 가 다른 agent에 의해 필요에 따라 수정이 될 수 있는 global variable에 의존하는 경우이다.
  당신은 왜 직사각형의 table이 정사각형이 되었는지를 알아내기 위해 global variable에 접근하는 모든 코드를 자세하게 읽어 보아야 하고, remote template code 또한 이 table 치수에 관한 것을 혼동하여 수정중임을 알아차리게 된다.
* 스파게티 코드:
  복사 - 붙여넣기로 만들어진 clause나 for loop이 많고 적절한 구분이 없이 만들어진 nested 된 페이지들을 스파게티 코드라고 한다.
  Python의 의미있는 indentation(논란의 여지가 있는 feature 중 하나)이 이러한 코드를 유지보수 하기에 어렵게 만든다.
  따라서 이런 일은 생각보다 많이 보이지는 않을 것이다.
* Python에서는 Ravioli code가 더 많다:
  이는 적절한 구조가 아니라 비슷한 기능을 하는 많은 작은 logic, class, object으로 구성되어 있다.
  당신이 `FurnitureTable` 을 쓸지, `AssetTable` 을 쓸지, `Table` 을 쓸지 아니면 `TableNew` 를 만들어야 할지 알지 못하게 된다면 `ravioli code` 의 늪에 빠졌을 지도 모른다.

## Modules

Python module은 
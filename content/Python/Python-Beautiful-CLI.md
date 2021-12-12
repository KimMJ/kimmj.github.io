---
title: "[번역]Python을 통해 이쁜 CLI 만들기"
date:  2020-02-22T23:08:39+09:00
weight: 10
draft: false
tags: ["python"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

링크 : <https://codeburst.io/building-beautiful-command-line-interfaces-with-python-26c7e1bb54df>

command line application을 만드는 것을 다루기 전에 빠르게 Command Line에 대해서 알아보자.

command line 프로그램은 컴퓨터 프로그램이 생성되었을 때부터 우리와 함께 해왔고, 명령어들로 구성되어있다.
commnad line 프로그램은 command line에서 또는 shell에서 동작하는 프로그램이다.

command line interface는 user interface이지만 마우스를 사용하는 것이 아닌 terminal, shell, console에서 명령어를 입력하여 사용하는 것이다.
console은 이미지나 GUI가 하나도 없이 전체 모니터 스크린이 텍스트로만 이루어진 것을 의미한다.

위키피디아에 의하면

> CLI는 주로 1960년대 중만에 컴퓨터 terminal에서의 대부분의 컴퓨터 시스템과의 상호작용을 의미하고 1970년대와 1980년대를 거쳐 OpenVMS, MS-DOS를 포함한 개인용 컴퓨터와 Unix system, CP/M과 Apple DOS에서 사용되어 왔다.
> 인터페이스는 보통 명령어를 텍스트로 받고 이 명령어를 통해 적절한 system function을 동작시키게 하는 command line shell에서 동작한다.

## 왜 Python인가?

Python은 유연성과 현존하는 프로그램들과도 잘 작동하기 때문에 보통 `glue code language`라고 여겨진다.
대부분의 Python 코드는 script와 command-line interface(CLI)로 작성된다.

이런 command-line interface와 tool은 거의 대부분 원하는 것들을 자동화할 수 있기 때문에 특히 강력하다.

우리는 예쁘고 상호작용을 하는 인터페이스, UI, UX가 매우 중요한 시대를 살고 있다.
우리는 이런 것들을 Command Line에 추가하고 사람들이 이를 받아들이고 Heroku같은 유명한 회사는 이를 공식적으로 사용할 것이다.

arguments와 option을 파싱하는 것에서부터 output에 색깔을 주고, progress bar를 추가하고 email을 전송하는 것과 같은 엄청난 CLI "frameworks"를 flagging하기까지 command line app을 build하는데 도움을 주는 많은 Python library와 module이 있다.

이런 module과 함께 우리는 `Heroku`나 `Vue-init`이나 `NPM-init`같은 Node programe처럼 예쁘고 상호작용을 하는 command line interface를 만들 수 있다.

예쁜 `vue init` CLI를 쉽게 만드려면 `Inquirer.js`를 Python에 이식하는 `Python-inquirer`를 사용한는 것을 권장한다.

불행히도 `Python-inquirer`는 `blessings`(Unix같은 시스템에서만 사용가능한 `_curses`와 `fcntl` module을 import하는 python package)를 사용하기 때문에 Windows에서 작동하지 않는다.
어떤 대단한 개발자들은 `_curses`를 Windows에 이식할 수도 있긴 할것이다.
Windows에서 `fcntl`의 대체제는 `win32api`이다.

하지만 구글링을 통해 나는 이를 고치게 되었고 이를 `PyInquirer`라고 불르게 되었다.
이는 `python-inquirer`의 대체제이고 더 좋은 점은 이것은 Windows를 포함한 모든 플랫폼에서도 사용이 가능하다는 것이다.

## Basics in Commnad Line Interface with Python

이제 간단하게 command line interface를 보고 Python으로 하나를 만들어보자.

command-line interface(CLI)는 실행파일의 이름으로 보통 시작한다.
그냥 console에서 이름을 입력하면 `pip`처럼 스크립트의 main entry point에 접근하게 된다.

script가 어떻게 개발되었는지에 따라 우리가 전달해 주어야 할 parameter들이 있고 이들은 이런 종류가 있다.

1. **Arguments**: 스크립트에 전달되어야 하는 *required* parameter이다.
   이를 작성하지 않으면 CLI는 error를 발생시킬 것이다.
   예를 들어 `django`는 여기서 arguments이다.
   ```bash
   pip install django
   ```
2. **Options**: 이름에서 알 수 있듯이 *optional* parameter이다.
   보통은 `pip install django --cache-dir ./my-cache-dir`처럼 name과 value의 쌍으로 사용한다.
   `--cache-dir`은 option parameter이고 value `./my-cache-dir`은 cache directory로 사용되는 것이다.
3. **Flags**: script에게 특정 행동을 disable할지 enable할지 알려주는 특수한 option parameter이다.
   대부분은 `--help`같은 것들이다.

`Heroku Toolbelt`같은 복잡한 CLI를 통해 우리는 main entry point 아래의 몇몇 command들에 접근할 수 있게 된다.
이를 보통 **commands**와 **sub-commands**라고 한다.

다른 python package들로 똑똑하고 예쁜 CLI를 어떻게 빌드하는지 보자.

## Argparse

`argparse`는 command line program을 생성하는 default python module이다.
간단한 CLI를 만드는 데에 필요한 모든 기능을 제공한다.

```python
import argparse

parser = argparse.ArgumentParser(description='Add som integers.')
parser.add_argument('integers', metavar='N', type=int, nargs='+', helm 'integer list')

parser.add_argument('--sum', action='store_const',
                    const=sum, default=max,
                    help='sum the integers (default: find the max)')

args = parser.parse_args()

print(args.sum(args.integers))
```

`argparse`는 간단한 추가적인 동작을 한다.
`argparse.ArgumentParser`는 프로그램에 description을 추가할 수 있도록 하는 반면 `parser.add_argument`는 command를 추가할 수 있도록 한다.
`parser.parse_args()`는 주어진 arguments를 리턴하고 보통 이것들은 name-value 쌍으로 제공된다.

예를 들어 `args.integers`를 통해 `integers` arguments에 접근할 수 있다.
위의 script에서 `--sum`은 optional argument이지만 `N`은 positional argument이다.

## Click

`click`으로 우리는 CLI를 `argparse`보다는 쉽게 만들 수 있다.
`click`은 argparse가 해결하는 문제와 동일한 것을 해결해준다.
하지만 약간은 다른 접근방식을 사용한다.
이는 `decorators` concept을 사용한다.
이는 command가 function이 되도록 하며 이를 decorators로 감쌀 수 있게 한다.

```python
# cli.py
import click

@click.command()
def main():
    click.echo("This is a CLI built with Click")

if __name__ == "__main__":
    main()
```

아래와 같이 argument와 option을 추가할 수 있다.

```python
# cli.py
import click

@click.command()
@click.argument('name)
@click.option('--greeting', '-g')
def main(name, greeting):
    click.echo("(), ()".format(greeting, name))

if __main__ == "__main__":
    main()
```

위의 script를 실행해보면 다음과 같을 것이다.

```bash
$ python cli.py --greeting <greeting> Oyetoke
Hey. OyeTOKI 
```

모든걸 모아 Google Books의 query books에 간단한 CLI를 만들었다.

<script src="https://gist.github.com/CITGuru/e66c25fafe4e04973c6c8eaae6d888c1.js"></script>

`click`에 대한 자세한 정보는 [official documentation](http://click.pocoo.org/)에서 확인할 수 있다.

## Docopt

`docopt`는 POSIC-style이나 Markdown 사용법을 파싱하여 쉽게 command line interface를 생성하는 lightweight python package이다.
`docopt`는 수년간 사용해온 command line interface를 설명하는 정형화된 helm message와 man page에 대한 convention을 사용한다.
`docopt`에서의 interface description은 helm message와 비슷하지만 정형화되어있다.

`docopt`는 파일의 젤 위에 required docstring이 어떻게 형식을 갖추는지가 중요하다.
tool의 이름 다음에 올 docstring에서의 맨 위의 element는 `Usage`이여야 하고 이는 command가 어떻게 호출되고자 하는지에 대한 리스트들을 나열해야 한다.

docstring에서 두번째 element는 `Options`여야 하고 이는 `Usage`에서 나타난 option과 arguments에 대한 정보를 제공해야 한다.
docstring의 내용들은 help text의 내용이 될 것이다.

<script src="https://gist.github.com/CITGuru/bb619be5155b48688e089e479cbd0939.js"></script>

## PyInquirer

`PyInquirer`는 interactive command line user interface이다.
우리가 위에서 보았던 package들은 우리가 원하는 "예쁜 interface"를 제공하지 않는다.
따라서 어떻게 `PyInquirer`를 사용하는지 알아보도록 하자.

`Inquirer.js`처럼 `PyInquirer`는 두가지 간단한 단계로 구성되어 있다.

1. 질문 리스트를 정의하고 이를 prompt에 전달한다.
2. prompt는 답변 리스트를 리턴한다.

```python
from __future__ import print_function, unicode_literals
from PyInquirer import prompt
from pprint import pprint

questions = [
    {
        'type': 'input',
        'name': 'first_name',
        'message': 'What\'s your first name',
    }
]

answers = prompt(questions)
pprint (answers)
```

상호작용하는 예시이다.

<script src="https://gist.github.com/CITGuru/17dc59353a0b985c918ef841aafa520c.js"></script>

결과는 다음과 같다.

![PyInquirer.py](/images/Python/PyInquirer.png)

이 스크립트의 몇몇 부분을 보도록 하자.

```python
style = style_from_dict({
Token.Separator: '#cc5454',
Token.QuestionMark: '#673ab7 bold',
Token.Selected: '#cc5454',  # default
Token.Pointer: '#673ab7 bold',
Token.Instruction: '',  # default
Token.Answer: '#f44336 bold',
Token.Question: '',
})
```

`style_from_dict`는 우리의 interface에 적용하고자 하는 custom style을 정의할 때 사용한다.
`Token`은 component와 같은 것이고 이 아래에는 다른 component들도 있다.

이전 예시에서 `questions` list를 `prompt`로 전달하여 처리하는 것을 보았다.

이 방식으로 우리가 생성할 수 있는 interactive CLI에는 이런 예시가 있다.

<script src="https://gist.github.com/CITGuru/01dd41b3367d8bd079e2e538e0c8a396.js"></script>

결과

![PyInquirer2.py](/images/Python/PyInquirer2.png)

## PyFiglet

`pyfiglet`은 ASCII text를 arts fonts로 변환해주는 python module이다.
`pyfiglet`은 모든 FIGlet(<http://www.figlet.org/>)들을 pure python에 이식한 것이다.

```python
from pyfiglet import Figlet
f = Figlet(font='slant')
print f.rederText('text to render')
```

결과

![pyfiglet.py](/images/Python/pyfiglet.png)

## Clint

`clint`는 CLI를 만드는데 필요한 모든 것들을 통합한 것이다.
color도 지원하고 뛰어난 nest-able indentation context manager도 지원하며 custom email-style quotes, auto-expanding column이 되는 column printer도 지원한다.

<script src="https://gist.github.com/CITGuru/42ddad7454d812d4325c508688d21580.js"></script>

![clint.py](/images/Python/clint.png)

## EmailCLI

모든걸 통합하여 나는 SendGrid를 통해 메일을 전송하는 간단한 cli를 작성하였다.
아래의 스크립트를 사용하려면 [SendGrid](https://sendgrid.com/)에서 API Key를 받아야 한다.

### Installation

```bash
pip install sendgrid click PyInquirer pyfiglet pyconfigstore colorama termcolor six
```

<script src="https://gist.github.com/CITGuru/a81b0cdb6ab32c73f578953805901468.js"></script>

![EmailCLI.py](/images/Python/EmailCLI.png)

* 읽으면 좋은 것: <https://www.davidfischer.name/2017/01/python-command-line-apps/>

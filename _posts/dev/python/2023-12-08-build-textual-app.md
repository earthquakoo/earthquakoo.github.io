---
layout: post
title:  "Python Textual 라이브러리로 TUI App 개발:Project 생성부터 실습까지"
subtitle:   "Python Textual 라이브러리로 TUI App 개발"
categories: dev
tags: python
comments: true
---

1. Introduction
2. Project 생성
3. 간단한 TUI app 만들기

---

# 1. Introduction

코딩을 하면서 그래픽 환경인 CLI, GUI, TUI에 대해 들어보신 적이 있을 것입니다.

- CLI (Command Line Interface): 명령어를 입력하여 상호작용하는 환경으로, 윈도우에는 cmd, 리눅스에는 terminal이 대표적입니다.
- GUI (Graphic User Interface): 그래픽으로 사용자가 상호작용하는 환경으로, UI(User Interface)에 시각적인 개념이 추가된 용어입니다.
- TUI (Text User Interface): 텍스트를 통해 사용자가 상호작용하는 환경으로, 주로 터미널에서 사용되며 리눅스의 vi(vim) 편집기가 대표적입니다. (위키피디아에서는 Text User Interface를 컴퓨터 터미널의 속성에 대한 의존성을 반영하기 위해 Terminal User Interface라고도 언급하고 있습니다.)

하지만 TUI는 종종 텍스트가 주를 이루어 한 눈에 들어오지 않아 쉽게 와닿지 않을 수 있습니다. 따라서, 이번에는 [Textual](https://textual.textualize.io/)을 활용하여 시각적으로 화려한 TUI 앱을 구축해보려 합니다.

# 2. Project 생성

os: window, terminal: git bash

```bash
python -m venv venv
source venv/scripts/activate
```

아래의 커맨드로 개발용 textual 패키지를 설치해줍니다.

```bash
pip install "textual-dev"
```

만약 textual 패키지 자체를 다운로드하거나 예제를 테스트하고 싶다면 다음 명령어로 textual 패키지를 설치할 수 있습니다. (TUI 앱을 개발할 목적이라면 앞서 언급한 명령어를 사용하세요)

```bash
pip install "textual"
```


# 3. 간단한 TUI app 만들기

이제 `Textual`을 이용해 문구를 입력하면 화면에 출력되는 간단한 TUI app을 만들어보겠습니다.

```python
from textual import on
from textual.app import App, ComposeResult
from textual.widgets import Input, RichLog
from textual.reactive import reactive


class ChatApp(App):

    def compose(self) -> ComposeResult:
        yield RichLog()
        yield Input()
    
    @on(Input.Submitted)
    def on_input_submitted(self, event: Input.Submitted):
        input = self.query_one(Input)
        log = self.query_one(RichLog)
        log.write(f"earthquake: {event.value}")
		input.value = ""


if __name__  == "__main__":
    app=ChatApp()
    app.run()
```

`compose` 메소드는 반복 가능한 인스턴스나 위젯, 혹은 위젯의 목록을 반환합니다. 여기서는 위젯을 생성하여 메서드를 generator로 만드는 것이 편리합니다.

입력을 받기 위해 `Input` 위젯과 입력받은 결과를 화면에 출력하기 위한 `RichLog` 위젯을 생성했습니다.

```python
...
def compose(self) -> ComposeResult:
	yield RichLog()
	yield Input()
```

- `on_input_submitted는` 이벤트 핸들러로, 이벤트 핸들러는 키 누르기, 마우스 클릭 등과 같은 이벤트에 대한 응답으로 textual에서 호출하는 메소드입니다. 여기서는 Input에서 리턴값이 생겼을 때의 이벤트를 의미합니다.
	- `on_input_submitted처럼` 함수명을 제시해서 이벤트 핸들러로 사용할 수도 있지만 `@on` 이라는 데코레이터를 이용해서도 이벤트 핸들러처럼 사용할 수 있습니다.

```python
...
@on(Input.Submitted)
def on_input_submitted(self, event: Input.Submitted):
...
```

- `query_one` 메소드는 generator 또는 유형과 일치하는 단일 위젯을 가져옵니다.
- 3번째줄에선 `RichLog` 위젯을 가져와 `RichLog`의 입력 메소드인 `write`로 `Input`에서 전달받은 값을 화면에 출력합니다.
- 이후 `Input`의 제출이 완료되면 `Input` 칸의 value를 초기화시켜줍니다.

```python
...
input = self.query_one(Input)
log = self.query_one(RichLog)
log.write(f"earthquake: {event.value}")
input.value = ""
...
```

실행 결과로 "hello world"를 입력하면 아래와 같은 결과를 얻을 수 있습니다.

![img](/assets/img/dev/python/textual-1.png)

더 자세한 정보는 [Textual](https://textual.textualize.io/)공식문서에서 확인이 가능합니다. 

다음 포스팅에선 해당 프로젝트를 PyPI에 배포해보겠습니다.
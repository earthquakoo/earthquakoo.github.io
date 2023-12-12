---
layout: post
title:  "Python Textual 라이브러리로 TUI App 개발:PyPI에 배포"
subtitle:   "Python Textual 라이브러리로 TUI App 개발"
categories: dev
tags: python
comments: true
---

1. Introduction
2. 환경설정
3. Textual app 생성
4. 패키지 관리
5. 패키지 테스트 및 배포
6. 패키지 업데이트

---

# 1. Introduction

이전 포스팅에서 `textual`을 이용해 간단한 TUI app을 만들어보았습니다. 이번 포스팅에서는 이 라이브러리를 사용자들이 쉽게 활용할 수 있도록 배포하는 방법에 대해 알아보겠습니다.

# 2. 환경설정

먼저 `poetry를` 사용하여 `PyPI`에 배포하기 위한 환경을 설정합니다.

```bash
pip install pipx

pipx install poetry
```

이후 바탕화면에서 새로운 poetry 프로젝트를 생성합니다.

```bash
poetry new textual_app
```

생성한 poetry 프로젝트로 이동하여 코드를 작성할 편집기로 폴더를 열어줍니다! (예: vscode)

```
cd textual-app
code .
```

프로젝트의 구조는 다음과 같습니다.

```
> tests
> textual_app
pyproject.toml
README.md
```

# 3. Textual app 생성

간단한 설정이 완료되었으므로 textual app을 만들어보겠습니다. 먼저 필요한 라이브러리를 설치합니다.

```bash
poetry add "textual[dev]"
```

설치된 패키지는 `pyproject.toml`의 dependencies에 추가됩니다. `pyproject.toml`은 Python 프로젝트의 빌드 시스템 요구 사항을 담은 파일입니다.

```toml
[tool.poetry]
name = "textual-app"
version = "0.1.0"
description = ""
authors = ["Earthquakoo <cream5343@gmail.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.9"
textual = {extras = ["dev"], version = "^0.44.1"}


[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

```

필요한 라이브러리가 설치되었다면 poetry 가상환경을 실행합니다.

```bash
poetry shell
```

이제 간단한 textual app을 만들어보겠습니다.

`textual_app` 폴더에서 `main.py` 파일을 생성하고 아래의 코드를 붙여넣기 해줍니다.

```python
from textual import on
from textual.app import App, ComposeResult
from textual.widgets import Input, RichLog


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

위와 같은 예시 프로젝트를 실행하기 위해서 터미널에 아래와 같이 입력해줍니다.

```bash
python textual_app/main.py
```

실행결과는 아래의 사진과 같습니다.

![img](/assets/img/dev/python/textual-1.png)


# 4. 패키지 관리

이제 `pyproject.toml`에서 몇 가지를 수정해야 합니다.

- `version`은 0.1.0으로 초기화 되어있습니다.
- `description`은 app에 대한 간단한 설명을 추가할 수 있습니다.
- `[tool.poetry.scripts]`로 패키지를 실행하는 명령을 정의할 수 있습니다.

여기서 중요한 것은 `[tool.poetry.scripts]` 입니다. 이전에 프로젝트를 실행하기 위해 터미널에서 `python textual_app/main.py` 로 실행했습니다. 그러나 이것은 조금 번거로우며 사용자가 길게 늘어진 패키지 타이핑을 실행하는 것은 좋지 못한 예입니다.

따라서 해당 패키지를 설치했을 때 간단한 타이핑으로 이 app을 실행할 수 있도록 `scripts를` 추가합니다.

```toml
[tool.poetry]
name = "textual-app"
version = "0.1.0"
description = "Simple textual app"
authors = ["Earthquakoo <cream5343@gmail.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.9"
textual = {extras = ["dev"], version = "^0.44.1"}

[tool.poetry.scripts]
textual-app = "textual_app.main:main"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

`scripts를` 설정했다면 `main.py`에서 해당 app의 경로를 정의한 것을 삭제하고 패키지를 실행하는 함수를 생성합니다.

```python
from textual import on
from textual.app import App, ComposeResult
from textual.widgets import Input, RichLog


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


def main():
    app = PreventApp()
    app.run()


# if __name__ == "__main__":
#     app = PreventApp()
#     app.run()
```

# 5. 패키지 테스트 및 배포

테스트를 위해 로컬에서 패키지를 설치하고 실행해봅니다.

```bash
poetry install
```

패키지가 제대로 설치되었다면 아래의 `textual-app` 명령어로 간단히 패키지를 실행할 수 있습니다!

```bash
textual-app
```

이제 다른 사람들이 해당 패키지를 다운로드할 수 있도록 PyPI에 배포를 해보겠습니다.

```bash
poetry build
```

`poetry build` 명령어를 실행하면 프로젝트 구조는 아래와 같아집니다.

```
> dist
> tests
> textual_app
pyproject.toml
README.md
```

이제 [PyPI](https://pypi.org/)사이트에서 계정을 생성하고 아래와 같이 [API 토큰](https://pypi.org/manage/account/token/)을 등록해주고 발급 받은 토큰은 따로 저장해둡니다.

![img](/assets/img/dev/python/pypi-1.png)

아래의 커맨드와 함께 저장한 토큰을 입력합니다.

```bash
poetry config pypi-token.pypi <Your api token>
```

이제 패키지를 배포합니다.

```bash
poetry publish --build
```

[PyPI](https://pypi.org/)에서 자신의 패키지명을 검색하면 배포가 된 것을 확인할 수 있습니다.

![img](/assets/img/dev/python/pypi-2.png)

터미널에서 배포한 패키지를 설치해서 테스트합니다.

```bash
pip install textual-app
```


# 6. 패키지 업데이트

만약 수정할 것이 있거나 새로운 기능이 생겨 업데이트가 필요하다면 `pyproject.toml`에서 `version`과 `description`을 수정합니다.

```toml
[tool.poetry]
name = "textual-app"
version = "0.1.1"
description = "Update textual app"
authors = ["Earthquakoo <cream5343@gmail.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.9"
textual = {extras = ["dev"], version = "^0.44.1"}

[tool.poetry.scripts]
textual-app = "textual_app.main:main"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

```

또한 `textual_app/__init__.py`의 `__version__`도 함께 수정합니다.

```python
__version__ = "0.1.1"
```

다시 아래의 커맨드로 업데이트된 패키지를 배포합니다.

```bash
poetry publish --build
```

이후 패키지를 업데이트하려면 다음을 실행합니다.

```bash
pip install textual-app --upgrade
```
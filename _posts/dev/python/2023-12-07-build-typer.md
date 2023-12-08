---
layout: post
title:  "Typer로 CLI App 구축하기"
subtitle:   "Typer로 CLI App 구축하기"
categories: dev
tags: fastapi python
comments: true
---


1. Introduction
2. Project 생성
3. 간단한 CLI app 만들기
4. CLI argument
5. CLI option
6. Commands
7. To do list 구현
---

# 1. Introduction

[Typer](https://typer.tiangolo.com/)는 python 3.6+ 유형 힌트를 기반으로 한 python용 CLI app 구축 라이브러리 입니다.

`Sigan` 프로젝트를 진행할 때 사용했던 라이브러리로써 공식문서도 잘 나와있었기에 CLI app을 개발하는데 있어 도움이 되었습니다.

이 포스팅에서는 공식문서를 기반으로 typer에 대해 살펴보고 최종적으로 to do list를 만들어보도록 하겠습니다!

# 2. Project 생성

os: window, terminal: git bash

```bash
python -m venv venv
source venv/scripts/activate
```

아래의 커맨드로 개발용 textual 패키지를 설치해줍니다.

```bash
pip install "typer[all]"
```

# 3. 간단한 CLI app 만들기

아래의 코드를 실행하면 `Hello World!` 라는 문구가 출력되는 아주 간단한 CLI app입니다.

```python
import typer

def main():
    print("Hello World!")

if __name__ == "__main__":
    typer.run(main)
```

```bash
python main.py

> Hello World!
```

아래의 커맨드는 해당 CLI App의 도움말을 확인할 수 있습니다.

```bash
python main.py --help
```

# 4. CLI argument

위의 예시는 CLI App이라고 보기에는 너무 간단합니다. 이제 CLI argument(인수)라는 것을 통해 CLI App을 좀 더 발전시켜보겠습니다.

CLI argument란 단어를 사용하여 특정 순서로 CLI App에 전달되는 CLI parameters(매개변수)를 나타냅니다. 이는 기본적으로 "required(필수적)"입니다.

아래는 CLI argument인 `name`을 추가한 예시입니다.

```python
import typer


def main(name: str):
    print(f"Hello {name}")


if __name__ == "__main__":
    typer.run(main)
```

전과 같이 `python main.py` 로 동작시키면 `Missing argument 'NAME'.` 이라는 에러를 발생시킵니다. 이를 `python main.py --help`로 확인해보면 argument인 `name`은 required(필수적)이라는 것을 확인할 수 있습니다.

```bash
python main.py --help
```

![img](/assets/img/dev/python/build-typer-1.png)

이는 argument인 `name`을 입력하지 않았다는 오류로 필수조건인 argument를 입력해주면 다시 정삭적으로 동작하게 됩니다.

```bash
python main.py earthquake

> Hello earthquake
```

만약 띄어쓰기를 적용하고 싶다면 `""` 안에 `name`을 입력하면 됩니다.

```bash
python main.py "earthquake woo"

> Hello earthquake woo
```

# 5. CLI option

CLI argument는 "required(필수적)"한 요소를 확인했다면 이제  `--help` 와같은 "optional(선택적)"한 CLI option에 대해 살펴보겠습니다.

CLI option은 말그래도 선택적인 요소로 적용할 수 있습니다. 아래의 예시는 `--formal` 이라는 CLI option 여부에 따라 출력결과가 달라지는 CLI app 입니다.

```python
import typer


def main(name: str, formal: bool = False):
    if formal:
        print(f"Good day Ms. {name}.")
    else:
        print(f"Hello {name}")


if __name__ == "__main__":
    typer.run(main)
```

```bash
python main.py earthquake
> Hello earthquake

python main.py earthquake --formal
> Good day Ms. earthquake
```

CLI argument와 CLI option는 간단히 정의할 수 있지만 좀 더 명시적인 것을 원한다면 아래와 같이 구현할 수 있습니다.

```python
import typer

def main(
    name: str = typer.Argument(...),
    formal: bool = typer.Option(False, "-f", "--formal")
):
    if formal:
        print(f"Good day Ms. {name}.")
    else:
        print(f"Hello {name}")

if __name__ == "__main__":
    typer.run(main)
```

# 6. Commands

대부분 CLI app에는 여러 command가 존재합니다. 예를 들어 `git`에는 `git add`, `git push` 등 차례대로 자체의 CLI argument와 option이 있는 것처럼!

Typer에서도 이와 같이 command를 구현할 수 있습니다. `@app.command()`에 아무 인자를 전달하지 않으면 함수의 이름이 command가 되고 `@app.command("delete")`와 같이 명령어를 전달하면 `delete`가 command가 됩니다.

```python
import typer

app = typer.Typer()


@app.command()
def create():
    print("Creating user: earthquake")


@app.command("delete")
def delete_func():
    print("Deleting user: earthquake")


if __name__ == "__main__":
    app()
```

출력 결과:

```bash
python main.py create
> Creating user: earthquake

python main.py delete
> Deleting user: earthquake
```


# 7. To do list 구현

보통의 to do list라 하면 task 생성, 삭제, 리스트 확인 등이 기반이 될 것입니다.

이를 기반으로 아래 3가지의 command와 함께 CLI app을 구현해보도록 하겠습니다.

- `create` : task create
- `delete` : task delete
- `show` : task show

Task를 저장하기위해 루트 디렉터리에 아래와 같은 `config.json`를 만듭니다. 

```json
{
	"tasks": []
}
```

아래는 to do list의 기본 동작을 위한 함수들입니다.
`utils.py`

```python
import json
from rich.table import Table
from rich.console import Console

console = Console()

def read_json(fpath):
    with open(fpath, 'r') as f:
        data = json.load(f)
        return data


def write_json(fpath, data):
    with open(fpath, 'w') as f:
        json.dump(data, f)


def add_task(fpath: str, new_task: str):
    data = read_json(fpath)
    data['tasks'].append(new_task)
    write_json(fpath, data)


def remove_task(fpath: str, task: str):
    data = read_json(fpath)
    if task not in data['tasks']:
        raise Exception(f"There's no task named {task}!")
    idx = data['tasks'].index(task)
    print(idx)
    data['tasks'].pop(idx)
    write_json(fpath, data)


def show_tasks(fpath: str):
    table = Table("ID", "Name")
    tasks = read_json(fpath)['tasks']
    for i in range(len(tasks)):
        table.add_row(str(i + 1), tasks[i])
    console.print(table)
```

`rich` 라는 모듈을 통해 터미널에서 좀 더 예쁘게 출력되도록 설정했습니다.

```python
import typer
import utils
from rich.console import Console
from rich.table import Table

app = typer.Typer(rich_markup_mode='rich')
console = Console()
cfg_path = "config.json"


@app.command("create")
def create(
    task: str = typer.Argument(..., help="Task to be created")
):
    utils.add_task(cfg_path, task)
    console.print("[yellow]Task created successfully![/yellow]")
    utils.show_tasks(cfg_path)


@app.command("delete")
def delete(
    task: str = typer.Argument(..., help="Task to be deleted")
):
    utils.remove_task(cfg_path, task)
    console.print("[yellow]Task removed successfully![/yellow]")
    utils.show_tasks(cfg_path)


@app.command("show")
def show():
    utils.show_tasks(cfg_path)

if __name__ == "__main__":
    app()
```

만약 위의 과정을 다 완료했다면 프로젝트의 구조는 아래와 같습니다.

```
> venv
config.json
todo.py
utils.py
```

- `python todo.py create <task name>`

![img](/assets/img/dev/python/build-typer-2.png)
![img](/assets/img/dev/python/build-typer-3.png)
- `python todo.py delete <task name>`

![img](/assets/img/dev/python/build-typer-4.png)

- `python todo.py show`

![img](/assets/img/dev/python/build-typer-5.png)

목표했던 to do list CLI app을 완성했습니다!

### reference

- https://typer.tiangolo.com/
- https://noisrucer.github.io/posts/typer-intro/

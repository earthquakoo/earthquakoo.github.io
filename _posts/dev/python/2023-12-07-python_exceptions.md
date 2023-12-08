---
layout: post
title:  "Python exceptions with FastAPI handling errors"
subtitle:   "Python exceptions with FastAPI handling errors"
categories: dev
tags: fastapi python
comments: true
---

- Introduction
- Handling Exceptions
- Raising Exceptions
- FastAPI Handling Errors & Custom exception handlers
- User defined Exceptions with FastAPI
  
---

# Introduction

Pyhton에서 코드를 구현하다보면 여러 에러를 맞이하게 됩니다. 이렇듯 코드를 실행하는 중에 발생한 에러는 exception(예외)라 합니다.

예외처리를 하기위해 python에서는 `try` `except` 구문이 존재합니다. `try`에 실행할 코드를 구현하고 `except`에 예외가 발생했을 때 처리하는 코드를 구현합니다.

이 포스팅에선 `try` `except`에 대해 살펴보고 사용자 정의 예외를 구현하고 이를 FastAPI에서 적용시켜보도록 하겠습니다.
# Handling Exceptions

에러는 문장이나 표현식이 올바르다 할지라도 실행할 때 에어를 일으킬 수 있습니다. 실행 중에 감지되는 에러들을 예외(exception)라 부르고 이는 무조건 치명적이지는 않습니다. 아래는 올바른 표현식이나 프로그램이 처리하지 않은 에러의 예시입니다.

```python
print(10 * (1/0))
```

```bash
Traceback (most recent call last):
  File "C:\Users\cream\desktop\reference\main.py", line 1, in <module>
    print(10 * (1/0))
                ~^~
ZeroDivisionError: division by zero
```

에러의 마지막줄의 `ZeroDivisionError` 는 내장 예외 중 하나입니다. 에러의 나머지는 예외의 형태와 원인에 기반을 둔 상세 내용을 제공합니다.

이러한 예외들을 `try` `except` 구문을 통해 선택적으로 처리할 수 있습니다. 아래는 올바른 정수가 입력될 때까지 예외를 일으키는 예시입니다.

```python
while True:
    try:
        x = int(input("Please enter a number: "))
        break
    except ValueError:
        print("Oops!  That was no valid number.  Try again...")
```

- 먼저 `try` 바로 아래의 코드가 실행됩니다.
- 예외가 발생하지 않으면 `except`를 건너뛰고 `try` 문의 실행은 종료됩니다.
- 예외가 발생하면 `try` 구문의 나머지를 건너뛰고 `except` 에서 해당 유형의 에러가 발생하면 예외처리가 됩니다.
- 만약 `except`에서 제시된 유형의 에러가 발생하지않으면 에러 메시지와 함께 실행이 종료됩니다.

`BaseException`의 하위 클래스 중 하나인 `Exception`은 치명적이지 않은 모든 예외의 기본 클래스입니다. ([예외 계층](https://docs.python.org/3/library/exceptions.html#exception-hierarchy) 은 여기서 확인 가능합니다.) 만약 위의 예시에서 무슨 에러가 발생하는지 잘 모르겠다면 `Exception`을 통해 거의 모든 예외를 처리할 수 있습니다. 

```python
while True:
    try:
        x = int(input("Please enter a number: "))
        break
    except Exception:
        print("Oops!  That was no valid number.  Try again...")
```

하지만 처리하려는 예외 유형을 최대한 구체적으로 지정하는 것이 좋습니다. 만약 더 정확한 에러를 알고 싶다면 `except` 구문에서 `as` 뒤에 변수를 지정해서 에러 메시지를 확인할 수 있습니다.

```python
while True:
    try:
        x = int(input("Please enter a number: "))
        break
    except Exception as e:
        print("Oops!  That was no valid number.  Try again...")
        print(e)
```

```bash
Please enter a number: s
Oops!  That was no valid number.  Try again...
invalid literal for int() with base 10: 's'
```
# Raising Exceptions

`raise` 구문은 사용자가 직접 지정한 예외가 발생하도록 강제할 수 있습니다. `raise`는 예외 인스턴스거나 예외 클래스(`BaseException` 하위 클래스) 중 하나 이어야 합니다.

아래는 `raise` 구문으로 직접 예외를 발생하도록 처리하고 `try` `except` 에서 처리한 예외를 다시 발생시키는 예시입니다.

```python
def test_exception():
    try:
        x = int(input("Please enter a number: "))
        if x % 2 != 0:
            raise Exception('Oops!  That was no valid number')
        print(x)
    except Exception as e:
        print('Exception in function', e)
        raise
 
try:
    test_exception()
except Exception as e:
    print('Exception in parent code', e)
```

```bash
Please enter a number: ;
Exception in function invalid literal for int() with base 10: ';'
Exception in parent code invalid literal for int() with base 10: ';'
```

# FastAPI Handling Errors & Custom exception handlers

FastAPI에선 일반적으로 `HTTPException`을 이용해 `status_code`와 에러 메시지인 `detail`로 예외처리를 합니다.

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```

클라이언트가 정상 요청을 할 경우 HTTP 상태코드인 200과 함께 다음 JSON 응답을 받게 됩니다.

```bash
{
  "item": "The Foo Wrestlers"
}
```

비정상 요청인 경우 HTTP 상태코드인 404와 함께 다음 JSON 응답을 받게 됩니다.

```bash
{
  "detail": "Item not found"
}
```

이제 간단한 사용자 정의 예외를 만들어 이 예외를 전역적으로 처리하는 코드를 작성해보겠습니다. `UnicornException` 이라는 사용자 정의 예외를 생성하고 `@app.exception_handler()`를 통해 사용자 정의 예외를 추가했습니다.

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse


class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name


app = FastAPI()


@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )


@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```

`/unicorns/yolo`로 요청하면 `raise UnicornException` 예외가 발생하고 이는 `unicorn_exception_handler` 에서 처리하고 다음과 같은 응답을 받게 됩니다.

```bash
{"message": "Oops! yolo did something. There goes a rainbow..."}
```

# User defined Exceptions with FastAPI

이것만으로도 충분한 사용자 예외 처리를 할 수 있어보입니다. 하지만 프로젝트를 진행하다보면 수많은 예외를 발생시켜야 할 때가 많습니다. 그렇기에 메인이 되는 `BaseException` 부모 클래스를 생성하고 이를 상속받아 FastAPI에서 `exception_handler`로 정의하면 재활용이 쉽고, 손쉽게 예외 처리를 할 수 있습니다.

저는 FastAPI에서 회원가입이 되지 않은 회원이 로그인을 시도하려할 때 발생시킬 사용자 정의 예외를 만드려고 합니다.

아래와 같이 `Exception` 클래스를 상속받아 사용자 정의 예외를 만들고 python 내장 클래스인 `__str__`을 정의해 객체가 print 함수에 전달될 경우 객체 내부의 `detail`을 반환하도록 했습니다.

```python
class BaseCustomException(Exception):
    """Base class for custom exceptions"""

    def __init__(self, status_code: int, detail: str):
        self.status_code = status_code
        self.detail = detail

    def __str__(self):
        return self.detail
```

직접 정의한 `BaseCustomException`을 상속받아 `UnregisteredUserError` 예외 클래스를 생성합니다. 여기선 `super().__init__` 을 통해 부모 클래스인 `BaseCutomException`의 인스턴스를 가져옵니다.

```python
...

class UnregisteredUserError(BaseCustomException):
    def __init__(self, user_name: str):
        super().__init__(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"{user_name} is an unregistered user."
        )
```

그리고 직접 정의한 `BaseCustomException`을 FastAPI에서 `exception_handler`로 사용하기 위한 handler를 생성합니다.

```python
...

def base_custom_exception_handler(request: Request, exc: BaseCustomException):
    return JSONResponse(status_code=exc.status_code, content={"detail": exc.detail})
```

이를 `add_exception_handler` 를 통해 예외 핸들러를 추가해줍니다.

```python
...
app = FastAPI()

...
app.add_exception_handler(BaseCustomException, base_custom_exception_handler)
```

이것이 실제로 적용된 코드가 궁금하다면 [Jobdamserver](https://github.com/earthquakoo/jobdamserver)에서 확인이 가능합니다!

### reference

- https://docs.python.org/ko/3/tutorial/errors.html
- https://fastapi.tiangolo.com/tutorial/handling-errors/
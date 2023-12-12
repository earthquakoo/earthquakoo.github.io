---
layout: post
title:  "HTTP 통신을 위한 Requests 라이브러리 with FastAPI"
subtitle:   "Python requests로 Rest API 호출하기"
categories: dev
tags: fastapi python
comments: true
---

- Introduction
- Requests 라이브러리
- Requests method with FastAPI
- 직접 적용한 requests with FastAPI

---

# Introduction

[Jobdam](https://github.com/earthquakoo/jobdamserver) 프로젝트에서 Python을 사용하여 HTTP 통신을 구현해야 했습니다. 

저는 Python 기반의 TUI(Terminal User Interface)인 [Textual](https://textual.textualize.io/)로 터미널 앱 [Jobdam](https://github.com/earthquakoo/jobdamserver)을 개발한 경험이 있습니다. 또한, 해당 프로젝트의 서버는 Python 기반의 FastAPI를 사용하여 구축되었기 때문에 Python을 이용한 HTTP 통신이 필수적이었습니다.

이를 위해 python으로 HTTP 통신을 가능하게 해주는 [requests](https://pypi.org/project/requests/)라이브러리를 사용하게 되었습니다. 이번 포스팅에서는 requests 라이브러리에 대한 개요와 Jobdam 프로젝트에서의 적용 방법에 대해 알아보겠습니다.

# 환경설정 및 Requests 라이브러리 개요

Requests 라이브러리는 HTTP 통신을 위한 다양한 메서드를 제공합니다. 이를 이용하여 REST API 방식의 Web API를 호출하고 데이터를 요청, 수정할 수 있습니다. 주요 메서드로는 `get`, `post`, `patch`, `put`, `delete` 등이 있습니다.

먼저 가상환경을 생성하고  패키지를 설치해줍니다.

```bash
python -m venv venv

source venv/scripts/activate

pip install requests
```


# Requests method with FastAPI

FastAPI는 빠르고 현대적인 Python 웹 프레임워크로, requests 라이브러리와 함께 사용하면 효율적인 웹 개발이 가능합니다.

이를 간단한 FastAPI 서버를 만들어서 테스트 해보겠습니다.

FastAPI 패키지를 설치해줍니다.

```bash
pip install "fastapi[all]"
```

FastAPI를 이용하여 간단한 서버를 만들어보겠습니다. `server.py` 파일을 생성하고 다음의 코드를 작성합니다.

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}
```

이제 클라이언트에서 해당 서버에 요청을 보내보겠습니다. `client.py` 파일을 생성하고 아래의 코드를 작성합니다.

```python
import requests

url = "http://localhost:8000"

resp = requests.get(url=url)

print(f"status_code: {resp.status_code}")
print(f"content: {resp.content}")
print(f"headers: {resp.headers}")
```

출력결과는 아래와 같습니다.

- `resp.status_code`는 말 그대로 HTTP 상태 코드를 반환합니다
- `resp.content`는 바이너리 데이터 값을 반환합니다.
- `resp.headers`는 response의 메타 데이터를 반환합니다.

```bash
status_code: 200
content: b'{"Hello":"World"}'
headers: {'date': 'Fri, 08 Dec 2023 13:09:06 GMT', 'server': 'uvicorn', 'content-length': '17', 'content-type': 'application/json'}
```

또한 `get` 방식으로 HTTP 요청을 할 경우 query string을 통해 응답받을 데이터를 필터링하는 경우가 많습니다.

이는 `get`의 `params` 인자로 넘겨주면 query string을 지정할 수 있습니다.

```python
import requests

url = f"http://localhost:8000/items/1"

resp = requests.get(url=url, params={"q": 1})

print(f"status_code: {resp.status_code}")
print(f"content: {resp.content}")
print(f"headers: {resp.headers}")
```

```bash
status_code: 200
content: b'{"item_id":1,"q":"1"}'
headers: {'date': 'Fri, 08 Dec 2023 13:16:03 GMT', 'server': 'uvicorn', 'content-length': '21', 'content-type': 'application/json'}
```

만약 `params`를 쓰지 않는다면 url에 그대로 모든 query string을 넣어줘도 무관합니다.

```python
...
url = f"http://localhost:8000/items/1?q=1"
...
```

이제 HTTP 요청에 데이터를 담아서 보내는 방식인 `post`에 대해서 살펴보겠습니다. Requests의 `post`는 `data`와 `json` 옵션으로 데이터를 담아서 보낼 수 있습니다. 

`data` 옵션은 HTML 양식(form) 포맷의 데이터를 전송할 수 있고 `content-type` 요청 헤더는 `application/x-www-form-urlencoded`로 설정됩니다.
`json` 옵션은 REST API 형식으로 JSON 포맷의 데이터를 전송할 수 있으며 `content-type` 요청 헤더는 `application/json`으로 설정됩니다.

`server.py` 를 아래와 같이 변경해줍니다. 2개 이상의 데이터를 받기 위해 `pydantic`의 `BaseModel`을 상속한 클래스를 생성해줍니다.

```python
from pydantic import BaseModel
from fastapi import FastAPI

app = FastAPI()


class UserData(BaseModel):
    user_name: str
    password: str


@app.post("/login")
def login(user_data: UserData):
    results = {"user_data": user_data}
    return results
```

이후 `client.py` 코드를 아래와 같이 변경 후 `data` 옵션을 이용해 요청해보겠습니다.

```python
import requests

url = f"http://localhost:8000/login"

data = {
    "user_name": "earthquake",
    "password": "pass123word123"
}

resp = requests.post(url=url, data=data)

print(f"status_code: {resp.status_code}")
print(f"content: {resp.content}")
print(f"headers: {resp.headers}")
```

`data` 옵션을 이용해 요청했더니 아래와 같은 에러를 발생시킵니다. 이유는 FastAPI는 별다른 양식 필드(form)을 지정하지 않으면 JSON으로 요청을 해야하기 때문입니다.

```bash
status_code: 422
content: b'{"detail":[{"type":"model_attributes_type","loc":["body"],"msg":"Input should be a valid dictionary or object to 
extract fields from","input":"user_name=earthquake&password=pass123word123","url":"https://errors.pydantic.dev/2.5/v/model_attributes_type"}]}'
headers: {'date': 'Fri, 08 Dec 2023 13:34:56 GMT', 'server': 'uvicorn', 'content-length': '255', 'content-type': 'application/json'}
```

옵션을 다시 `json`으로 변경한 뒤 실행하면 정상적인 응답이 오는 것을 확인할 수 있습니다. `json` 옵션을 이용하면 dictionary로 들어온 객체도 JSON의 형태로 변환 후 요청이 됩니다.

```python
...
resp = requests.post(url=url, json=data)
...
```

```bash
status_code: 200
content: b'{"user_data":{"user_name":"earthquake","password":"pass123word123"}}'
headers: {'date': 'Fri, 08 Dec 2023 13:35:05 GMT', 'server': 'uvicorn', 'content-length': '68', 'content-type': 'application/json'}
```

# 직접 적용한 requests with FastAPI

아래는 [Jobdam](https://github.com/earthquakoo/jobdam)프로젝트에서 requests 모듈을 이용해 HTTP 통신을 적용한 사례입니다. `create_chat_room` 이라는 함수에 dictionary 데이터를 인자로 받아 해당 endpoint에 데이터를 전송하고 지정된 리턴값을 받습니다.

```python
def create_chat_room(self, data: dict):
	resp = requests.post(
		url=cfg.base_url + "/chat_room/create",
		json=data,
		headers=auth_utils.build_jwt_header(cfg.config_path)
	)
	if resp.status_code == 201:
		create_room_resp = global_utils.bytes2dict(resp.content)
		return {"status_code": resp.status_code}
	elif resp.status_code == 400:
		detail = global_utils.bytes2dict(resp.content)['detail']
		return {"status_code": resp.status_code, "detail": detail}
	else:
		detail = global_utils.bytes2dict(resp.content)['detail']
		return {"status_code": resp.status_code, "detail": detail}
```

여기서 주목해야할 점은 5번째 줄에 `headers`입니다. 

제가 만든 app에서는 해당 요청을 보내는 사용자가 인증이 된 사용자인지를 확인해야합니다. 그러기 위해선 HTTP Authorization request header를 `post` 요청의 `headers`에 실어서 보내야합니다. HTTP Authorization request header의 양식은 다음과 같습니다. 

`Authorization: <type> <credentials>`

- `type`은 인증 타입 혹은 인증 스키마라고도 불리며 대표적인 예로는 `Bearer` 와 `jwt`가 있습니다.
- `credentials`는 base64 형태로 인코딩되는 것으로 흔히 불리는 `access token`이 이에 해당합니다.

`auth_utils.build_jwt_header(cfg.config_path)`의 함수를 보면 HTTP Authorization request header의 양식을 반환하는 것을 알 수 있습니다. 

```python
def build_jwt_header(fpath):
    return {
        "Authorization": "Bearer " + get_access_token_from_json(fpath)
    }
```

이를 통해 요청을 하는 사용자가 인증된 사용자라면 정상적인 응답을 받을 수 있고 만약 인증된 사용자가 아니라면 아래와 같이 인증되지 않았다라는 응답을 받게 됩니다.

```bash
"GET /chat_room/all_rooms_list HTTP/1.1" 401 Unauthorized
```

또한 8번째 줄을 보면 `resp.content` 를 `global_utils.byes2dict()` 라는 함수에 인자로 넘겨준 것을 확인할 수 있습니다.

`global_utils.byes2dict()` 함수는 아래와 같이 바이너리 형태로 넘어오는 `resp.content`를 dictionary 객체로 변환해주는 역할을 합니다.

```python
def bytes2dict(b):
    return json.loads(b.decode('utf-8'))
```

requests 모듈을 직접 적용한 프로젝트의 코드를 확인하고 싶다면 [Jobdam](https://github.com/earthquakoo/jobdam)에 방문해주세요!

### reference

- [Requests 라이브러리 공식 문서](https://docs.python-requests.org/en/latest/user/quickstart/)
- [파이썬에서 requests 라이브러리로 원격 API 호출하기](https://www.daleseo.com/python-requests/)
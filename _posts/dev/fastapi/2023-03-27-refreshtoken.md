---
layout: post
title:  "[FastAPI] Access token과 refresh token 구현"
subtitle:   "FastAPI base project"
categories: dev
tags: fastapi, python, access token, refresh token
comments: true
---

## 개요

처음 프로젝트에서 구현할 때는 refresh token을 데이터베이스에 저장하여 구현하는 방식을 이용했습니다. 하지만 공부를 하다보니 이 구현 방식은 모순적인 부분이 존재했습니다. 

JWT로 구현한 로그인 방식은 stateless를 전제하는데 이는 데이터를 서버에 저장하지 않고 token의 정보로만 상태를 관리하는 것입니다. 

한마디로 token에 정보를 담을 수 있는데 데이터베이스에 저장하려고 했던 것입니다. 이는 서버가 상태를 관리하게 되는 것이고 stateless를 전제로 하는 JWT 목적을 상실하게 됩니다.

이 문제점을 인식한 저는 JWT의 목적대로 새로이 로직을 구성하게 되었습니다. 모든 코드는 저의  [깃허브](https://github.com/earthquakoo/FastAPI-Base-Project)에서 확인 가능합니다.


## create token


`/src/auth/router.py`
```python
@router.post('/login', response_model=schemas.Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends(), db: Session = Depends(get_db)):
    user = service.authenticate_user(db, form_data.username, form_data.password)
    if not user:
        raise exceptions.InvalidEmailOrPasswordException()
    
    if service.get_current_active_user(db ,email=user.email, is_activate=False):
        raise exceptions.UnregisteredEmail()
    
    access_token = utils.create_access_token(data={"sub": user.email})
    refresh_token = utils.create_refresh_token(data={"sub": user.email})
    
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "refresh_token": refresh_token
        }
```
로그인과 동시에 access token과 refresh token을 생성합니다. 이때 `"sub": user.email`로 유저의 이메일 정보를 token에 담습니다.

또한 아래 함수를 통해 `"exp": expire`, `"type": "token type"` 라는 정보를 token에 추가적으로 담습니다. `"exp"`는 token의 유효기간으로 access token은 2시간, refresh token은 2주일로 설정했습니다. 

`/src/auth/utils.py`
```python
from src.auth.config import get_jwt_settings

jwt_settings = get_jwt_settings()

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=jwt_settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({
        "exp": expire,
        "type": "access_token"
    })
    encoded_jwt = jwt.encode(to_encode, jwt_settings.SECRET_KEY, algorithm=jwt_settings.ALGORITHM)
    return encoded_jwt


def create_refresh_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=jwt_settings.REFRESH_TOKEN_EXPIRE_MINUTES)        
    to_encode.update({
        "exp": expire,
        "type": "refresh_token"
    })
    encoded_jwt = jwt.encode(to_encode, jwt_settings.SECRET_KEY, algorithm=jwt_settings.ALGORITHM)
    return encoded_jwt
```
이후 `.env`에 저장해둔 `SECRET_KEY`, `ALGORITHM`으로 암호화합니다.


## Update access token 

```python
def decode_access_jwt(token):
    try:
        payload = jwt.decode(token, jwt_settings.SECRET_KEY, algorithms=[jwt_settings.ALGORITHM])
        if payload["type"] == "access_token":
            email: str = payload.get("sub")
            return email
        raise HTTPException(status_code=401, detail='Scope for the token is invalid')
    except ExpiredSignatureError:
            raise HTTPException(status_code=401, detail='Token expired')


def decode_refresh_jwt(token):
    try:
        payload = jwt.decode(token, jwt_settings.SECRET_KEY, algorithms=[jwt_settings.ALGORITHM])
        if payload["type"] == "refresh_token":
            email: str = payload.get("sub")
            return email
        raise HTTPException(status_code=401, detail='Scope for the token is invalid')
    except ExpiredSignatureError:
            raise HTTPException(status_code=401, detail='Token expired')
```
암호화된 token을 복호화 시키고 해당 token의 type이 일치하면 token안에 저장되어있던 유저 이메일 정보를 반환합니다.


```python
from fastapi import Security
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer

security = HTTPBearer()

def auth_access_wrapper(auth: HTTPAuthorizationCredentials = Security(security)):
	return decode_access_jwt(auth.credentials)


def auth_refresh_wrapper(auth: HTTPAuthorizationCredentials = Security(security)):
	return decode_refresh_jwt(auth.credentials)
```
`HTTPAuthorizationCredentials`를 통해 인증된 token의 정보를 가져와 복호화시킨 값을 반환합니다.


```python
@router.get("/update_token")
def update_token(user=Depends(utils.auth_refresh_wrapper)):
    if user is None:
        raise exceptions.CredentialsException()
    new_token = utils.create_access_token({"sub": user})
    return {"access_token": new_token}
```
`update_token` 함수를 통해 refresh token의 유효기간이 지나지 않았다면 새로운 access token을 발급합니다.
![img](/assets/img/dev/updatetoken.PNG)
`update_token`의 자물쇠 버튼을 누르면 아래의 창이 나옵니다.

![img](/assets/img/dev/updatetoken2.PNG)
로그인시 발급받았던 refresh token을 입력하고 Authorize합니다.

![img](/assets/img/dev/updatetoken3.PNG)
만약 refresh token의 유효기간이 남아있다면 excute시 새로운 access token을 발급합니다.

---
layout: post
title:  "[FastAPI] 4. Authentication"
subtitle:   "FastAPI base project"
categories: dev
tags: fastapi python
comments: true
---

1. Authentication settings
2. User schema
3. Register
4. Email authentication 
5. Login
---

## Authentication settings

이전에 나왔던 데이터베이스와 이메일 세팅과 동일합니다. 

`/src/auth/config.py`
```python
from functools import lru_cache
from dotenv import load_dotenv
from pydantic import BaseSettings

load_dotenv('.env')


class JWTSettings(BaseSettings):
    SECRET_KEY: str
    ACCESS_TOKEN_EXPIRE_MINUTES: int
    REFRESH_TOKEN_EXPIRE_MINUTES: int
    ALGORITHM: str
    
    class Config:
        env_file = ".env"
        

@lru_cache() 
def get_jwt_settings():
    return JWTSettings()
```
Access token과 refresh token 두 개를 구현하기 위해 `secret key`를 발급 받고 ALGORITHM 변수를 만들어 `.env`에 저장해두었습니다.
- `openssl rand -hex 32` 커맨드로 임의의 `secret key`를 발급받습니다.
- `ACESS_TOKEN_EXPIRE_MINUTES=60*2`, `REFRESH_TOKEN_EXPIRE_MINUTES=60*24*7`
- `ALGORITHM=HS256`


## User Schema

`/src/auth/schemas.py`
```python
from typing import Union

from email_validator import validate_email, EmailNotValidError
from pydantic import BaseModel, validator, Field

import src.auth.exceptions as exceptions
import src.auth.exceptions as auth_exceptions

class UserBase(BaseModel):
    email: str
    
    @validator('email')
    def email_must_be_valid(cls, v):
        try:
            validation = validate_email(v)
            email = validation.email
        except EmailNotValidError as e:
            raise auth_exceptions.EmailNotValidException()
        return email


class UserCreate(UserBase):
    password: str = Field(default=..., min_length=4, max_length=30)
    

class UserCreateOut(UserBase):
    user_id: int
    
    class Config:
        orm_mode = True
```

Pydantic의 BaseModel을 상속받아 User 정보의 base가 되는 `UserBase` 스키마를 만들었습니다. `UserBase`를 상속받아서 회원가입시 필요한 정보인 email과 password를 입력할 `UserCreate` 스키마도 만들어줍니다.  `UserCreateOut`은 end point에서 response_model로 이용할 예정입니다.

`orm_mode`가 `True`로 설정되면 dict 자료형이 아닌데도 dict 자료형처럼 읽을 수 있게 해주고 SQLAlchemy의 ORM 모델처럼 사용할 수 있게 해줍니다.    

ex) `user.user_id`


```python
class VerificationCode(UserBase):
    verification_code: str = Field(default=..., max_length=6)
```
회원가입시 이메일 인증코드를 발송하게 되는데 그 인증코드를 확인하는 pydantic model입니다.


```python
class Token(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str


class TokenData(BaseModel):
    username: Union[str, None] = None
```
Token과 관련된 pydantic model로 end point에서 사용됩니다.


## Register

회원가입 절차에 대한 end point router입니다.

`/src/auth/router.py`
```python
@router.post('/register', status_code=status.HTTP_201_CREATED, response_model=schemas.UserCreateOut)
async def register(user: schemas.UserCreate, background_tasks: BackgroundTasks, db: Session = Depends(get_db)):
    user_dict = user.dict()
    
    # Check if there's a duplicated email in the DB
    if service.get_current_active_user(db ,email=user_dict['email'], is_activate=True):
        raise exceptions.EmailAlreadyExistsException(email=user_dict['email'])   
    
    # Send verification code
    verification_code = utils.generate_verification_code(len=6)
    recipient = user_dict['email']
    subject="[Girok] Please verify your email address"
    content = utils.read_html_content_and_replace(
        replacements={"__VERIFICATION_CODE__": verification_code},
        html_path="src/email/verification.html"
    )
    background_tasks.add_task(email_utils.send_email, recipient, content, subject)
    
    # Hash verification code
    hashed_verification_code = utils.hash_verification_code(verification_code)
    user_dict.update(verification_code=hashed_verification_code)
    
    # Hash password
    hashed_password = utils.hash_password(user_dict['password'])
    user_dict.update(password=hashed_password)
    
    if service.get_current_active_user(db ,email=user_dict['email'], is_activate=False):
        update_user = db.query(glob_models.User).filter(glob_models.User.email==user_dict['email']).first()
        update_user.password = hashed_password
        update_user.verification_code = hashed_verification_code
        
        db.add(update_user)
        db.commit()
        db.refresh(update_user)
        
        return update_user
    
    new_user = glob_models.User(**user_dict)
    
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    
    return new_user
```



```python
@router.post('/register', status_code=status.HTTP_201_CREATED, response_model=schemas.UserCreateOut)
async def register(user: schemas.UserCreate, background_tasks: BackgroundTasks, db: Session = Depends(get_db)):
```
Response_model은 `UserCreateOut`으로 출력 데이터 모델을 지정해주었습니다. [BackgroudTasks](https://fastapi.tiangolo.com/tutorial/background-tasks/)는 응답을 반환한 후 실행할 백그라운드 작업을 정의합니다. 여기서는 이메일 인증 코드를 유저에게 보내는 역할을 합니다.
![img](/assets/img/dev/post1.PNG)
이런 식으로 이메일과 패스워드를 입력하면 response_model로 `UserCreateOut`에서 설정해둔 형태로 출력됩니다.


```python
    if service.get_current_active_user(db ,email=user_dict['email'], is_activate=True):
        raise exceptions.EmailAlreadyExistsException(email=user_dict['email'])
```
`get_current_activate_user` 함수는 인자로 유저의 이메일과 유저의 활성화 여부를 넘기면 해당 유저의 정보를 반환합니다. 만약 유저가 입력한 이메일이 이미 가입되어 있다면 예외처리로 에러를 발생시켜줍니다.


```python
    # Send verification code
    verification_code = utils.generate_verification_code(len=6)
    recipient = user_dict['email']
    subject="[Girok] Please verify your email address"
    content = utils.read_html_content_and_replace(
        replacements={"__VERIFICATION_CODE__": verification_code},
        html_path="src/email/verification.html"
    )
    background_tasks.add_task(email_utils.send_email, recipient, content, subject)
    
    # Hash verification code
    hashed_verification_code = utils.hash_verification_code(verification_code)
    user_dict.update(verification_code=hashed_verification_code)
```
유저가 입력한 이메일로 인증코드를 발송하는 코드입니다. 생성한 코드를 `verification.html` 안에 `__VERIFICATION_CODE__` 넣어 메일을 보냅니다. 이 인증코드는 hashing해서 데이터베이스에 저장하고 인증코드를 확인하는 과정에서 조회됩니다.


```python
    # Hash password
    hashed_password = utils.hash_password(user_dict['password'])
    user_dict.update(password=hashed_password)
    
    if service.get_current_active_user(db ,email=user_dict['email'], is_activate=False):
        update_user = db.query(glob_models.User).filter(glob_models.User.email==user_dict['email']).first()
        update_user.password = hashed_password
        update_user.verification_code = hashed_verification_code
        
        db.add(update_user)
        db.commit()
        db.refresh(update_user)
        
        return update_user
```
만약 유저가 입력한 이메일이 데이터베이스에 저장은 되어 있으나 활성화가 되어 있지 않으면 새로 입력한 비밀번호와 코드를 저장하고 이를 데이터베이스에 업데이트합니다

예를 들어, 유저가 오늘 회원가입을 진행하다가 갑자기 회원가입 하기 싫어졌다고 합니다. 그렇다면 데이터베이스에 유저의 정보는 저장되어 있는 상태지만 `is_activate`가 활성화되지 않았습니다. 그러다가 며칠 뒤 다시 회원가입을 진행한다고 했을 때 원래의 유저 정보 중 비밀번호와 이메일 인증 코드를 업데이트해 다시 회원가입을 진행하는 방식입니다.


```python
    new_user = glob_models.User(**user_dict)
    db.add(new_user) 
    db.commit()
    db.refresh(new_user)
```
만약 데이터베이스에 저장되어 있지 않은 새로운 유저라면 변경된 유저 정보를 SQLAlchemy 모델로 넘겨 데이터베이스 추가하고 저장합니다.
![img](/assets/img/dev/database1.PNG)
데이터베이스를 조회해보면 저장이 잘 되어있는 것을 확인할 수 있습니다.

## Email authentication

```python
@router.post("/register/verification_code", status_code=status.HTTP_200_OK)
async def verify_email(user: schemas.VerificationCode, db: Session = Depends(get_db)):
    user_dict = user.dict()
    
    user = db.query(glob_models.User).filter(glob_models.User.email == user_dict['email']).first()
    
    if service.get_current_active_user(db ,email=user_dict['email'], is_activate=True):
        raise exceptions.EmailAlreadyExistsException(email=user_dict['email'])   
    
    if not utils.verify_code(user_dict['verification_code'], user.verification_code):
        raise exceptions.InvalidVerificationCode()

    user.is_activate = True
    
    db.add(user)
    db.commit()
    db.refresh(user)
    
    return "Email authentication is complete."
```
데이터베이스에 저장된 verification code와 유저가 입력한 verification code가 다르다면 오류를 발생시키고 같다면 `is_activate=True`로 유저를 활성화시켜줍니다. 이후 데이터베이스에 업데이트된 내용을 추가합니다.
![img](/assets/img/dev/database2.PNG)

## Login

```python
@router.post('/login', response_model=schemas.Token)
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: Session = Depends(get_db)
    ):
    user = service.authenticate_user(db, form_data.username, form_data.password)
    if not user:
        raise exceptions.InvalidEmailOrPasswordException()
        
    access_token = utils.create_access_token(data={"sub": user.email})
    refresh_token = utils.create_refresh_token(data={"sub": user.email})
    
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "refresh_token": refresh_token
        }
```
`OAuth2PasswordRequestForm`은 클래스 디펜던시로 사용자 인증을 위해 이용하고 아래와 같은 form body를 가집니다. 
- `uesrname`
- `password`
- `scope(Optional)`
- `grant_type(Optional)`
- `client_id(Optional)`
- `client_secret(Optional)`
유저가 입력한 이메일과 패스워드가 데이터베이스에 저장된 것과 일치하지 않다면 오류를 발생하고 일치하다면 access token과 refresh token을 발급 후 로그인시킵니다.

> Refresh token은 access token과 동일하게 로그인할 때마다 발급되고 refresh token은 access token이 만료되었을 때 access token을 새로 발급해주는 역할을 합니다. 

JWT의 이론적인 부분이 궁금하시다면 [JWT 인증방식과 Access token, refresh token](https://earthquakoo.github.io/study/2023/03/26/review-book-hyperledger-fabric/)
JWT의 구현 과정이 궁금하시다면 [Access token과 refresh token 구현](https://earthquakoo.github.io/dev/2023/03/27/refreshtoken/) 를 참고해주시면 되겠습니다.
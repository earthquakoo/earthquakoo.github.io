---
layout: post
title:  "[FastAPI] 4. Authentication"
subtitle:   "FastAPI base project"
categories: dev
tags: fastapi python
comments: true
---

## JWT 인증

JWT 방식은 로그인을 하면 암호화된 token을 발급하는데, token은 유저 정보를 포함한 JSON이 반환됩니다. 암호화된 token을 복호화하기 위해 `secret key`가 필요합니다. API에서 호출할 시 token을 헤더에 포함시켜 호출하는데 이 때 유저정보를 포함하기 때문에 db를 조회할 필요가 없어집니다.

하지만 JWT 인증방식은 token이 한 번 발급되면 보안상 문제가 되었을 때 무력화시킬 방법이 없다는 점입니다. 그래서 유효기간을 두는데 이 유효기간이 짧으면 계속 로그인을 해야되고 유효기간이 길어지면 보안상 문제가 발생합니다. 그래서 이 문제를 어느정도 방지하기 위해 access token과 refresh token 둘 다 사용하는 것입니다.

Refresh token은 access token과 동일하게 로그인할 때마다 발급되고 refresh token은 access token이 만료되었을 때 access token을 새로 발급해주는 역할을 합니다. 

전체적인 플로우는 [Refresh Tokens: Adding Refresh Spice to User Auth Flow in FastAPI.](https://philipokiokio.hashnode.dev/refresh-tokens-user-auth-fastapi) 이 자료를 참고했습니다.


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
- `openssl rand -hex 32` 커맨드로 token을 복호화할 임의의 `secret key`를 발급받습니다.
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

여기서 주목해야할 점은 `UserBase`라는 클래스에서 email만 받는 것입니다. 만약 유저가 password를 입력하고 이를 바로 데이터베이스에 저장하게 되면 보안상 문제가 발생합니다. 이를 방지하기 위해 나중에 hashing한 password를 받아올 예정입니다.

`UserBase`를 상속받아서 회원가입시 필요한 정보인 email과 password를 입력할 `UserCreate` 스키마를 만듭니다.


```python
class VerificationCode(BaseModel):
    email: str
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


## 회원가입

회원가입 절차에 대한 end point router입니다. 대부분의 내용은 공식문서에서 확인할 수 있으나 제가 변환한 코드나 예외들을 하나씩 살펴보겠습니다.

`/src/auth/router.py`
```python
@router.post('/register', status_code=status.HTTP_201_CREATED, response_model=schemas.UserCreateOut)
async def register(user: schemas.UserCreate, background_tasks: BackgroundTasks, db: Session = Depends(get_db)):
    user_dict = user.dict()
    
    # Check if there's a duplicated email in the DB
    if service.get_current_active_user(db ,email=user_dict['email'], is_activate=True):
        raise exceptions.EmailAlreadyExistsException(email=user_dict['email'])   
    
    if service.get_current_active_user(db ,email=user_dict['email'], is_activate=False):
        update_user = db.query(glob_models.User).filter(glob_models.User.email==user_dict['email']).first()
        db.delete(update_user)
        db.commit()
    
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
- Response_model은 `UserCreateOut`으로 설정해 유저의 비밀번호가 유출되는 것을 방지해줍니다. 
- [BackgroudTasks](https://fastapi.tiangolo.com/tutorial/background-tasks/)는 응답을 반환한 후 실행할 백그라운드 작업을 정의합니다.
	- 여기서는 이메일 인증 코드를 유저에게 보내는 역할을 합니다.

```python
    if service.get_current_active_user(db ,email=user_dict['email'], is_activate=True):
        raise exceptions.EmailAlreadyExistsException(email=user_dict['email'])   
    
    if service.get_current_active_user(db ,email=user_dict['email'], is_activate=False):
        update_user = db.query(glob_models.User).filter(glob_models.User.email==user_dict['email']).first()
        db.delete(update_user)
        db.commit()
```
- 유저가 입력한 이메일이 이미 가입되어 있다면 예외처리를 해주고 입력한 이메일이 데이터베이스에는 있으나 등록이 되어있지 않다면 해당 이메일을 데이터베이스에서 삭제해줍니다.
	- `is_activate`로 유저가 등록되어있는지 아닌지를 확인합니다.

`/src/auth/service.py`
```python
def get_current_active_user(db: Session, email: EmailStr, is_activate: bool):
    return db.query(glob_models.User).filter(glob_models.User.email == email, glob_models.User.is_activate == is_activate).first()
```


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
- 입력한 이메일로 인증코드를 발송하는 코드입니다.
- 이 인증코드는 hashing해서 데이터베이스에 저장하고 나중에 인증코드를 확인하는 과정에서 조회됩니다.


```python
    # Hash password
    hashed_password = utils.hash_password(user_dict['password'])
    user_dict.update(password=hashed_password)
    
    new_user = glob_models.User(**user_dict)
    db.add(new_user) 
    db.commit()
    db.refresh(new_user)
```
- 유저가 입력한 비밀번호를 hashing해서 데이터베이스에 저장합니다.
- 이후 변경된 유저 정보를 데이터베이스에 추가시키고 저장합니다.


## 이메일 인증

`/src/auth/router.py`
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


## 로그인

`/src/auth/router.py`
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
- 로그인과 동시에 access token과 refresh token을 발급합니다.


## Access token 새로 발급

```python
@router.get("/update_token")
def update_token(username=Depends(utils.auth_refresh_wrapper)):
    if username is None:
        raise HTTPException(status_code=401, detail="not authorization")
    new_token = utils.create_access_token({"sub": username})
    return {"access_token": new_token}
```
- 
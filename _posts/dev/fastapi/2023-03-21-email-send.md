---
layout: post
title:  "[FastAPI] 3. Email send"
subtitle:   "FastAPI base project"
categories: dev
tags: fastapi python
comments: true
header-img:
---

## Email send

이메일 인증에 필요한 메일 발송 세팅을 살펴보겠습니다. 이메일인증은 구글 이메일을 통한 것으로 구현하겠습니다. 구글 이메일을 통해 메일을 발송하기 위해 몇 가지 설정을 해야했습니다.

- [구글 앱 비밀번호 설정](https://support.google.com/accounts/answer/185833?hl=ko) 절차를 통해 2단계 인증을 만들고 앱 비밀번호를 설정합니다.
- `.env` 환경변수 파일에 이메일을 보낼 계정(ex. xxxx@gmail.com)과 생성된 앱 비밀번호 16자리를 저장합니다.
	- `GMAIL_SENDER=xxxxx@gmail.com`
	- `GMAIL_APP_PASSWORD=xxxxxxxxxxxxxxxx`

이후 메일 발송 세팅은 데이터베이스 세팅과 비슷하고  [FastAPI-MAIL](https://sabuhish.github.io/fastapi-mail/example/) 을 참조했습니다.

`/src/email/config.py`
```python
from functools import lru_cache
from dotenv import load_dotenv
from pydantic import BaseSettings

load_dotenv('.env')


class EmailSettings(BaseSettings):
    GMAIL_SENDER: str
    GMAIL_APP_PASSWORD: str
    
    class Config:
        env_file = ".env"
        
        
@lru_cache()
def get_email_settings():
    return EmailSettings()
```

메일 발송에 필요한 정보들을 `.env` 환경변수에서 가져와 `pydantic.BaseSettings`를 통해 서브 클래스로 만들어 주고 함수를 통해 인스턴스를 반환해주었습니다.


Python에서 메일을 보내기 위해서는 [smtplib](https://docs.python.org/ko/3/library/smtplib.html) 라는 모듈을 사용해야합니다. SMTP(Simple Mail Transfer Protocol)은 메일을 보내는데 사용되는 프토콜입니다. 

SMTP 서버에 접속하기 위해서는 SMTP 서버와 포트를 이용해 SMTP 객체를 생성해야합니다. 저는 구글 서버(`host="smtp.gmail.com"`, `port="587"`)를 이용해 메일을 보내고자 합니다.

`/scr/email/email.py`
```python
import smtplib

from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from src.email.config import get_email_settings

settings = get_email_settings()

class EmailSender:
    def __init__(self, email, app_password):
        try:
            self.smtp = smtplib.SMTP(host="smtp.gmail.com", port="587")

            self.smtp.ehlo()
            self.smtp.starttls()
            self.smtp.login(email, app_password)
            self.email = email
        except Exception as e:
            print("Error message: ", e)

        pass

    def send_email(self, to_email, html_content, subject):
        try:
            content = MIMEMultipart()
            content["subject"] = subject
            content["from"] = self.email
            content["to"] = to_email
            content.attach(MIMEText(html_content, "html"))

            self.smtp.send_message(content)
        except Exception as e:
            print("Error message: ", e)

    def __del__(self):

        self.smtp.quit()
        pass


email_sender = EmailSender(
    settings.GMAIL_SENDER, settings.GMAIL_APP_PASSWORD)
```

- `init`을 통해 SMTP 서버에 연결하고 계정과 암호가 인증되면 사용자 인증을 받게 됩니다.
- 이후 `send_email` 메서드를 사용하여 송신자와 수신자를 설정하고 html을 첨부하여 메일을 발송하게 됩니다.
---
layout: post
title:  "[FastAPI] 3. Email send"
subtitle:   "FastAPI base project"
categories: dev
tags: fastapi python
comments: true
header-img:
---

1. Email send
2. Email templates
  
---

## Email send

이메일 인증은 구글 stmp를 통해 진행했습니다. 구글 이메일로 메일을 발송하기 위해선 몇 가지 설정을 해야합니다.

- [구글 앱 비밀번호 설정](https://support.google.com/accounts/answer/185833?hl=ko) 절차를 통해 2단계 인증을 만들고 앱 비밀번호를 설정합니다.
- `.env` 파일에 이메일을 보낼 계정(ex. xxxx@gmail.com)과 생성된 앱 비밀번호 16자리를 저장합니다.
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

Python에서 메일을 보내기 위해서는 [smtplib](https://docs.python.org/ko/3/library/smtplib.html) 라는 모듈을 사용해야합니다. SMTP(Simple Mail Transfer Protocol)은 메일을 보내는데 사용되는 프로토콜입니다. 

SMTP 서버에 접속하기 위해서는 SMTP 서버와 포트를 이용해 SMTP 객체를 생성해야합니다. 저는 구글 서버(`host="smtp.gmail.com"`, `port="587"`)를 이용했습니다.

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
SMTP 서버에 연결하고 계정과 암호가 인증되면 사용자 인증을 받게 됩니다. 이후 `send_email` 메서드를 사용하여 제목과 송신자, 수신자를 설정하고 html을 첨부하여 메일을 발송하게 됩니다.

## Email templates

이메일 템플릿은 [unlayer](https://unlayer.com/features?gclid=CjwKCAjwoIqhBhAGEiwArXT7KxDUdn1swo8y0dOnUJtEeirLkpjHT2eMlzc6gAdncEIO7mcjsyswixoCi3cQAvD_BwE) 라는 사이트에 제작을 했습니다. GUI를 통해 손쉽게 메일 템플릿을 제작할 수 있어 편리합니다!

제가 만든 메일 템플릿은 아래와 같으며 수정해서 사용하길 원한다면 `/src/auth/verification.html` 파일을 수정해서 사용하시면 되겠습니다. 
![img](/assets/img/dev/emailtemplates.PNG)
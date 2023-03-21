---
layout: post
title:  "[FastAPI] 1. FastAPI base project"
subtitle:   "FastAPI base project"
categories: dev
tags: fastapi python
comments: true
header-img:
---

## 개요

'Girok'이라는 프로젝트를 진행하면서 FastAPI로 회원가입 + 이메일인증, 로그인 기능 구현을 담당하게 되었습니다. 이를 구현하면서 다른 웹프로젝트를 진행해야할 때도 회원가입과 로그인 기능을 높은 확률로 만들게 될 것 같았습니다.

그렇기에 만들어놓은 코드를 FastAPI Base Project라 명명해 다른 프로젝트가 있을 때 재활용하고자 했습니다. 완벽한 코드는 아니지만 재활용을 거쳐 완성도 있는 코드가 될 때까지 꾸준히 업데이트할 예정입니다.

**프로젝트가 진행된 이후에 코드 베이스를 가지고 작성하는 리뷰형식입니다. 공식문서를 어느정도 숙지를 하셔야 이해하기 편하실거라 생각합니다.**

모든 코드는 저의 [깃허브](https://github.com/earthquakoo/FastAPI-Base-Project)에 다 나와있습니다.


## 프로젝트 구조

FastAPI 공식문서에 기초적인 프로젝트 구조가 나와습니다. 하지만 'Girok' 프로젝트를 진행하면서 팀원과의 제의받은 프로젝트 구조는 아래와 같으며 [Fastapi-best-practices](https://github.com/zhanymkanov/fastapi-best-practices)를 참고해 서비스별로 구조를 만들게 되었습니다.

```
├── src
│   ├── auth
|   |   ├── config.py
│   |   ├── models.py  
│   │   ├── exceptions.py
│   │   ├── router.py
│   │   ├── schemas.py
│   │   ├── service.py
│   │   └── utils.py
│   ├── email
|   |   ├── config.py
│   │   ├── email.py
│   │   └── verification.html
│   ├── config.py
│   ├── database.py 
│   ├── dependencies.py
│   ├── exceptions.py
│   ├── models.py
│   └── main.py
├── requirements
│   └── dev.txt
├── .env
├── .gitignore
└── venv
```

 - `src` : 가장 상위에 존재하는 폴더
	- `auth` : Authentication(인증)을 서비스로 회원가입, 로그인, 이메일 인증에 관한 폴더
	- `email` : 이메일 인증을 위해 메일을 보내는 서비스와 메일 내용에 관한 폴더
	- `requirements` : 패키지 폴더

- `router.py` : End point로 각 기능의 핵심
- `shemas.py` : Pydantic model
- `models.py` : Database model
- `service.py` : 모듈별 비즈니스 로직
- `dependencies.py` : Router의 종속
- `config.py` : 환경 변수 설정
- `utils.py` : 비즈니스와 다르게 뒤에서 동작하는 로직
- `exceptions.py` : 모듈별 예외처리
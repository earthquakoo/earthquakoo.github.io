---
layout: post
title:  "JWT 인증 방식과 Access token, refresh token"
subtitle:   "JWT, access token, refresh token"
categories: study
tags: fastapi, python, jwt, access token, refresh token
comments: true
---

## JWT란?

JWT(Json Web Token)은 선태적 서명 및 선택적 암호화를 사용하여 데이터를 만들기 위한 인터넷 표준 인증 방식입니다. 즉, 인증에 필요한 정보들을 token에 담아 암호화시켜 사용하는 방식입니다.

JWT는 공개/개인 키를 쌍으로 이용하여 toekn에 서명할 경우 서명된 토큰은 개인 키를 보유한 서버가 서명된 token이 정상적인 토큰인지 확인하고 인증합니다. 이러한 JWT 구조는 인증 정보를 담아 안전하게 인증을 시도할 수 있게끔 전달합니다.

JWT는 header, payload, signature 세 가지 정보를 인코딩한 값을 콤마를 사이에 두고 이어붙이 형태로 생성됩니다. 이때 비대칭 암호화 방식을 사용하기 때문에 서버측에서는 이 token을 받아서 signature를 복호화하여 디코딩하는 방식으로 token의 유효성을 검증할 수 있습니다.

## JWT 인증 과정

JWT 방식은 로그인을 하면 암호화된 token을 발급하는데, token은 유저 정보를 포함한 JSON이 반환됩니다. 암호화된 token을 복호화하기 위해 secret key가 필요합니다. API에서 호출할 시 token을 헤더에 포함시켜 호출하는데 이 때 유저정보를 포함하기 때문에 db를 조회할 필요가 없어집니다.

하지만 JWT 인증방식은 token이 한 번 발급되면 보안상 문제가 되었을 때 무력화시킬 방법이 없다는 점입니다. 그래서 유효기간을 두는데 이 유효기간이 짧으면 계속 로그인을 해야되고 유효기간이 길어지면 보안상 문제가 발생합니다. 그래서 이 문제를 어느정도 방지하기 위해 access token과 refresh token 둘 다 사용하는 것입니다.

Refresh token은 access token과 동일하게 로그인할 때마다 발급되고 refresh token은 access token이 만료되었을 때 access token을 새로 발급해주는 역할을 합니다. 

## JWT 구성

### 1. Header
```python
{
 "typ": "JWT",
 "alg": "HS256"
}
```
Header의 경우 token의 타입이나 서명 생성에 사용된 알고리즘을 저장합니다.

### 2. Payload
```python
{
 "sub": "user_email",
 "iss": "me",
 "exp": 123456789,
 "iat": 123456789
}
```
payload는 Claim이라는 사용자에 대한 혹은 token에 대한 property를 key-value의 형태로 저장합니다. Claim은 말 그래도 token에서 사용할 정보의 조각, 혹은 본인이 넣고자 하는 정보를 뜻합니다.

표준 스펙에서는 key의 이름을 3글자로 표기했습니다. JWT의 핵심 목표는 사용자에 대한, 토큰에 대한 표현을 압축하는 것이기 때문에 이렇게 표기한다고도 볼 수 있습니다. 

- **iss**(Issuer) : 토큰 발급자
- **sub**(Subject) : 토큰 제목(혹은 토큰에서 사용자에 대한 식별 값)
- **aud**(Audience): 토큰 대상장
- **exp**(Expiration Time) : 토큰 만료 기한
- **nbf**(Not before) : 토큰 활성화 날짜
- **iat**(Issued at) : 토큰 발급 시간
- **jti**(JWT id) : JWT 토큰 식별자(issuer가 여러 명일 때 구분하기 위해 사용)

이러한 표준 스펙으로 정의되어 있는 7가지를 모두 포함할 필요는 없습니다. 표준 스펙 이외에도 사용자가 필요하다 싶으면 추가해도 전혀 문제가 없다고 합니다.

### 3. Signature
```
HMACSHA256(
	base64UrlEncode(header) + "."+
	base65UrlEncode(payload),
	your-256-bit-secret
)
```
Signature에서 사용하는 알고리즘은 header에서 정의한 알고리즘 방식을 활용합니다. Signature의 구조는 header+payload와 서버가 가지고 있는 유일한 key 값을 합친 것으로 header에서 정의한 알고리즘으로 암호화합니다.


## Access Token

유저의 로그인 상태를 유지하기 위해 로그인시 서버는 클라이언트에게 access token은 전달합니다. 클라이언트는 access token을 가지고 서버와 통신하며 로그인 상태를 유지합니다. Access token은 따로 정보를 저장하는 것이 아닌 token에 담긴 유저 정보를 확인해 해당 유저가 로그인 상태라는 것을 인지하는 방식입니다.

하지만 access token은 만료기간이 되기 전까지 꾸준히 사용되는데 이 때문에 token에 있는 유저 정보가 노출될 수 있습니다. 

이를 해결하기 위해 access token에 만료 기간을 부여하여 만약 token이 노출되더라도 그 유효 시간을 짧게 조절해 보안을 강화할 수 있습니다. 또한 refresh token을 사용한다면 만료 기간이 지난 access token을 재발급하여 로그인 상태를 유지시킬 수 있습니다.

## Refresh Token

Refresh token은 access token의 만료 기간을 짧고, 자주 재발급 하도록 만들어 보안을 강화하면서도 사용자에게 잦은 로그아웃을 경험하지 않도록 하게 합니다. 

Access token은 리소스에 접근하기 위해서 사용된 token이라면 refresh token은 기존에 클라이언트가 가지고 있던 access token이 만료되었을 때 access token을 재발급 받기 위해 사용됩니다. 

Refresh token은 access token에 비해 긴 유효 기간을 가집니다. 예시로 제가 구현한 바로는 access token은 2시간의 유효 기간을 가지게 하였고, refresh token은 2주의 유효 기간을 가지도록 했습니다. 이는 서비스에 따라 적절한 유효기간을 설정하는 것이 좋을 것입니다.

## Refresh, access token 메커니즘

![img](/assets/img/study/mechanism.PNG)
클라이언트가 로그인을 요청하고 성공하면, 서버는 access token과 refresh token을 제공합니다. 이후 클라이언트는 인가에 필요한 요청에 access token을 실어 보냅니다. 시간이 흘러 access token이 만료 되었다면, 클라이언트는 refresh token을 서버로 전달하여 새로운 access token을 발급받습니다. 

**Reference**
- https://brunch.co.kr/@jinyoungchoi95/1
- https://velog.io/@jinyoungchoi95/JWTJson-Web-Token-%EC%9D%B8%EC%A6%9D%EB%B0%A9%EC%8B%9D
- https://daco2020.tistory.com/315

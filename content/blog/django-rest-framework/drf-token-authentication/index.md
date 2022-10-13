---
title: "drf 에서 token 을 사용한 인증 구현하기"
date: "2022-10-11"
last_modified_at: "2022-10-13"
category: django-rest-framework
---

## 0. 서론
어플리케이션을 위한 인증에 `drf TokenAuthentication` 를 사용하기로 했었는데 내가 구현하고자 하는 인증 프로세스는 다음과 같다.

### 0.1 인증 프로세스 요구사항
1. 이메일과 패스워드만을 사용하여 회원가입/로그인 한다.
2. 회원가입 시 발급되는 토큰을 request header 에 담아 사용자 인증 한다. 
3. 인증된 사용자만 API 에 접근 가능하다.

이것을 구현하는 방법을 정리한다.

### 0.2 User Model 정보
이메일과 패스워드만을 사용하기 위해 User Model 을 커스텀했다. 패스워드 필드는 기본적으로 추가되기 때문에 이렇게 구현하면 인증 프로세스 요구사항 1번을 만족하도록 할 수 있다.    
추후 연락처, 주소 등 다른 정보를 받아야 하는 필요가 생기면 그때 모델을 수정하여 필드를 추가 할 예정이다.
```python
from django.contrib.auth.base_user import AbstractBaseUser
from django.contrib.auth.models import UserManager
from django.db import models

class User(AbstractBaseUser):
    username = None
    email = models.EmailField(unique=True)
    
        USERNAME_FIELD = 'email'
        REQUIRED_FIELDS = []
    
        objects = UserManager()
    
        class Meta:
            db_table = 'auth_user'
            verbose_name = '사용자'
```

## 1. 설치
회원가입과 로그인 기능이 구현 되어 있는 패키지를 설치한다.

## 1.1 파이썬 패키지 설치
```shell
% pip install django-allauth
% pip install dj-rest-auth
```

`django-allauth` 는 회원 인증, 등록, 관리 및 써드파티(소셜) 인증을 처리하는 `django` 패키지이다.  
`dj-rest-auth` 는 회원 인증 및 등록에 대한 API 엔드포인트를 제공한다.

> 개발하면서 `django-allauth` 는 필요 없지 않나 생각했지만 소스코드를 자세히 들여다 보니 `dj-rest-auth` 의 회원 인증 관련된 API(`dj_rest_auth.registration`) 를 사용하려면 `django-allauth` 패키지가 필요하기 때문에 필수로 설치해야 한다.

## 1.2 settings.py 파일 수정
먼저 `INSTALLED_APPS` 에 앱을 추가한다. 
```python
INSTALLED_APPS = [
    ...
    'rest_framework.authtoken',
    'dj_rest_auth',
    'dj_rest_auth.registration',
    'allauth',
    'allauth.account',
    ...
]

```

다음으로는 DRF 의 `permission` 과 `authentication` 정책을 설정한다. 어떤 사용자에게 권한을 부여할지, 어떤 방식으로 인증하는지를 설정해 주는 것이다.  
아래 코드는 `모든 인증되지 않은 사용자 거부. 토큰으로 사용자 인증` 정도로 해석 하면 되는데, 이것으로 손쉽게 인증 프로세스 요구사항중 2, 3번 항목을 만족하도록 구현했다. 
```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
    )
}
```

끝으로 회원가입시 `username` 을 사용하지 않고 `email` 만을 사용 할 수 있도록 설정을 해준다. 
```python
ACCOUNT_AUTHENTICATION_METHOD = 'email'
ACCOUNT_EMAIL_REQUIRED = True
ACCOUNT_EMAIL_VERIFICATION = 'none'
ACCOUNT_UNIQUE_EMAIL = True
ACCOUNT_USERNAME_REQUIRED = False
ACCOUNT_USER_MODEL_USERNAME_FIELD = None
```

### 1.3 urls.py 파일 수정
이제 `dj_rest_auth` 의 엔드포인트를 사용할 수 있도록 `urls.py` 파일을 다음과 같이 수정한다.  
`allauth` 까지 추가해야 하는 이유는 `dj_rest_auth.registration` 도 같이 사용해야 하기 때문이다.
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    ...
    path('auth/', include('dj_rest_auth.urls')),
    path('auth/', include('dj_rest_auth.registration.urls')),
    path('auth/', include('allauth.urls')),
    ...
]
```

### 1.4 데이터베이스 migrate
이제 `migrate` 하여 테이블을 생성하면 준비는 모두 완료된다.
```shell
% python manage.py migrate
```

## 2. 사용하기
### 2.1 회원가입
#### 2.1.1 EndPoint
/auth/
#### 2.1.2 Method
POST
#### 2.1.3 request body
```json
{
    "email": "user005@localhost",
    "password1": "abc1234",
    "password2": "abc1234"
}
```
#### 2.1.4 response body
회원가입 시 인증에 사용 가능한 토큰을 리턴한다.
```json
{
    "key":"4bf78c2a6b09e038205427d1ee02070f860b1c6a"
}
```

### 2.2 로그인
#### 2.2.1 EndPoint
/auth/login/
#### 2.2.2 Method
POST
#### 2.2.3 request body
```json
{
    "email": "user005@localhost",
    "password": "abc1234"
}
```
#### 2.2.4 response body
로그인 시 회원가입때 생성된 토큰을 리턴한다.
```json
{
    "key":"4bf78c2a6b09e038205427d1ee02070f860b1c6a"
}
```

### 2.3 서비스 API 예제
`DEFAULT_PERMISSION_CLASSES` 을 설정함으로 인하여 HTTP 요청시 헤더에 토큰 값을 넣지 않는다면 해당 API 에 접근이 제한된다. 다음 예제에서 처럼 요청 헤더에 토큰 값을 넣어준다면 정상적으로 접근이 가능하다.  
#### 2.3.1 EndPoint
/api/user
#### 2.3.2 Method
GET
#### 2.3.3 request header
key : Authorization  
value : Token e19a42b1c77eb6c4a1373b5858001589e1531e20
#### 2.3.4 response body
```json
[
    {
        "id": 3,
        "email": "ohk9523@naver.com",
        "password": ""
    },
    {
        "id": 4,
        "email": "user003@localhost",
        "password": ""
    },
    {
        "id": 5,
        "email": "user004@localhost",
        "password": ""
    }
]
```
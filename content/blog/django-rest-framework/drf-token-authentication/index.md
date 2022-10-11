---
title: "drf token 인증"
date: "2022-10-11"
last_modified_at: "2022-00-00"
category: django-rest-framework
---

## 0. 서론
어플리케이션을 위한 인증에 `drf TokenAuthentication` 를 사용하기로 했었는데 내가 구현하고자 하는 인증 프로세스는 다음과 같다.

### 0.1 인증 프로세스
1. 이메일과 패스워드만을 사용하여 회원가입/로그인 한다.
2. 회원가입 시 발급되는 토큰을 사용하여 사용자 인증 한다. 
3. 인증된 사용자만 API 에 접근 가능하다.

이것을 구현하는 방법을 정리한다.

### 0.2 User Model 정보
이메일과 패스워드만을 사용하기 위해 User Model 을 커스텀했다. 연락처, 주소 등 다른 정보가 필요하게 되면 그때 수정할 예정이다.
```python
import ...

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
회원가입과 로그인 기능이 구현 되어 있는 
가장 먼저 패키지를 설치한다.

```shell
% pip install django-allauth
% pip install dj-rest-auth
```

`django-allauth` 는 회원 인증, 등록, 관리 및 써드파티(소셜) 인증을 처리하는 `django` 패키지이다.  
`dj-rest-auth` 는 회원 인증 및 등록에 대한 API 엔드포인트를 제공한다.

개발하면서 `django-allauth` 는 필요 없지 않나 생각했지만 소스코드를 자세히 들여다 보니 `dj-rest-auth` 의 회원 인증 관련된 API(`dj_rest_auth.registration`) 를 사용하려면 `django-allauth` 패키지가 필요하기 때문에 필수로 설치해야 한다.

## 2. 설정

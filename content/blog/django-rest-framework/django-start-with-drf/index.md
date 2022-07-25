---
title: "drf 사용하여 API 만들기"
date: "2022-07-22"
---

## 0. 서론
장고 프로젝트를 생성하는것부터 간단한 API 를 만드는 과정을 정리한다.

## 1. 장고 프로젝트 생성
프로젝트의 `root` 로 사용될 디렉토리를 먼저 생성하고 그 안에서 프로젝트를 생성 하는 명령어를 실행한다.  
나는 프로젝트 이름을 `config` 로 정했다.

```shell
$ mkdir ingredient-manage
$ cd ingredient-manage 
$ django-admin startproject config .
```

이렇게 하고 나면 다음과 같은 구조의 디렉토리와 파일을 볼 수 있는데, 모든 프로젝트 관련 파일은 `ingredient-manage` 디렉토리에 위치한다. 이후 여기에서 `app` 을 추가하는 방식으로 개발을 진행할것이다.

```
ingredient-manage
├── config
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    ├── wsgi.py
├── manage.py
```

## 2. IDE 설정
각 IDE 에 맞게 파이썬 인터프리터 및 테스트 환경을 설정해준다.

[Pycharm 설정 방법](/IDE/the-pycharm-i-know/)

## 3. django-rest-framework 설치
drf 를 사용하기 위해 필요한 패키지를 설치한다.

```shell
$ cd ingredient-manage
$ pip install djangorestframework
$ pip install django-filter 
```

## 4. app 생성하기
다음으로 app 을 하나 생성한다.  
각 기능별로 app 을 분리하여 재사용 할 수도 있지만 지금 단계에서는 하나의 app 에 모든 기능을 만들 예정이다.

```shell
$ django-admin startapp management
```

## 5. settings.py 파일 설정하기
### 5.1 INSTALLED_APPS
drf 와 방금 생성한 app 을 사용하기 위해 `INSTALLED_APPS` 에 drf 와 새로 생성한 app 을 추가했다.

```python
INSTALLED_APPS = [
...
    'rest_framework',
    'django_filters',
    'management.apps.ManagementConfig'
```

### 5.2 Internationalization
i18n 설정을 다음과 같이변경한다. 언어 코드와 타임존을 변경했고, DB 에 한국 시간(`KST`)을 저장하기 위해 `USE_TZ` 를 `False` 로 변경했다. 만약 `True` 상태라면 `UTC` 시간이 저장된다.

```python
LANGUAGE_CODE = 'ko-kr'

TIME_ZONE = 'Asia/Seoul'

USE_I18N = True

USE_L10N = True

USE_TZ = False
```

### 5.3 secret 파일
나는 이번 장고 프로젝트를 github 의 public repository 에 업로드 할 예정인데, 이렇게 하면 패스워드나 키가 공개되어 문제가 발생할 수 있다.   
그래서 이런 중요한 정보는 별도로 다른 파일에 보관하고 `settings.py` 에서 그 파일을 불러오도록 할 것이다. 

먼저 `django-secret.json` 이라는 파일을 프로젝트 루트 디렉토리(`ingredient-manage`)에 생성한다. 그 다음 공개하지 않고 싶은 패스워드나 키를 추가한다.
```json
{
  "DJANGO_SECRET_KEY": "",
  "DB_DEFAULT_ENGINE": "",
  "DB_DEFAULT_NAME": "",
  "DB_DEFAULT_USER": "",
  "DB_DEFAULT_PASSWORD": "",
  "DB_DEFAULT_HOST": "",
  "DB_DEFAULT_PORT": "",
  "PROD_ALLOWED_HOSTS": []
}
```

그다음으론 이 파일을 읽은 뒤 적절한 위치에 적절히 호출하면 된다.    
나는 `settings.py` 파일의 `BASE_DIR` 옵션 바로 밑에서 `django-secret.json` 파일을 불러왔다. 
```python
import json
from pathlib import Path

from django.core.exceptions import ImproperlyConfigured

...

BASE_DIR = Path(__file__).resolve().parent.parent

# get secret file
with open('./django-secret.json') as f:
    django_secret = json.loads(f.read())


def get_secret(setting, secrets=django_secret):
    try:
        return secrets[setting]
    except KeyError:
        error_msg = "Set the {0} enviroment varialble".format(setting)
        raise ImproperlyConfigured(error_msg)
        
...
```

그리고 아래처럼 호출했다.

```python
...

SECRET_KEY = get_secret("DJANGO_SECRET_KEY")

...
```

`SECRET_KEY` 뿐 아니라 DB 정보도 같은 방식으로 호출하면 중요한 정보를 노출하지 않을 수 있다.

### 5.4 settings 분리하기
이제 `settings.py` 파일을 분리한다.  개발과 운영 환경을 다르게 유지하기 위한것인데, `DEBUG` 옵션 처럼 각 운영 환경에서 다른 값을 설정해야 하는 경우 필요한 작업이다.

#### 5.4.1 python package 생성

![new-python-package](./541-0-new-python-package.png)

`config` 디렉토리에 `settings` 라는 이름의 `Python Package` 를 생성한 뒤 그 내부에 `base.py`, `local.py`, `prod.py` 파일 세개를 추가한다.  
이때 파일 구조는 아래 사진과 같다.

![directory-tree](./541-1-directory-tree.png)

#### 5.4.2 base.py 설정
다음으로는 `settings.py` 파일의 내용을 모두 복사해 `base.py` 파일에 붙여넣은 뒤 삭제한다.

![directory-tree](./542-directory-tree.png)

> `base.py` 파일은 모든 환경에서 공통으로 사용되는 설정을 포함한다.

#### 5.4.3 local.py, prod.py 설정
`local.py` 파일과 `prod.py` 파일은 각각 개발, 운영 환경에서만 필요한 설정을 포함하면 되는데, 나는 아래처럼 설정했다.

 

```python
# local.py
# 기본 설정 그대로 사용

from .base import *
```

```python
# prod.py
# 운영서버에서는 DEBUG 를 False 로 설정해야 한다

from .base import *

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = False
```

#### 5.4.4 local.py 를 사용하여 테스트서버 실행하기
방금전에 생성한 `local.py` 를 사용하여 테스트 서버를 실행한다.  
아래 사진처럼 `Environment variables` 옵션에 `DJANGO_SETTINGS_MODULE=config.settings.local` 를 추가해주면 이 설정파일로 테스트 서버가 실행된다.

![django-server-settings](./544-django-server-settings.png)

운영서버를 실행할 때에는 gunicorn 에 `DJANGO_SETTINGS_MODULE=config.settings.prod` 라는 옵션을 추가하면 된다.

## References
[1. django 튜토리얼](https://docs.djangoproject.com/ko/4.0/intro/tutorial01/)  
[1. 프로젝트 이름을 config 로 사용한 이유](https://forum.djangoproject.com/t/project-naming-conventions/339/12) 
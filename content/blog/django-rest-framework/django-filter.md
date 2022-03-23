---
title: "django-filter 라이브러리 사용법"
date: "2022-03-23"
---

## 0. 서론
API 엔드포인트에 쿼리스트링을 추가하여 필터링 된 데이터를 가져오고 싶어서 찾아보게 되었다.  
[django-rest-framework 공식 사이트](https://www.django-rest-framework.org/api-guide/filtering/#api-guide) 를 참고했다

## 1. 설치 후 설정하기
가장먼저 해당 라이브러리를 설치해야 한다
```shell
pip install django-filter
```
그 다음 django 에 사용할 수 있도록 설정한다. ```settings.py``` 파일의 적절한 위치에 아래 설정들을 추가한다.
```python
INSTALLED_APPS = [
...
'django_filters',
...
]
```

```python
REST_FRAMEWORK = {
'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend']
}
```

## 2. 적용하기
설치와 설정이 끝났다면 ViewSet 클래스에 ```filter_backends``` 와 ```filterset_fields``` 를 추가하여 아주 쉽게 적용할 수 있다.
```python
# views.py

from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.viewsets import ModelViewSet

from ..models import Product
from ..serializers import ProductSerializer

class ProductModelViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['business', 'name']
```
사용할 때에는 ```filterset_fields``` 에 포함된 컬럼을 쿼리스트링으로 사용하여 다음과 같이 요청하면 된다.  
```http://localhost:8000/api/product/?business=1&name=샘플```

## 3. 키워드를 포함하는 값 검색하기
하지만 위와 같이 사용하면 키워드와 정확히 일치하는 데이터만을 반환한다.  
django ORM 의 ```model.objects.filter=name__icontains='샘플'``` 과 같은 결과를 받고싶다면 다음과 같이 하면 된다.  

먼저 커스텀 필터를 추가해야 한다.  
따로 파일을 추가하는것이 관리하기 편할것 같아서 프로젝트 파일 안에 ```filter.py``` 파일을 추가하고 필터를 생성했다.
```python
# filters.py

import django_filters
from django_filters import FilterSet

from .models import Product


class ProductFilter(FilterSet):
    name = django_filters.CharFilter(lookup_expr='icontains')

    class Meta:
        model = Product
        fields = ['business', 'name']
```
```Meta``` 클래스의 ```fields``` 에는 정의했지만 ```ProductFilter``` 클래스에선 정의하지 않은 컬럼(business)의 경우 기존과 동일하게 정확히 일치하는 데이터를 가져온다.

이제 ```filterset_fields``` 를 ```filterset_class``` 로 변경 후 생성한 필터 클래스를 참조하도록 한다.
```python
# views.py

from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.viewsets import ModelViewSet

from ..filters import ProductFilter
from ..models import Product
from ..serializers import ProductSerializer

class ProductModelViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_class = ProductFilter
```

[출처](https://stackoverflow.com/questions/65158059/django-filters-icontains-type-of-lookup-expression-doesnt-work-properly)

## 99. 끝나고 나서
```serializer```, ```filter_backends```, ```filterset_class``` 등 ```ModelViewSet``` 의 파라미터들이 어떻게 작동하는지 정확히 알지 못하고 사용중이다.  
일단 front/back 분리가 완료되면 부족했던점에 대해 알아봐야 할것같다.
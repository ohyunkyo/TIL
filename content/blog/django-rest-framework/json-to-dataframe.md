---
title: "json 데이터를 dataframe 으로 변환하기"
date: "2022-04-12"
category: django-rest-framework
---

## 0. 서론
모델의 데이터를 가공하여 엑셀 파일을 리턴하는 기능을 만들어 사용중이였다.  
원하는대로 작동하긴 했지만 가독성도 떨어지고 복잡한 소스코드를 계속 사용할 생각은 없었다.  
그래서 drf 로 리팩토링 하는 과정에서 다른 방법을 사용하여 더 좋은 소스코드를 작성하고자 했다.

엑셀파일을 어디에서 생성할것인지가 첫번째 고민이였다.  
vue.js 에서 json 데이터로 엑셀 파일을 생성하는 라이브러리가 있다는것을 알게됐기 때문인데,  
관련 지식이나 경험이 없어서 친구의 조언대로 기존 방식(백엔드에서 생성 후 전달)을 사용하기로 했다.

엑셀파일을 생성할 때에는 하드코딩에 가까운 기존 방식보다는 pandas 라이브러리를 사용하는것이 더 적합하다고 생각했다.  
간단한 메서드로 엑셀파일을 생성 할 수 있는것처럼 보였고, dataframe 형식을 배워두면 유용하게 사용할 수 있을것 같았다.

## 1. 기본 예제 실습하기
pandas 예제에서는 queryset 으로 쉽게 dataframe 을 생성했다.  
그래서 비슷한 방식으로 `serializer.data` 를 넘기면 똑같이 dataframe 이 생성될것이라 기대했다.  

```python
# 처음 실습한 기본 예제

from django_pandas.io import read_frame

qs = PurchaseOrder.objects.all()
df = read_frame(qs)
```

```python
# serializer 를 사용하기위해 변형한 예제

...
qs = PurchaseOrder.objects.all()
serializer = PurchaseOrderSerializer(qs)
df = read_frame(serializer)
```

그러나 당연히 실패했다.  
`read_frame` 은 쿼리셋을 dataframe 으로 변환하는 메서드이기 때문이다.  

## 2. 구글링하기
당연히 serializer 를 dataframe 으로 바꿀 수 있는 방법이 있을것이라 생각했다.  
하지만 그중에 django-rest-pandas 는 업데이트된지도 너무 오래됐고 다른곳에서 활용하기 힘들것 같아서 제외하고 다른 방법을 찾아봤다.  

serializer to dataframe 같은 검색어로 시간을 허비하던 중 serializer 가 JSON 데이터를 리턴한다는 사실이 떠올랐다.

## 3. json 데이터를 dataframe 으로 변환하기
`json_normalize()` 메서드를 사용하여 json 데이터를 dataframe 으로 변환 할 수 있었다.

```python
# purchase_order_api.py

from pandas.io.json import json_normalize

queryset = PurchaseOrder.objects.filter(created_at__icontains=created_at).values(
...
)
serializer = PurchaseOrderDownloadSerializer(queryset, many=True)
df = json_normalize(serializer.data)
```

[출처](https://www.delftstack.com/ko/howto/python-pandas/json-to-pandas-dataframe/)
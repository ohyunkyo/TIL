---
title: "Serializer 에 대하여"
date: "2022-03-29"
category: django-rest-framework
---

## 역할
DRF 에서 데이터를 JSON 형태로 직렬화(Serialize) 한다.  
queryset 이나 모델 인스턴스 같은 데이터를 JSON 으로 바꿔준다고 생각하면 된다.

## 특징
Django 의 ModelForm 과 유사하다  
전달 받은 데이터의 유효성을 검사하고 형변환 할수 있다.  
Form 은 데이터를 HTML 형태로 변환 하지만 Serialize 는 JSON 문자열로 변환 해준다.

## serializer custom field
drf 를 사용한 API 서버에서 데이터를 조회할 때 기본적으로는 model 에 포함된 데이터만 조회가 가능하다.  
그런데 조회시에 model 에 포함되지 않은 커스텀 데이터를 보고싶다면 어떻게 해야할까??  
Serializer fields 중 `SerializerMethodField` 를 사용하면 된다.  

다음예제 처럼 사용하면 되는데, `fields` 옵션이 `'__all__'` 일 경우 자동으로 포함된다.
```python
class ProductHistorySerializer(serializers.ModelSerializer):
    business_name = serializers.SerializerMethodField()
    quantity_abs = serializers.SerializerMethodField()
    
    class Meta:
        model = ProductHistory
        fields = '__all__'
    
    def get_business_name(self, obj):
        return obj.business.name
    
    def get_quantity_abs(self, obj):
        return abs(obj.quantity)
```
나처럼 foreignKey 필드를 조회할 때 해당 모델의 id 가 아닌 다른 필드 값을 같이 조회하고 싶다면 아주 좋은 옵션인것 같다.  

[출처1](https://www.django-rest-framework.org/api-guide/fields/#serializermethodfield)
[출처2](https://ssungkang.tistory.com/entry/Django-Serializer-Custom-Field-SerializerMethodField)

## 가공된 데이터를 포함하는 DB Row 생성
form 에서 받은 데이터 그대로 저장하는것이 아닌 추가적인 데이터 가공이 필요한 경우 해당 데이터를 serializer 에 담아 같이 저장하도록 할 수 있다.
```
# views.py

class MyViewSet(ModelViewSet):
    queryset = ProductHistory.objects.all().order_by('-created_at')
    serializer_class = ProductHistorySerializer
    filter_backends = [DjangoFilterBackend]
    filterset_class = ProductHistoryFilter
    
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
    
        if serializer.is_valid():
            ...
            데이터 가공
            ...
            
            instance_data = {
                'business': business,
                'quantity': new_quantity,
                'expiry_at': new_expiry,
                'admin': admin
            }
            serializer.save(**instance_data)
            
            return Response(serializer.data, HttpStatus.HTTP_200_OK)
        else:
            return Response('fail')
```

[출처](https://show-me-the-money.tistory.com/entry/Django-Rest-Framework-Serializer%EC%97%90-Model-Instance%EB%A5%BC-%EC%9D%B8%EC%9E%90%EA%B0%92%EC%9C%BC%EB%A1%9C-%EB%B3%B4%EB%82%B4%EA%B8%B0)

## DB 업데이트 하기
기존 `create()` 에서 사용하던 소스에 serializer 를 생성할 때 모델 인스턴스를 추가 하기만 하면 된다

```python
        ...
        model_instance = self.queryset.get(pk=kwargs.get('pk'))
        serializer = self.get_serializer(model_instance, data=request.data)
        ...
```

[출처](https://stackoverflow.com/questions/37021954/django-update-viewset)

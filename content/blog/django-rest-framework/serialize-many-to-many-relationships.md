---
title: "다대다 관계 모델 직렬화(serialize) 하기"
date: "2022-03-31"
category: django-rest-framework
---

## 0. 서론
기본적으로 `manytomanyfield` 를 가진 모델(Package)을 직렬화할 경우 연관된 모델(Product)의 pk 만을 가지고 오게 된다.

기본 serializer 가 반환하는 데이터  
![before-customize](/TIL/images/serialize-many-to-many-relationships/before-customize.png)

나는 Package 를 직렬화 할때 Product 의 pk 이외에 다른 컬럼을 가져와야 했다.  
그리고 package_product 테이블에 생성한 custom field 인 quantity 도 가져와야 했기 때문에 `serializer` 를 커스터마이징 할 필요가 있었다.

내가 원하는 데이터
![after-customize](/TIL/images/serialize-many-to-many-relationships/after-customize.png)

## 1. 새로운 serializer 생성하기
기존에 사용하던 serializer 는 `create()`, `list()`, `update()`, `delete()` 에서 계속 사용하고 `package detail` 을 위해 하나 더 만들기로 했다. 
```
# serializer.py

class PackageDetailSerializer(serializers.ModelSerializer):
    is_split = serializers.SerializerMethodField()
    business_id = serializers.ReadOnlyField(source='business.id')
    business_name = serializers.ReadOnlyField(source='business.name')
    package_product_list = PackageProductSerializer(source='packageproduct_set', many=True)
    package_supplement_list = PackageSupplementSerializer(source='packagesupplement_set', many=True)
    
    class Meta:
        model = Package
        fields = ['id', 'name', 'description', 'serial_code', 'product_code', 'memo', 'is_split', 'business_id', 'business_name', 'package_product_list', 'package_supplement_list']
    
    def get_is_split(self, obj):
        if obj.is_split:
            return '분할함'
        else:
            return '분할하지 않음'
```
serializer 의 field 를 생성할 때 다른 serializer 를 사용했다.  
`PackageProductSerializer` 를 내가 원하는 데이터를 리턴하도록 만들었기 때문에 `PackageDetailSerializer` 에서도 동일하게 원하는 데이터를 확인할 수 있게 되었다.  
[출처1](https://hyun-am-coding.tistory.com/entry/9-1-Serializers)
[출처2](https://www.django-rest-framework.org/api-guide/serializers/#dealing-with-nested-objects)

`source` 인수는 필드를 채우는데 사용할 속성의 이름이라고 한다. 이 인수를 사용하지 않으면 필드이름과 동일한 데이터를 찾게된다.  
예를들어 business_name 필드를 만들때 `source='business.name'` 이라고 지정해주지 않았다면 Package 모델에서 business_name 이라는 필드를 찾게 된다는 말이다.
[출처](https://www.django-rest-framework.org/api-guide/fields/#source)

`_set` 은 Package 모델에서 자신을 foreign key 로 가지고 있는 모델인 PackageProduct 에 접근하기 위한 방법이다.
[출처](https://freeprog.tistory.com/55)

`many=True` 플래그는 단순한 데이터가 아닌 리스트를 반환해야 할 경우 사용한다.



[처음본 곳](https://stackoverflow.com/questions/41976819/django-serialize-a-model-with-a-many-to-many-relationship-with-a-through-argume/41996831#41996831)



## 2. ViewSet 에서 serializer 분리하기
위에서 생성한 serializer 를 `detail()` 메서드에서만 사용해야 했기때문에 메서드 별로 다른 serializer 를 사용하는 방법을 찾아봤다.

`ModelViewSet` 을 상속받는 ViewSet 의 경우 `serializer_class` 속성으로 직렬화에 사용할 클래스를 지정할 수 있다.  
만약 하나의 ViewSet 에서 메서드마다 사용할 직렬화 클래스를 다르게 설정하고 싶다면 다음과 같이 하면 된다

```python
# views.py

class PackageModelViewSet(ModelViewSet):
    queryset = Package.objects.all()
    filter_backends = [DjangoFilterBackend]
    filterset_class = PackageFilter
    
    def get_serializer_class(self):
        if self.action == 'retrieve':
            return PackageDetailSerializer
    
        return PackageSerializer
```

`serializer_class` 속성을 그대로 사용하고 싶었지만 `get_serializer_class` 메서드를 오버라이딩 하면 적용되지 않는듯 했다.

[출처](https://stackoverflow.com/questions/22616973/django-rest-framework-use-different-serializers-in-the-same-modelviewset)

## 99. 끝나고 나서
package_product_list 와 package_supplement_list 의 데이터중 `package detail` 컴포넌트에서 사용하지 않는 데이터가 많다.  
성능 이슈가 발생할 가능성이 있기때문에 이중에서 필요한 데이터만 가져오는 방법을 찾아야할것같다. 
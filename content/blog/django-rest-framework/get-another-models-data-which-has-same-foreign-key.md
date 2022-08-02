---
title: "foreign key 가 동일한 다른 모델의 데이터 가져오기"
date: "2022-03-31"
category: django-rest-framework
---

## 0. 서론
다음과 같은 구조에서 `ProductDetailSerializer` 가 해당 Product 인스턴스와 같은 Business 모델을 Foreign Key 로 가지는 Package 를 가져오고 싶었다.    

```python
# 품목 정의
class Business(models.Model):
    name = models.CharField(max_length=20, unique=True)
    description = models.TextField(null=True, blank=True)
    
    class Meta:
        verbose_name = '품목 정의'
    
    
# 상품 정의
class Product(models.Model):
    name = models.CharField(max_length=64, unique=True)
    description = models.TextField(null=True, blank=True)
    business = models.ForeignKey(Business, on_delete=models.PROTECT)
    maximum_quantity = models.SmallIntegerField()
    
    class Meta:
        verbose_name = '상품 정의'
    
    
# 부자재
...
생략
...
    
    
# 패키지
class Package(models.Model):
    name = models.CharField(max_length=64, unique=True)
    description = models.TextField(null=True, blank=True)
    serial_code = models.CharField(max_length=64, unique=True, null=True, blank=True)
    product_code = models.CharField(max_length=64, null=True, blank=True)
    is_split = models.BooleanField(default=True)
    memo = models.TextField(null=True, blank=True)
    business = models.ForeignKey(Business, on_delete=models.PROTECT)
    product = models.ManyToManyField(Product, through='PackageProduct')
    supplement = models.ManyToManyField(Supplement, through='PackageSupplement')
```

## 1. SerializerMethodField 사용하여 가져오기
처음엔 필터가 적용된 PackageSerializer 를 사용해야 한다고 생각했다.  
그래서 아래처럼 `SerializerMethodField()` 에 필터링된 쿼리셋을 리턴하는 방법으로 구현했다.

```python
class ProductDetailSerializer(serializers.ModelSerializer):
    business_id = serializers.ReadOnlyField(source='business.id')
    business_name = serializers.ReadOnlyField(source='business.name')
    package_product_list = PackageProductSerializer(source='packageproduct_set', many=True)
    package_list = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'description', 'maximum_quantity', 'business_id', 'business_name', 'package_product_list', 'package_list']
    
    def get_package_list(self, product):
        qs = Package.objects.filter(business_id=product.business.id)
        serializer = PackageSerializer(instance=qs, many=True)
        return serializer.data
```

[출처](https://stackoverflow.com/questions/28309507/django-rest-framework-filtering-for-serializer-field)

## 2. SerializerMethodField 사용하지 않기
이후 TIL 에 '다대다 관계 모델 직렬화(serialize) 하기' 라는 글을 쓰기 위해 공부하던 중 더 낫다고 생각되는 방법을 찾아서 수정했다

```python
class ProductDetailSerializer(serializers.ModelSerializer):
    business_id = serializers.ReadOnlyField(source='business.id')
    business_name = serializers.ReadOnlyField(source='business.name')
    package_product_list = PackageProductSerializer(source='packageproduct_set', many=True)
    package_list = PackageSerializer(source='business.package_set', many=True)
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'description', 'maximum_quantity', 'business_id', 'business_name', 'package_product_list', 'package_list']
```

이렇게 하면 기존 방법과 같은 기능을 하지만 더욱 간결하고 가독성도 좋아진 소스라고 생각한다.  

## 99. 끝나고 나서
내가 원했던것이 무엇인지 직관적으로 볼 수 있도록 각 마크다운 파일들을 수정해야 할것같다.  
'대충 이런 형태의 데이터를 원한다' 라는 생각으로는 질문을 하기도 힘들고 성공적으로 구현했다고 하더라도 정리하는데 힘들고 오래걸리기 때문.
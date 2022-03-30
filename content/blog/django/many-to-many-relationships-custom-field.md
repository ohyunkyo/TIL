---
title: "django model 의 다대다 관계에서 필드 추가하기"
date: "2022-03-30"
---

## 0. 서론
기본적으로 다대다 관계에서는 새로운 테이블이 생성된다.   
그리고 column 으로는 관계를 맺게 되는 row 들의 pk 를 저장하게 된다.

아래의 표는 내가 프로젝트를 진행하며 생성된 다대다 관계 테이블이다.  

| id  | package_id | product_id |
|-----|------------|------------|
| 1   | 1          | 1          |
| 2   | 2          | 1          |
| 3   | 2          | 2          |

이 상황에서 나는 다대다 관계 별 수량을 같이 저장해야 할 필요성을 느꼈다. 

## 1. 사용방법
나의 경우에는 package/product, package/supplement 이렇게 두개의 다대다 관계를 생성했다.  
```python
# 패키지 모델

class Package(models.Model):
    name = models.CharField(max_length=64, unique=True)
    description = models.TextField(null=True, blank=True)
    serial_code = models.CharField(max_length=64, unique=True, null=True, blank=True)
    product_code = models.CharField(max_length=64, null=True, blank=True)
    is_split = models.BooleanField(default=True)
    memo = models.TextField(null=True, blank=True)
    business = models.ForeignKey(Business, on_delete=models.PROTECT)
    product = models.ManyToManyField(Product)
    supplement = models.ManyToManyField(Supplement)
    
    class Meta:
        verbose_name = '패키지'
```
마이그레이션이 완료되어 테이블이 생성된 상황에서 다음과 같이 변경했다.
```
# 패키지 모델 변경사항
    ...
    product = models.ManyToManyField(Product, through='PackageProduct')
    supplement = models.ManyToManyField(Supplement, through='PackageSupplement')
    ...

# 패키지_상품
class PackageProduct(models.Model):
    package = models.ForeignKey(Package, on_delete=models.PROTECT)
    product = models.ForeignKey(Product, on_delete=models.PROTECT)
    quantity = models.SmallIntegerField()
    
    class Meta:
        db_table = 'inventory_package_product'


# 패키지_부자재
class PackageSupplement(models.Model):
    package = models.ForeignKey(Package, on_delete=models.PROTECT)
    supplement = models.ForeignKey(Supplement, on_delete=models.PROTECT)
    quantity = models.SmallIntegerField()
    
    class Meta:
        db_table = 'inventory_package_supplement'
```

[출처1](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField.through)
[출처2](https://lee-seul.github.io/django/2019/02/21/django-extend-manytomanyfield.html)
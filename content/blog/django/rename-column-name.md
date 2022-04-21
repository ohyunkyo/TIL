---
title: "django orm 에서 column name 변경하여 리턴하기"
date: "2022-04-21"
---

## 0. 서론
```python
# model.py

# 상품 정의
class Product(models.Model):
  name = models.CharField(max_length=64, unique=True)
  description = models.TextField(null=True, blank=True)
  business = models.ForeignKey(Business, on_delete=models.PROTECT)
  maximum_quantity = models.SmallIntegerField()

  class Meta:
      verbose_name = '상품 정의'


# 부자재
class Supplement(models.Model):
  name = models.CharField(max_length=64, unique=True)
  description = models.TextField(null=True, blank=True)
  business = models.ForeignKey(Business, on_delete=models.PROTECT)
  maximum_quantity = models.SmallIntegerField()

  class Meta:
      verbose_name = '부자재'
```

```python
# model.py

# 상품 재고 처리 기록
class ProductHistory(models.Model):
  business = models.ForeignKey(Business, on_delete=models.PROTECT, blank=True)
  product = models.ForeignKey(Product, on_delete=models.PROTECT, blank=True)
  status = models.CharField(max_length=20, default='pending', choices=common_const.COMMON_STATUS)
  quantity = models.IntegerField()
  inventory_process = models.ForeignKey(InventoryProcess, on_delete=models.PROTECT)
  expiry_at = models.DateField(null=True, blank=True)
  created_at = models.DateTimeField(auto_now_add=True)
  updated_at = models.DateTimeField(auto_now=True)
  reserved_at = models.DateTimeField(null=True, blank=True)
  admin = models.ForeignKey(User, on_delete=models.PROTECT, blank=True)

  class Meta:
      verbose_name = '상품 재고 처리 기록'
      indexes = [
        models.Index(fields=['status', 'expiry_at', 'created_at', 'updated_at', 'reserved_at'], name='product_history_common'),
      ]


# 부자재 재고 처리 기록
class SupplementHistory(models.Model):
  business = models.ForeignKey(Business, on_delete=models.PROTECT, blank=True)
  supplement = models.ForeignKey(Supplement, on_delete=models.PROTECT, blank=True)
  status = models.CharField(max_length=20, default='pending', choices=common_const.COMMON_STATUS)
  quantity = models.IntegerField()
  inventory_process = models.ForeignKey(InventoryProcess, on_delete=models.PROTECT)
  expiry_at = models.DateField(null=True, blank=True)
  created_at = models.DateTimeField(auto_now_add=True)
  updated_at = models.DateTimeField(auto_now=True)
  reserved_at = models.DateTimeField(null=True, blank=True)
  admin = models.ForeignKey(User, on_delete=models.PROTECT, blank=True)

  class Meta:
      verbose_name = '부자재 재고 처리 기록'
      indexes = [
        models.Index(fields=['status', 'expiry_at', 'created_at', 'updated_at', 'reserved_at'], name='supplement_history_common'),
      ]
```

```python
class ProductInventory:

	# noinspection PyMethodMayBeStatic
	def get_list(self, business_id):
		product_history_list = ProductHistory.objects.filter(business_id=business_id, status='complete'). \
			values('product_id', 'expiry_at', 'created_at__date', 'inventory_process__effect').order_by('product_id', 'expiry_at').annotate(quantity=Sum('quantity')). \
			values('product_id', 'product__name', 'expiry_at', 'quantity', 'created_at__date', 'inventory_process__effect')

		# template 에 전달될 상품 재고처리 기록 변수 초기화
		product_inventory_list = defaultdict(dict)

		# 개별 상품의 재고처리 기록을 가공한다
		for product_history in product_history_list:
			# 재고처리 기록의 column 을 변수에 set
			product_id = str(product_history['product_id'])
			name = product_history['product__name']
			expiry_at = str(product_history['expiry_at'])
			quantity = int(product_history['quantity'])
			effect = product_history['inventory_process__effect']

			...

		return dict(product_inventory_list).items()


class SupplementInventory:

	# noinspection PyMethodMayBeStatic
	def get_list(self, business_id):
		supplement_history_list = SupplementHistory.objects.filter(business_id=business_id, status='complete'). \
			values('supplement_id', 'expiry_at', 'created_at__date', 'inventory_process__effect').order_by('supplement_id', 'expiry_at').annotate(quantity=Sum('quantity')). \
			values('supplement_id', 'supplement__name', 'expiry_at', 'quantity', 'created_at__date', 'inventory_process__effect')

		# template 에 전달될 부자재 재고처리 기록 변수 초기화
		supplement_inventory_list = defaultdict(dict)

		# 개별 부자재의 재고처리 기록을 가공한다
		for supplement_history in supplement_history_list:
			# 재고처리 기록의 column 을 변수에 set
			supplement_id = str(supplement_history['supplement_id'])
			name = supplement_history['supplement__name']
			expiry_at = str(supplement_history['expiry_at'])
			quantity = int(supplement_history['quantity'])
			effect = supplement_history['inventory_process__effect']
			
			...
			
        return dict(supplement_inventory_list).items()

```
테이블 명만 다르고 컬럼명은 모두 일치하는 서로 다른 두개의 모델이 같은 과정을 거쳐 같은 형태의 데이터를 리턴하길 원했다.  
처음에는 다른 방법이 생각나지 않아 일단 각 모델별 메서드에서 똑같은 코드를 반복해서 사용했다.  
이후 코드를 재사용할 수 있도록 하나의 메서드에서 컨트롤 가능하도록 만드려고 했는데,  
그러기 위해선 각 쿼리셋이 동일한 key 를 가진 dict 를 반환하도록 하는게 좋을것 같았다.

## 1. annotate 사용하기
django 의 기능인 annotation 을 사용하여 컬럼 이름을 바꾼채로 리턴하도록 할 수 있다.
```python
# 예제

from django.db.models import F

MyModel.objects.annotate(renamed_value=F('cryptic_value_name')).values('renamed_value')
```

```python
# 적용

from django.db.models import F

class InventoryBusiness:
    def __init__(self, business_id):
        self.business_id = business_id

    def get_product_inventory(self):
        queryset = ProductHistory.objects.filter(business_id=self.business_id, status='complete'). \
            values('product_id', 'expiry_at', 'created_at__date', 'inventory_process__effect').order_by('product_id', 'expiry_at').\
            annotate(quantity=Sum('quantity'), model_pk=F('product_id'), model_name=F('product__name')). \
            values('model_pk', 'model_name', 'expiry_at', 'quantity', 'created_at__date', 'inventory_process__effect')
    
        return self.get_inventory(queryset)
    
    def get_supplement_inventory(self):
        queryset = SupplementHistory.objects.filter(business_id=self.business_id, status='complete'). \
            values('supplement_id', 'expiry_at', 'created_at__date', 'inventory_process__effect').order_by('supplement_id', 'expiry_at').\
            annotate(quantity=Sum('quantity'), model_pk=F('supplement_id'), model_name=F('supplement__name')). \
            values('model_pk', 'model_name', 'expiry_at', 'quantity', 'created_at__date', 'inventory_process__effect')
    
        return self.get_inventory(queryset)
        
    def get_inventory(self, queryset):
        for model_history in queryset:
            model_pk = model_history['model_pk']
            ...
```

## 99. 끝나고 나서

### 테이블을 구분한 이유
지금보니 상품, 부자재 테이블을 나눌 필요가 전혀 없었던것 같다.  
하나의 테이블에서 상품인지 부자재인지만 구분하면 됐을텐데 괜히 테이블도 두개로 나누고 재고 테이블도 둘로 분리되어 관리가 어렵게 된것 같아 보인다.  
시간이 된다면 마이그레이션을 진행해서 하나로 통합하고 싶다.

### annotate 사용하기
어노테이션은 새로운 컬럼을 추가하는것과 같다고 한다.   
현재 품목별 재고보기에서 파이썬으로 처리하는 부분을 어노테이션을 사용하여 추가 작업 없이 쿼리만으로 해결 가능할것 같다.

[출처](http://raccoonyy.github.io/django-annotate-and-aggregate-like-as-excel/)
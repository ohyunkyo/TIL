---
title: "django 에서 트랜잭션 적용하기"
date: "2022-04-20"
category: django
---

## 0. 서론 
어플리케이션 외부의 데이터 혹은 업로드 한 엑셀 데이터를 가공하여 발주서 테이블에 등록하는 기능을 만들었다.  
이때 특정 데이터로 패키지를 검색해서 발주서 객체에 추가하는데, 패키지를 검색하지 못해서 예외가 발생할 경우 해당 작업 전체가 취소되어야 한다.

django 에 대해 잘 알지 못했기 때문에 트랜잭션은 나중에 적용하기로 하고 일단 하드코딩으로 기능을 구현했다.
```python
current_time = timezone.now()
created_at = current_time.strftime("%Y%m%d%H%M%S")
        
for loop
    ...
    try:
        package = Package.objects.get(product_code=product_code)
    except Package.DoesNotExist:
        # 패키지가 없을경우 이번에 넣은 데이터 모두 삭제
        purchase_order_list = PurchaseOrder.objects.filter(created_at=current_time, status='pending')
        purchase_order_list.delete()
        
        data = {
            'message': 'Package MultipleObjectsReturned',
            'product_code': product_code
        }

        return JsonResponse(data)
    ...
    purchase_order = PurchaseOrder()
    purchase_order.package = package
    ...
    purchase_order.created_at = current_time
    purchase_order.save()
```
패키지가 존재하지 않을경우 현재 작업에서 지금까지 등록된 데이터를 모두 삭제하고 오류가 발생했다고 알려준다.

## 1. 트랜잭션이란?
트랜잭션은 데이터 조작을 위한 단위라고 한다.  
쉽게 말해서 한개 이상의 쿼리(DML)가 포함된 쿼리 집합이라고 생각하면 될것 같은데 쿼리와 마찬가지로 commit 이나 rollback 할 수 있다.  

영화관처럼 반드시 결제와 예약이 같이 이뤄져야 할 때, 결제후 예약이 누락되는 문제가 발생해선 안되기 때문에
결제와 예약을 하나의 트랜잭션으로 묶고 commit 과 rollback 을 정하도록 한다.

[트랜잭션에 대해서](https://wonit.tistory.com/462)

## 2. django 의 트랜잭션
데이터베이스 설정을 변경하는것으로 하나의 request 전체를 트랜잭션으로 지정 할 수도 있다. 이후 일부 View 에서는 적용되지 않도록 할 수 있다.  
아니면 아예 처음부터 일부 View 에만 적용되도록 할 수도 있다.

## 3. django 의 쿼리 커밋
django 에서는 기본적으로 autocommit 옵션이 true 로 설정되어 자동으로 커밋 된다고 한다.  
'개발자가 따로 커밋하지 않으니 당연히 자동으로 커밋해주겠지' 라고 생각할 수 있지만 이번에 트랜잭션에 대해 공부하기 전에는 커밋에 관해 전혀 생각도 하지 못했다.

## 4. 트랜잭션 적용해보기
발주서 추가 하는 부분에 `transaction.atomic()` 라는 context manager 를 사용하면 기존처럼 데이터를 추가한 뒤 삭제하지 않아도 된다.
```python
class Importer:
	def __init__(self, request):
		self.request = request
		self.created_at = timezone.now()
		self.date_string = self.created_at.strftime("%Y%m%d%H%M%S")

	def find_package(self, **kwargs):
		keyword = None
		mode = kwargs['mode']

		try:
			if mode == 'serial_code':
				keyword = kwargs['serial_code']
				package = Package.objects.get(serial_code=keyword)
			elif mode == 'product_code':
				keyword = kwargs['product_code']
				package = Package.objects.get(product_code=keyword)
			else:
				package = None
		except Package.DoesNotExist:
			raise Exception('등록 되지 않은 패키지 입니다(' + mode+': ' + keyword + ')')
		except Package.MultipleObjectsReturned:
			raise Exception('중복 등록된 패키지 입니다.(' + mode+': ' + keyword + ')')

		return package

	def import_json(self):
		json_data = json.loads(self.request.body)

		try:
			with transaction.atomic():
				for purchase_order_object in json_data['purchase_order']:
					serial_code = purchase_order_object['serial_code']

					package = self.find_package(mode='serial_code', serial_code=serial_code)

					# 모델 객체에 데이터 저장
					purchase_order = PurchaseOrder()
					purchase_order.order_number = dateformat.format(self.created_at, 'Ymd')
					...
					purchase_order.created_at = self.created_at

					# 모델 객체 DB insert
					purchase_order.save()

			data = {
				...
			}

		except Exception as e:
			data = {
				...
			}

		return data
```

[출처1](https://myjorney.tistory.com/142)
[출처2](https://blog.doosikbae.com/146)

## 5. 예외 발생시 처리하기
`try` 블록이 `transaction.atomic()` 블록을 감싸도록 한 후 예외를 처리하면 된다.

[출처](https://lee-seul.github.io/django/2019/02/02/django-transactionmanagementerror.html)
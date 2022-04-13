---
title: "내가 사용해본 django-pandas"
date: "2022-04-13"
---

## pandas 란?
pandas 는 데이터 조작 및 분석을 위해 python 으로 작성된 소프트웨어 라이브러리이다.  
또한 R, SQL, spreadsheet 등 에서 가능하던 작업을 pandas 에서 수행 가능하도록 하는 메서드를 가지고 있다.

[출처](https://pandas.pydata.org/docs/getting_started/comparison/index.html)

## django-pandas 사용하기
이런 pandas 를 django 프로젝트에서 사용 가능하도록 도와주는 도구가 django-pandas 이다.  

### 설치하기
django-pandas 를 사용하기 전에 pandas 를 설치해야 한다
```shell
pip install numpy
pip install pandas
```
이제 django-pandas 를 설치한다.
```shell
pip install django-pandas
```

### queryset 으로 dataframe 생성하기
`read_frame()` 메서드를 사용하면 간단하게 생성할 수 있다.
```python
from django_pandas.io import read_frame
qs = InventoryProcess.objects.all()
df = read_frame(qs)

print(df)
```

```shell
# 결과 

   id     name  effect
0   3  초기재고    True
1   4    반품    True
2   5    출고   False
3   6    입고    True
```

이제 이 데이터 프레임의 양식을 변경해보자

### column 순서 변경하기
dataframe 에 원하는 순서대로 컬럼 명을 나열한 리스트를 전달하면 해당 순서대로 컬럼의 위치가 변경된다. 
```python
from django_pandas.io import read_frame
qs = InventoryProcess.objects.all()
df = read_frame(qs)

df = df[['name', 'effect', 'id']] # 추가한 코드

print(df)
```
```shell
# 결과 

     name  effect  id
0  초기재고    True   3
1    반품    True   4
2    출고   False   5
3    입고    True   6
```

### 컬럼 명 변경하기 - 전체
`columns` 속성을 사용하여 사용하여 dataframe 의 컬럼 이름을 변경한다.  
아래의 방법으로는 모든 컬럼의 이름을 변경하기 때문에 실제 컬럼 갯수와 리스트의 요소 갯수가 일치해야 한다.
```python
from django_pandas.io import read_frame
qs = InventoryProcess.objects.all()
df = read_frame(qs)

df = df[['name', 'effect', 'id']]
df.columns = ['이름', '처리방법', 'id'] # 추가한 코드

print(df)
```

```shell
# 결과 

     이름   처리방법  id
0  초기재고   True   3
1    반품   True   4
2    출고  False   5
3    입고   True   6
```

### 컬럼명 변경하기 - 개별
전체 컬럼명이 아닌 일부만 변경하고 싶을 경우 `rename()` 메서드에 기존컬럼 이름과 변경할 컬럼 이름을 전달한다.
```python
from django_pandas.io import read_frame
qs = InventoryProcess.objects.all()
df = read_frame(qs)

df = df[['name', 'effect', 'id']]
df.columns = ['이름', '처리방법', 'id']
df.rename(columns={'id': 'pk'}, inplace=True) # 추가한 코드

print(df)
```

```shell
# 결과 

     이름   처리방법  pk
0  초기재고   True   3
1    반품   True   4
2    출고  False   5
3    입고   True   6
```

### 인덱스 제거하여 출력하기
`to_string()` 메서드의 인수에 `index=False` 를 전달하여 인덱스가 표시되지 않도록 한다. 
```python
from django_pandas.io import read_frame
qs = InventoryProcess.objects.all()
df = read_frame(qs)

df = df[['name', 'effect', 'id']]
df.columns = ['이름', '처리방법', 'id']
df.rename(columns={'id': 'pk'}, inplace=True) 

print(df.to_string(index=False)) # 변경한 코드
```

```shell
# 결과 

이름  처리방법  pk
초기재고  True   3
반품  True   4
출고 False   5
입고  True   6
```

[출처1](https://seong6496.tistory.com/133)
[출처2](https://hogni.tistory.com/51)
[django-pandas github](https://github.com/chrisdev/django-pandas)

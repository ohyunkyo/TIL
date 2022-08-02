---
title: "프로젝트 내부의 API를 호출하면 안되는 이유"
date: "2022-03-14"
category: django
---

## 0. 서론
월별 재고 확인을 위해 각 상품의 재고를 가져와 월별로 출력하는 기능이 필요했다.  
이 월별 재고를 다른곳에서도 확인 가능할 수 있도록 drf 를 사용해 RESTful API 로 데이터를 제공하기로 결정했다.

## 1. REST API 제작
`year/month/product_id` 형태의 url 로 요청하면 해당 상품의 월별 최종 재고 데이터를 `JsonResponse` 로 반환 하도록 제작했다.

## 2. 월별 재고 페이지 제작
앞서 제작한 API 를 호출하여 특정연도에 상품의 월별 재고를 출력하는 페이지를 제작하려고 했다.  
이때 해당 API 의 endpoint 로 요청하여 데이터를 받는 방법과 API View 를 호출하는 방식을 생각했었는데 첫번째 방법은 문제가 있을것 같아서 두번째 방법으로 해결했다.

## 3. 어떤 문제가 있을까?
같은 앱에 있는 API endpoint 로 요청하는것은 좋은 예제라고 볼 수 없다고 한다. [(출처1)](https://stackoverflow.com/questions/60529072/django-correct-method-of-consuming-my-own-rest-api-internally-in-the-views-py)    
최대한 이런 방법을 피하고 해당 클래스에 직접 액세스해야 한다고 하는데 아마도 속도 문제인것 같다.  [(출처2)](https://www.reddit.com/r/django/comments/7sxiqn/internal_api_requests_does_it_make_sense_for_a/)

기본적으로 이 방법은 nginx 와 uwsgi 를 통해 새로운 HTTP 요청을 전달하기 때문에 더 많은 시간이 소요된다고 한다.  
더 많은 시간이 소요 된다는것은 더 많은 자원을 사용 한다는뜻이고, 서버와 어플리케이션 제공자에게 부담이 될 수밖에 없다.

## 99. 끝나고 나서
내가 원하던 형태로 작동한다. 그러나 이것이 정답인것 같지는 않다.  
[지금 사용한 방법](https://github.com/ohyunkyo/inventory-manage/commit/f2b6c40806074f10650c0e6e64b4b806bc538b91#r68620070) 보다 더 좋은 방법이 있을것같다는 생각이 든다.

[drf 구글 그룹](https://groups.google.com/g/django-rest-framework/c/26tIiJB7vQw) 에서는 믹스인 사용을 권장한다.  
기능 개발이 모두 끝나면 코드 리팩토링할 예정인데, 믹스인에 대해 알아보고 필요한 부분에 적용해야겠다. 
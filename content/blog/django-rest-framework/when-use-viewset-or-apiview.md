---
title: "ViewSet 과 APIView 를 사용해야 할 때"
date: "2022-04-05"
---

## ViewSet 을 사용해야 할 때
API가 기본 CRUD 작업을 수행할 일이 많을 경우.
### 장점
모든 코드를 쓸 필요 없이 적은 양의 코드로도 CRUD 를 구현할 수 있다.

## APIView 를 사용해야 할 때
많은 커스텀 작업이 필요한 경우.
### 장점
익숙하지 않은 사람들이 코드를 이해하기 쉽다.

[출처](https://www.reddit.com/r/django/comments/sm07s2/drf_when_to_use_viewsets_vs_generic_views_vs/)
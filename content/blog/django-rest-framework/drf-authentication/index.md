---
title: "drf 의 인증"
date: "2022-07-29"
last_modified_at: "2022-07-00"
---

## 0. 서론
drf 로 구현한 API 서버에서 사용자 인증을 고민하던중 여러 방법이 있다는것을 알게되었다.  
그 방법들중 일부를 간단히 정리하고 내가 사용할 것을 정했다.

## 1. drf Authentication
인증이란 `요청`과 `자격 증명`을 연결하는 방법이다.  
인증은 항상, 모든 경우에서 가장 먼저 실행된다. 이후 권한 및 제한 정책으로 해당 `자격 증명의 요청` 을 허용해야 하는지 여부를 결정하게 되며, 인증 자체가 들어오는 요청을 허용하거나 허용하지 않는 작업을 하지는 않는다. 

drf 는 이런 인증 체계를 바로 사용할 수 있도록 제공한다.

## 2. drf 에서 사용 가능한 토큰 인증
여러 인증 방법중 토큰을 사용한 인증 방법에 대해 정리한다.

## 2.1 drf TokenAuthentication
drf 의 기본 토큰 인증이다. 다음과 같은 특징을 가진다.
- 하나의 토큰으로 모든 세션을 처리한다.
- 만료 기한이 없다.
- 클라이언트-서버 형태에 적합하다.
- 헤더에 토큰이 담겨 올 경우 해당 토큰을 DB에서 검색한다.

## 2.2 JWT
- 토큰 자체에 사용자의 정보가 포함되어있다.
- 일정 기간이 지나면 만료된다.
- RESTful 한 환경에서 사용자 데이터를 주고받을 수 있다.

## 3. 선택
실 사용경험이 없어서 인지 이런 장단점이 와닿진 않았고, 오히려 요즘 많이 쓴다는 JWT 를 써야겠다는 생각이 들었다.  
그러나 써드파티 패키지가 아닌 것을 먼저 사용해보는게 좋겠다는 생각으로 `drf TokenAuthentication` 를 사용하기로 정했다.

## References
[1. Authentication](https://www.django-rest-framework.org/api-guide/authentication/#authentication)  
[2. stackoverflow](https://stackoverflow.com/questions/31600497/django-drf-token-based-authentication-vs-json-web-token)  
[2. 블로그](https://medium.com/chanjongs-programming-diary/django-rest-framework-drf-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-jwt-%EA%B8%B0%EB%B0%98-authentication-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0-with-simplejwt-%EC%B4%88%EA%B8%B0-%ED%99%98%EA%B2%BD-%EC%84%B8%ED%8C%85-1-e54c3ed2420c)  
[2.1 drf TokenAuthentication 1](https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication)    
[2.1 drf TokenAuthentication 2](http://recordingbetter.com/drf/2017/08/02/Session-%EC%9D%B8%EC%A6%9D%EA%B3%BC-Token-%EC%9D%B8%EC%A6%9D)  
[2.2 JWT](https://pronist.dev/143)  
[3. 선택 1](https://yceffort.kr/2021/05/drawback-of-jwt)  
[3. 선택 2](https://velog.io/@wildmental/Django-%EC%A0%91%EC%86%8D-%EC%9D%B8%EC%A6%9D-Built-in-Token-VS-JWTJSON-Web-Token-%EB%B9%84%EA%B5%90-%EB%81%9D)
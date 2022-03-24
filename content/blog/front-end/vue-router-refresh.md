---
title: "Vue Router 를 사용하여 페이지 새로고침 하기"
date: "2022-03-24"
---

## 0. 서론
모달에서 `DELETE` 요청이 완료되면 `inventory process list` 라우트로 이동하도록 하고싶었다.  
그런데 이동하고자 하는 라우트와 현재 라우트가 동일했기 때문에 에러가 발생했다.
```vue
# 기존 소스

this.$router.push({name: 'inventory process list'})
```

## 1. 해결 방법
동일한 라우트로 이동하는것이 문제라면 페이지를 새로고침 하면 되겠다고 생각했다.  
그러나 Vue Router 에는 `reload()` 메소드가 없다는것 같아서 다른 방법을 찾아봤다.
```vue
# 변경한 소스

this.$router.go(this.$router.currentRoute);
```
`go()` 메소드를 사용하여 현재 라우트로 이동하도록 변경했더니 해결되었다.

## 2. Vue Router 의 메소드
`push()` 와 `go()` 메소드에는 어떤 차이가 있길래 같은 라우트를 인자로 전달했음에도 다른 결과를 보여주는걸까?

처음 아래의 블로그를 봤을땐 `this.$router.currentRoute` 가 정수를 리턴할것이라고 생각했지만 실제로 콘솔창에 출력해본 결과, 현재 라우트에 대한 객체가 출력되었다.

내 생각과 좀 다른걸로 봐선 Vue Router 의 메소드가 작동하는 원리는 Vue Router 에 대해서 따로 공부하면서 알아봐야 할것같다.

[출처](https://sunny921.github.io/posts/vuejs-router-03/)
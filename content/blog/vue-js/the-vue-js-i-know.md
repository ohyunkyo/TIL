---
title: "내가 이해한 vue.js"
date: "2022-03-18"
---

## vue.js 를 선택한 이유
지금 내가 프론트엔드 프레임워크를 사용하려면 비교적 배우기 쉽고 복잡하지 않은 것을 선택하여 시작 하는것이 유리하다고 생각했다.  
vue.js 는 [작은 프로젝트를 빠르게 만들기 쉽고, 자바스크립트 문법에 대해 잘 몰라도 된단다.](https://velog.io/@leehaeun0/React-vs-Vue-%EC%9E%A5%EB%8B%A8%EC%A0%90-%EB%B9%84%EA%B5%90)

## vue.js 란 무엇일까
이름에서 보이는것처럼 js 기반의 웹 프레임워크이다.  
다른 프레임워크와는 다르게 html 에 vue 문법을 조합하여 사용하는 방식이다.  

## 이걸 왜 쓰나요?
간단히 말하면 이게 더 편리하다는거다.
### Virtual DOM
DOM 이라는건 HTML 문서를 트리 구조로 표현한것 이라고 생각하면 된다.  
이 DOM 을 통해 HTML 과 CSS 를 조작할 수 있는것이다.  

하나의 페이지만 봐도 수많은 HTML 태그가 있고 그만큼 DOM 요소들도 많다.  
그런데 만약 사용자 요청에 따라 HTML 을 변경해야 할 일이 많다면 관리 해야 할 DOM 요소들이 많아지고,
그만큼 개발자가 신경써야 할 사항도 점점 많아지게 된다.

이때 기존 방식보다 더 단순하고 편리하게 관리 가능한 V-DOM 을 사용 한다는것이다.
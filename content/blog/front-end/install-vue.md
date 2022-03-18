---
title: "vue.js 설치해보기"
date: "2022-03-17"
---

## 0. 서론
앞으로 개발할 예정인 재고관리 API 서버와 연동하기 위해 프론트엔드 프레임워크를 사용하여 간단히 화면을 구성해보기로 했다.

## 1. 설치
일단 vue.js 를 먼저 설치 한 뒤 프로젝트를 생성해줬다.
```shell
npm install -g @vue/cli
vue create project_name
```

## 2. 실행
서버를 실행시킨다.
```shell
npm run serve
```
> 내부망 IP 로도 접근 가능하도록 설정된다. django 에서 `runserver` 를 실행했을땐 안되던것이다.
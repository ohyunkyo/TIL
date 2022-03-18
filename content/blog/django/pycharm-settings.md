---
title: "pycharm 에서 django 세팅하기"
date: "2022-03-18"
---

## 개요
pycharm 에서 run configurations 을 통해 테스트 서버를 실행하는 방법 정리

## 프로젝트 파이썬 인터프리터 설정
PyCharm - Preferences - Project: {project_name} - Python Interpreter 항목에서 적절한 인터프리터 버전을 선택한다

## 실행 환경 설정
Run - Edit Configurations 에서 새로운 실행 환경을 생성한다.  
settings 를 분리했다면 Environment variables 옵션에 DJANGO_SETTINGS_MODULE 을 추가한다.  
Python Interpreter 옵션도 적절히 선택한다.
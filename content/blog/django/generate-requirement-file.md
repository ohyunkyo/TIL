---
title: "requirements 파일 생성하기"
date: "2022-03-17"
category: django
---

## 역할
현재 프로젝트에서 필요로 하는 파이썬 패키지들 목록과 버전을 정의한 파일.  
새로운 환경에서 해당 파일을 사용하여 한번에 패키지들을 설치할 수 있다.

## 파일 생성 방법
`$ pip freeze > requirements.txt`

## 파일 사용 방법
`$ pip install -r requirements.txt`

## 유의사항
보통 프로젝트 디렉토리 내부에 생성하는듯 하다.
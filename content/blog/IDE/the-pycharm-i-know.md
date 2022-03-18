---
title: "내가 사용하는 pycharm"
date: "2022-03-18"
---

## Run Django Server
pycharm 에서 run configurations 을 통해 테스트 서버를 실행하는 방법 정리
### 프로젝트 파이썬 인터프리터 설정
PyCharm - Preferences - Project: {project_name} - Python Interpreter 항목에서 적절한 인터프리터 버전을 선택한다
### 실행 환경 설정
Run - Edit Configurations 에서 새로운 실행 환경을 생성한다.  
settings 를 분리했다면 Environment variables 옵션에 DJANGO_SETTINGS_MODULE 을 추가한다.  
Python Interpreter 옵션도 적절히 선택한다.

## Bookmark
- F3 = 해당 라인 북마크 지정
- ⌥ + F3 = 해당 라인 기억북마크(Mnemonic Bookmark) 지정
- 북마크, 기억북마크 라인에서 + F3 = 지정 해제
- ⌃ + 숫자 = 기억북마크에 저장된 숫자로 이동
- ⌘ + F3 = 전체 북마크 목록을 본다
- 전체 북마크 목록 + 알파벳 = 알파벳에 설정된 기억북마크로 이동

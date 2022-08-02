---
title: "깃 커밋 리셋하기"
date: "2022-07-29"
last_modified_at: "2022-07-00"
category: git
---

## 0. 서론
커밋할때 실수로 포함하지 말아야 할 파일을 포함했다. 해당 커밋에서 이 파일을 제거할 방법을 찾아보다가 커밋 전으로되돌리는 방법을 알게되어 정리한다.

## 1. git reset
```shell
$ git reset --soft HEAD~1
```

위의 명령어는 최근의 커밋 1개를 취소하고 해당 커밋의 수정 내역(`changes`)을 다시 `staged` 상태로 만들어준다

## 99. git fork 에서 커맨드 라인 사용법
fork 에는 soft reset 기능이 없어서 아래 버튼을 눌러 콘솔창을 띄운 뒤 명령어를 입력했다.

![console-button](./999-console-button.png)

## References
[마지막 커밋 되돌리기](https://codechacha.com/ko/git-undo-last-commit/)
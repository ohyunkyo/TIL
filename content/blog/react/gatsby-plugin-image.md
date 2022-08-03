---
title: "개츠비 이미지 플러그인 적용"
date: "2022-05-06"
last_modified_at: "2022-08-02"
category: react
---

## 0. 서론
`gatsby-plugin-image` 을 사용중임에도 불구하고 이미지 리사이즈, 블러 등의 기능이 작동하지 않아 플러그인 사용법에 대해 알아보게 되었다.

## 1. 해결 방법
이미지 확장자를 대문자에서 소문자로 변경하면 된다. `PNG` 를 사용해도 이미지는 출력 되니 확장자의 문제라고는 전혀 생각도 못했지만 어쨌든 `png` 로 변경하니 잘된다.

## References
https://github.com/gatsbyjs/gatsby/issues/6698#issuecomment-986158429

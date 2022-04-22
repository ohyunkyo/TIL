---
title: "env 파일 사용하기"
date: "2022-04-22"
---

## 0. 서론
기존에 API 엔드포인트의 주소를 각 컴포넌트(vue파일)에 명시했었는데, axios 를 모듈화 하기위해선 해당 URL 을 개별적으로 관리할 필요가 있다고 느꼈다.  

## 1. .env 파일이란?
vue 의 환경변수 파일은 여러개로 나뉠 수 있다.  

개발환경(`npm run serve`)에서는 `.env.development` 파일을 자동으로 인식한다.  
그렇기 때문에 개발환경에서 필요한 변수를 선언해주면 된다.  
또한 `.gitignore` 파일에 등록하여 깃에 같이 업로드 되지 않도록 해야한다.

`.env.production` 파일은 배포버전에서 자동으로 인식한다.(어떻게 인식하는지 잘 모르겠음)  
실제 운영시 필요한 변수를 선언 하면 될것같다.

그리고 그냥 `.env` 파일은 `.env.development` 과 `.env.production` 모두에서 인식하기 때문에 공통적으로 필요한 변수를 선언하면 된다.

[출처](https://hjcode.tistory.com/96)

## 2. .env 파일 생성하기
위의 환경변수 파일들은 모두 루트 디렉토리에 생성해야 한다.  
`.gitignore` 파일이나 `package.json` 파일이 있는 그 위치에.

[출처](https://crispypotato.tistory.com/52)

## 3. 변수 선언하기
다음과 같이 `VUE_APP_` 라는 접두어가 붙은 변수들은 자동으로 로딩된다.  
vue2 에선 따로 로딩해줬어야 한다고 한다.
```text
VUE_APP_NUM=10
VUE_APP_STR=hi
```
나는 다음과 같이 개발서버 URL 을 설정했다.

```text
# .env.development

VUE_APP_API_URL = http://localhost:8000/
```

[출처1](https://kang-ji.tistory.com/241)
[출처2](https://joshua1988.github.io/vue-camp/deploy/env-setup.html#env-%E1%84%91%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%AF)

## 4. 환경변수 사용하기
다음과 같은 방법으로 사용할 수 있다.  
위에서도 다뤘듯 별도의 로딩 과정 없이 바로 사용할 수 있다.
```javascript
# src/api/index.js

import axios from "axios";

function createInstance(){
    return axios.create({
        baseURL: process.env.VUE_APP_API_URL,
    });
}

export const instance = createInstance();
```
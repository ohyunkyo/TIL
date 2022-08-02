---
title: "vue.js 기초 사용법"
date: "2022-03-21"
---

## 개요
백엔드 개발자로서 구매한 템플릿을 변경하는데 필요한 부분만 간단히 정리.

## 라우팅
특정 URL 에 특정 컴포넌트를 라우팅 시킨다.  
URL 의 경로와 이름을 지정한다.  

## 백엔드와 통신하기
HTTP 통신 라이브러리인 axios 를 사용한다.  
백엔드 API 엔드포인트로 요청한 뒤 결과 데이터를 사용 할 수 있다.  

## 받아온 데이터 페이지에 전달하기
url 의 데이터를 받아 business_list 에 전달했다.  
data 에서 return 하는것은 vue 문법인것같다.
```vue
<script>
    import axios from "axios";
    let url = "http://localhost:8000/api/business";
    export default {
      data () {
        return {
          business_list: []
        }
      },
      mounted() {
        axios({
          method: "GET",
          url: url
        })
        .then(response => {
            this.business_list = response.data;
          })
          .catch(response => {
            console.log("실패!!", response);
          });
    
      }
    }
</script>
```

## 유효성 검사하기
validation 모듈을 사용하여 유효성검사가 가능하다.  
유효성 검사를 통과했을 때 데이터를 전송한다.  
front 의 유효성 검사이기 때문에 무조건 믿지 말고 back 에서도 해야할것 같다.  

## 태그에서 변수 사용하기
다음 예제처럼 `title` 이 아닌 `:title` 을 사용하면 value 에 변수를 사용할 수 있다.
```vue
<base-page-heading :title="name">
</base-page-heading>
```

## 특정 라우터로 이동 가능한 버튼
`router-link` 태그를 사용하면 특정 라우터에 인자를 전달하여 이동시킬 수 있다.  
공식 문서를 참고했는데도 warning 이 발생하여 더 알아봐야 할것같다.
```vue
<b-button variant="info" size="lg" router-link :to="{name: 'business update', params: {id: id}}">수정하기</b-button>
```

## 수정시 form 에 기존 데이터 넣기
```vue
<script>
    ...
    mounted() {
    axios({
    method: "GET",
    url: url+this.$route.params.id
    })
    .then(response => {
    console.log(response.data);
    this.form.id = response.data.id;
    this.form.name = response.data.name;
    this.form.description = response.data.description;
    })
    .catch(response => {
    console.log("실패!!", response);
    });
    },
    ...
</script>
```

## PATCH 요청시 유의할점
URL 의 가장 뒤에 `/` 를 넣어줘야한다. 모든 요청시 이런식으로 해야하는지는 잘 모르겠다.
```vue
<script>
    ...
    url: url+this.$route.params.id+"/",
    ...
</script>
```
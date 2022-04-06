---
title: "Vue Router 사용한 페이지 이동"
date: "2022-04-06"
---

## 0. 서론
API 에서 받아온 날짜 형식을 변경해서 사용하고 싶었다.  
그리고 그 변경된 형식을 `router-link` 태그의 `params` 속성에 사용하여 페이지 이동하는데 사용할 생각이였다.

```
Y:M:D H-m-s -> YMDHms
```

## 1. router-link 로 페이지 이동하기
vue 에 대해서 자세히 공부하고 시작한것이 아니라 템플릿을 구매하고 필요한부분만 찾아서 사용했기 때문에 이 방식밖에 알지 못했다.  
그래서 filter 기능을 params 에 적용하면 원하는 기능을 구현할 수 있다고 생각했다.  
그러나 아래와 같은 형식으로는 에러가 출력됐다. 해당 위치에서 filter 를 사용하기 위해 다른 형식으로도 시도해봤지만 결국 되지 않았다.
```vue
...
<router-link :to="{name: 'purchase order queue', params: {created_at: purchase_order.created_at | dateToString }}">{{ purchase_order.created_at }}</router-link>
...
```

## 2. 메서드를 사용하여 페이지 이동하기
`router-link` 를 사용하여 문제를 해결하기위해 구글링 하던 중 다른 방법으로도 페이지를 이동할 수 있다는것을 알게 되었다.  
vue 컴포넌트 내에 메서드를 생성하고 클릭시 메서드가 실행되도록 하는 방식이다.  
이런 방식을 사용하면 코드가 조금 복잡해지지만 커스텀 할 수 있다는 장점이 있다.

```vue
...
<b-link @click="purchaseOrderQueue(purchase_order.created_at)">{{ purchase_order.created_at }}</b-link>
...
```

```vue
...
methods: {
    purchaseOrderQueue(dateValue) {
        let dateFormat = "YYYYMMDDHHmmss"
        let dataString = moment(new Date(dateValue)).format(dateFormat)
        this.$router.push({
            name: "purchase order queue",
            params: { created_at: dataString}
        })
    }
}
...
```
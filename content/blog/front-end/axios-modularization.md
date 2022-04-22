---
title: "axios 모듈화 하기"
date: "2022-04-22"
---

## 0. 서론
코드의 재사용성을 높이기 위해 현재 뷰 컴포넌트에서 각자 axios 객체를 생성하고 요청하던것을 모듈화 하기로 했다.,

## 1. 모듈화의 장점
쉽고 효율적이다.

간단하면서도 강력한 장점이다. 사용할 API 가 많아지면 많아질수록 더욱 효율적으로 사용 가능하고 관리하기도 쉽다.

## 2. 모듈화 하기
우선 axios 인스턴스를 생성하여 리턴하는 파일을 만들어준다.  
`.env` 파일에 정의된 URL 을 사용한다.
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

이번엔 인스턴스를 활용해 실제로 요청을 보내는 함수를 생성한다.  
위에서 만든 인스턴스를 `import` 하고 `get` 메서드를 사용해 요청을 보낸다.
```javascript
# src/api/inventory.js

import {instance} from './index'

// 품목별 재고 가져오기
function fetchInventoryBusiness(business_id) {
    return instance.get(
        'api/inventory/business/' + business_id
    )
}

// 월별 재고 가져오기
function fetchInventoryMonthly(product_id) {
    return instance.get(
        'api/inventory/monthly/',
        {
            params: {
                mode: 'monthly',
                model: 'product',
                year: 2022,
                model_pk: product_id
            }
        }
    )
}

export {
    fetchInventoryBusiness,
    fetchInventoryMonthly
}
```

실제로 뷰 컴포넌트에서 사용하는 부분이다.  
`then, catch` 를 `inventory.js` 에서 해줘도 되겠지만 각 컴포넌트에서 정의해주는 편이 더 활용하기 편하다고 생각했다.
```vue
# src/views/business/Business.vue

...
<script>
import {getInventoryBusiness} from '/src/api/inventory'

export default {
  data () {
    return {
      inventory_business: []
    }
  },
  mounted() {
    const result = getInventoryBusiness(this.$route.query.pk)
    result.then(response => {
      this.inventory_business = response.data
    }).catch(err => {
      console.log("에러 발생", err)
    });
  }
}
</script>
...

```

[출처1](https://heewon26.tistory.com/158)
[출처2](https://yamoo9.github.io/axios/guide/)
[출처3](https://greedysiru.tistory.com/804)
[출처4](https://dev-jsk.tistory.com/109)
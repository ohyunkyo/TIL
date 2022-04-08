---
title: "axios 에서 form 전송하기"
date: "2022-04-08"
---

## 0. 서론
validation 을 사용하지 않는 form 은 처음이였는데 데이터가 제대로 전송되지 않아서 찾아봤다.

```javascript
data (){
    return {
    form: {
        open_market: null,
        purchase_order_file: null
    },
    
    ...
    
methods: {
    onSubmit() {
        axios({
            method: "POST",
            url: url,
            data: this.form
        })
    }
}

...
```

## 1. 문제점
기본적으로 axios 에서 데이터를 전송할 때에는 json 형태의 데이터가 전송된다.  
그렇기 때문에 form 을 사용해야 하는 경우(file 전송 등)에는 별도의 설정이 필요하다.

## 2. 해결방법
input 의 데이터를 FormData 형식으로 변경 후 전송한다.
```javascript
...
onSubmit(){
    let form = new FormData()
    form.append('open_market', this.form.open_market)
    form.append('purchase_order_file', this.form.purchase_order_file)
    axios({
        method: "POST",
        url: url,
        data: form,
        headers: {
            'Content-Type': 'multipart/form-data'
        }
    })
    .then(response => {
        console.log(response.data)
        this.$router.push({name: 'purchase order queue', params: {date_string: response.data.result_data}})
    })
    .catch(error => {
        alert('등록에 실패했습니다')
        console.log(error.response);
    });
}
...
```

[출처1](https://yamoo9.github.io/axios/guide/form-format.html#%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80)
[출처2](https://solbel.tistory.com/886)


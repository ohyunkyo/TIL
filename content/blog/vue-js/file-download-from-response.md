---
title: "http response 의 파일을 다운로드 하기"
date: "2022-04-27"
---

## 0. 서론
엑셀 파일이 포함된 HTTP Response 를 리턴하도록 만든 뷰가 있다.  
브라우저로 API 엔드포인트에 접속하여 테스트 했을때 엑셀 파일이 정상적으로 다운로드 되었다.  
이제 프론트엔드에서 이 엔드포인트로 요청을 전달하여 엑셀 파일을 받아오기만 하면 된다.  
그러나 응답 결과는 엑셀 파일이 아니라 JSON 이였다.

원인 파악을 위해 여러 자료를 찾아보다 주변의 도움을 받고 문제를 해결했다.

## 1. 기존 소스코드
엑셀이 포함된 HTTP Response 를 리턴하는 뷰
```python
# purchase_order_api.view

class PurchaseOrderDownloadAPI(APIView):
    def get(self, request, *args, **kwargs):
        ...
        response = HttpResponse(headers={
            'Content-Type': 'application/vnd.ms-excel',
            'Content-Disposition': 'attachment; filename*=UTF-8\'\'purchase_order_' + date_string + '.xls'
        })

        df.to_excel(response, index=False, engine='xlwt')

        return response
```

해당 뷰의 엔드포인트에 요청 후 응답을 받아 리턴하는 소스
```javascript
# purchase_order.js

function downloadPurchaseOrder(dateString) {
    return instance.get(
        'api/purchase_order_download/' + dateString + '/'
    ).then(response => {
        console.log(response)
        alert('발주서 다운로드가 완료되었습니다.')
        return response.data
    }).catch(err => {
        console.log("에러 발생", err)
    });
}
```
## 2. 해결하기
아래의 소스를 받고 나의 프로젝트에 적용하여 일단 기능을 구현했다.
```javascript
# purchase_order.js

function downloadPurchaseOrder(dateString) {
    return instance({
        url: 'api/purchase_order_download/' + dateString + '/',
        method: 'get',
        responseType: 'blob'
    }).then(response => {
        const url = window.URL.createObjectURL(response.data);
        const link = document.createElement("a");
        link.href = url;
        link.setAttribute(
            "download",
            '발주서_'+dateString
        );
        document.body.appendChild(link);
        link.click();
        link.remove();

        console.log(response)
        alert('발주서 다운로드가 완료되었습니다.')

        this.$router.go(this.$router.currentRoute)
    }).catch(err => {
        console.log("에러 발생", err)
    });
}
```

## 3. 항목별 설명

### 3.1 responseType 설정
axios 응답으로 받아오는 데이터 형식의 default 값은 `json` 이다.  
그러나 자바스크립트에서 텍스트가 아닌 대용량 데이터를 주고받을때에는 `blob` 이라는 객체를 사용해야 한다고 한다.  
그렇기 때문에 일단 응답 형식을 `blob` 으로 설정한다.

[출처](https://sambalim.tistory.com/94)

### 3.2 blob 객체의 엑셀 데이터를 다운로드 하기
axios 가 받아온 `blob` 형태의 response 에는 http header, footer 등 필요하지 않는 데이터가 있기 때문에 바이너리 데이터만을 사용한다.

이 바이너리 데이터로 URL 을 만들고 브라우저의 클릭 이벤트를 발생시켜 해당 URL 을 클릭하게 되면 파일이 다운로드 된다.  
이렇게 하지 않으면 보안상 이슈로 브라우저가 다운로드를 막는다고 한다.
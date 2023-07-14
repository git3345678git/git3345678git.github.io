---
layout: post
title: "Blind SQL injection with time delays"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# Blind SQL injection with time delays

```
This lab contains a [blind SQL injection](https://portswigger.net/web-security/sql-injection/blind) vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

To solve the lab, exploit the [SQL injection](https://portswigger.net/web-security/sql-injection) vulnerability to cause a 10 second delay.

#### Hint

You can find some useful payloads on our [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).




本實驗包含一個盲 SQL 注入漏洞。 該應用程序使用跟踪 cookie 進行分析，並執行包含提交的 cookie 值的 SQL 查詢。

不會返回 SQL 查詢的結果，並且應用程序不會根據查詢是否返回任何行或導致錯誤而做出任何不同的響應。 但是，由於查詢是同步執行的，因此可以觸發條件時間延遲來推斷信息。

解決實驗室，利用SQL注入漏洞造成10秒延遲。
暗示

您可以在我們的 SQL 注入備忘單上找到一些有用的有效載荷。



```


cheatsheet:
https://portswigger.net/web-security/sql-injection/cheat-sheet


PostgreSQL:

pg_sleep(10) 睡10秒
```
'||pg_sleep(10)--
```




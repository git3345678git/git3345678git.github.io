---
layout: post
title: "Authentication bypass via information disclosure"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Information_Disclosure
author: ""
tags: ['PortSwigger_Lab', 'Information_Disclosure']
---





# Authentication bypass via information disclosure
通過信息洩露繞過身份驗證


This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.

To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete Carlos's account.

You can log in to your own account using the following credentials: `wiener:peter`



該實驗室的管理界面有一個身份驗證繞過漏洞，但在不了解前端使用的自定義 HTTP 標頭的情況下利用它是不切實際的。

解決lab，獲取header name，然後用它繞過lab的認證。 訪問管理界面並刪除 Carlos 的帳戶。

您可以使用以下憑據登錄到您自己的帳戶：`wiener:peter`



解說:
https://blog.csdn.net/ZripenYe/article/details/120793759#:~:text=%E5%8F%91%E5%B8%83-,Lab%3A%20Authentication%20bypass%20via%20information%20disclosure%EF%BC%9A%E9%80%9A%E8%BF%87,%E4%BF%A1%E6%81%AF%E6%B3%84%E9%9C%B2%E7%BB%95%E8%BF%87%E8%AE%A4%E8%AF%81

trace http 
https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/TRACE




這題不會解，查網路來的。關鍵在於 
trace 的請求，可以拿到自定義標頭檔。

```
https://0a9a00dd040f4c65c0eb846800ad0045.web-security-academy.net/admin



response:
Admin interface only available to local users



```

關鍵在於判端是否為local使用者。

這裡可以用ip判斷:

而用trace 返回的:
X-Custom-IP-Authorization: 52.43.1.2 
是自己的wan ip 並不是 local ip

把它改成127.0.0.1 說不定可以成功。


結果真的成功。











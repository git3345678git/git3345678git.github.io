---
layout: post
title: "Information disclosure on debug page"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Information_Disclosure
author: ""
tags: ['PortSwigger_Lab', 'Information_Disclosure']
---





# Information disclosure on debug page

LAB Not solved

This lab contains a debug page that discloses sensitive information about the application. To solve the lab, obtain and submit the `SECRET_KEY` environment variable.


此實驗室包含一個調試頁面，其中披露了有關應用程序的敏感信息。 解決實驗室，獲取並提交 `SECRET_KEY` 環境變量。





```
https://0a76005b049716eec0d03a63003f000f.web-security-academy.net/
```

bp抓包:
找到sitemap可以發現:

```
https://0a76005b049716eec0d03a63003f000f.web-security-academy.net/cgi-bin/phpinfo.php
```



搜尋`SECRET_KEY` 環境變量。
p38cljr44si0aa5au9qc2ixodkcnshjd



成功提交。




---
layout: post
title: "File path traversal, simple case"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Directory_Traversal
author: ""
tags: ['PortSwigger_Lab', 'Directory_Traversal']
---



# File path traversal, simple case


首頁
```
https://0a45002a03f2e364c0472dad00090030.web-security-academy.net/
```

抓包:
隨便點 view detail

```http
GET /image?filename=1.jpg HTTP/1.1
Host: 0a45002a03f2e364c0472dad00090030.web-security-academy.net
Cookie: session=UjOKc41066wjtLAbuUaKGl2AXGB0afYo
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:104.0) Gecko/20100101 Firefox/104.0
Accept: image/avif,image/webp,*/*
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://0a45002a03f2e364c0472dad00090030.web-security-academy.net/product?productId=3
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close


```


它會先去抓
filename=1.jpg

然後在顯示完整頁面。

這邊可以用bp intruder 讓它跑字典攻擊。

這邊用的字典是 payload all the things 的 Directory Traversal
https://github.com/swisskyrepo/PayloadsAllTheThings




最後可以看BP 的resonse 就知道成不成功。


這邊BP 的免費版，跑得實在是太慢。

我在想可不可以用用Kali 的 dirbuster


但是它只會顯示成功的目錄，不會有完整的response data 所以之後可能還是要自己手動去提交。不然只能自己寫爬蟲。


因為其它爆破工具，比較少可以自定義header 有些時候只有在登入狀態下的網站，就完全沒法。



看來看去還是BP 最好可以view response 可以看狀態碼，可以看data length。
















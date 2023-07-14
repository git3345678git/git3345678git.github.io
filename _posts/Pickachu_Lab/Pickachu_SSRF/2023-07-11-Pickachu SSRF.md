---
layout: post
title: "Pickachu SSRF"
description: ""
date: 2023-07-11
categories:
  - Pickachu_Lab
  - Pickachu_SSRF
author: ""
tags: ['Pickachu_Lab', 'SSRF']
---



# Pickachu SSRF
###### tags: `Pickachu`

常見函數:

curl 


file_get_contents
(有時可以使用UNC路徑)
```
\\localhost\c$\windows\win.ini
```














DICT 協議
詞典網絡協議，在RFC 2009中進行描述。它的目標是超越Webster protocol，並允許客户端在使用過程中訪問更多字典。Dict服務器和客户機使用TCP端口2628。



```
dict.org 站點提供字典
dict://  使用dict 協議


ex:

//查找 curl 這個字
curl dict://dict.org/find:curl


//m 帶表 match 查找 會發現回顯 多了 gcide "cur" 一堆 類似的 gcide 就是 dict.org 其中一個分類資料庫
curl dict://dict.org/m:curl



//查找heisenbug定義 會出現jargon
curl dict://dict.org/d:heisenbug


//查找heisenbug定義 指定分類資料庫
curl dict://dict.org/d:heisenbug:jargon


//以此類推
curl dict://dict.org/d:daniel:gcide


//查找所有分類資料庫
curl dict://dict.org/show:db


還有一堆命令.......

curl dict://dict.org/show:strat

curl dict://dict.org/show:server

curl dict://dict.org/status


```

### dict 命令詳解:
https://www.rfc-editor.org/rfc/rfc2229.html#section-3.5.4



dict协议  (字典协议,探测端口指纹信息,写入和反弹shell)

```
dict://localhost:80
dict://localhost:22
dict://localhost:21
dict://localhost:3306
```



我這邊就先做到探測端口，dict 是tcp協議，亂去連接可以拿到banner，但連接會失敗。


剩下寫webshell 好像都要redis，之後玩玩看。


還有一堆繞過WAF的payload















## 參考:

https://zhuanlan.zhihu.com/p/101163833
https://www.freebuf.com/vuls/365150.html
https://everything.curl.dev/usingcurl/dict
https://xz.aliyun.com/t/11215#toc-10
https://www.modb.pro/db/226551
https://www.twblogs.net/a/5eef69b6418820fe02fa1729

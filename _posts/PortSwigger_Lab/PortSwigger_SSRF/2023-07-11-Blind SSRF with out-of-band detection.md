---
layout: post
title: "Blind SSRF with out-of-band detection"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SSRF
author: ""
tags: ['PortSwigger_Lab', 'SSRF']
---





# Blind SSRF with out-of-band detection

Burpsuite之BurpCollaborator模塊介紹.
https://www.getit01.com/p2018082041377676/

OOB 案例:
https://www.jianshu.com/p/d544beaeadf4

要專業版





1.在商品頁面中BP抓包


2.用bp collaborator client 生成一個 server 網址
```
aesp3vkdtwuz8gssi8n3bh3ryi49sy.burpcollaborator.net
```


3.把refer 改成 https://aesp3vkdtwuz8gssi8n3bh3ryi49sy.burpcollaborator.net 

等待或按下pull now 


4.伺服器會有記錄。


這題也就是 當ssrf 請求在頁面沒有回顯時，可以透過dns log 伺服器，來看看對方伺服器有沒有執行我們的requet。



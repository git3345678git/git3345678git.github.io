---
layout: post
title: "Rogue potato"
description: ""
date: 2023-07-11
categories:
  - Windows後滲透
  - Potato
author: ""
tags: ['Windows後滲透', 'Potato']
---



# Rogue potato
###### tags: `potato提權` `window提權`




本文不會堤及太多原理，只會寫一些簡單的過程，小弟我功力太差，非常抱歉。



回顧:
OXID 解析器是“rpcss”服務的一部分，在端口 135 上運行。從 Windows 10 1809 和 Windows Server 2019 開始，不再可能在不同於 135 的端口上查詢 OXID 解析器

也就是說 juicy potato 沒用了。

既然端口被寫死135了 那麼指定遠程主機的135端口，135收到轉發到受害者的假設(6666)端口，上面運行假的 OXID RPC 服務器。



![](https://i.imgur.com/4mSjAX8.png)


圖中有些部分與事實不符，但是流程可以很好的解釋。

不符的地方:
步驟一並不是 system。



1.受害者發出請求，oxid 請求(為了知道去哪調用RPC)

2.轉發到假的 oxid resolver 解析

OXID Resolver是在支持COM +的每台计算机上运行的服务。
它执行两项重要职责：
它存储与远程对象连接所需的RPC字符串绑定，并将其提供给本地客户端。
它将ping消息发送到本地计算机具有客户端的远程对象，并接收在本地计算机上运行的对象的ping消息。OXID解析器的此方面支持COM +垃圾回收机制。




3.我們可以修改 oxid 的 response 傳給受害者
RPC 支持的協議，我們偽造的 OXID 解析器能選擇協議序列標識
| RPC transport | RPC protocol sequence string | 
| - | - | 
| SMB     |    ncacn_np  |
|  TCP/IP    |   ncacn_ip_tcp   |
| UDP     |  ncadg_ip_udp    |
|   SPX   |   ncacn_spx   |
|    IPX    |  ncadg_ipx      |
|  NetBIOS over IPX    |  ncacn_nb_ipx    |
|   NetBIOS over TCP   |  ncacn_nb_tcp    |
|  NetBIOS over NetBEUI   |  ncacn_nb_nb    |
|  AppleTalk	    |   ncacn_at_dsp   |
|   RPC over HTTP   |  ncacn_http    |


作者 一開始選用 ncacn_ip_tcp 最終失敗

接者他選用 ncacn_np 使用命名管道

他看上了 RpcEptMapper  服務
這個服務和 rpcss 服務共享進程空間，並且都在 NETWORK SERVICE 帳戶下運行

預設指定連接到\pipe\epmapper

也就是說，我們最終給受害者一個管道，連接到這管道中你可以獲取RPC接口。

那要怎麼突破預設管道，反而連接到攻擊者的pipe呢

在Print spoofer 漏洞中你可以看到答案，就是一個路徑檢查bug


最終我們可以拿到 NETWORK SERVICE帳戶的模擬令牌以及RPCSS服務的LUID！ 


拿到後還需要使用 token kidnapping 技術 拿到system



### token kidnapping

這領域我也不懂(先填坑)

对每个进程进行爆破，直到找到满足如下条件的进程：
进程运行用户是SYSTEM
令牌级别至少是Impersonation级别
攻击者运行的权限至少拥有SeImpersonatePrivilege


介紹:
https://i.blackhat.com/asia-21/Thursday-Handouts/as21-Cocomazzi-The-Rise-of-Potatoes-Privilege-Escalations-in-Windows-Services.pdf

https://www.anquanke.com/post/id/204721


解碼器的博客 - 我們認為它們是土豆，但它們是豆子（再次從服務帳戶到系統）
https://decoder.cloud/2019/12/06/we-thought-they-were-potatoes-but-they-were-豆子/

GitHub - antonioCoco / RogueWinRM（從服務帳戶到系統的 Windows 本地權限升級）https://github.com/antonioCoco/RogueWinRM




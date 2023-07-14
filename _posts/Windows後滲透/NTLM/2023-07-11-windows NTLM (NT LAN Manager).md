---
layout: post
title: "windows NTLM (NT LAN Manager)"
description: ""
date: 2023-07-11
categories:
  - Windows後滲透
  - NTLM
author: ""
tags: ['Windows後滲透', 'NTLM']
---



# windows NTLM (NT LAN Manager)
###### tags: `Windows 域滲透` `window提權`

## 1.NTLM本地认证


Windows将用户的密码存储在本地计算机的SAM文件中，文件位置：C:\Windows\System32\config\SAM

winlogon 接收用戶輸入密碼 -> LSASS將密碼緩存，並hash密碼，然後比對SAM中的hash，比對成功給用戶會話令牌(會話這邊不繼續深入)。




## 2.NTLM 网络认证 (重點探討)

NTLM的网络认证，仔细细分可以分为工作组环境下的认证和域环境下的认证。大致原理相同，都是采用Challenge/Response验证机制。

* 工作組(workgroup)，中間無可信任機構的認證，
無信任機構
A 要直接轉帳給 B。 

* 域环境有可信任機構，(Kerberos 本文不會探討這塊)
有信任機構，就像是銀行一樣，A 在銀行開戶，B也在銀行開戶，所以 A B 都是銀行認可的用戶，A轉帳給B中間透過銀行來認證。



NTLM 是一種認證的協議，讓雙方可以確認對方是否是可信任的



### 相比HTTP SMB 這種協議或是 他比較底層，所以http SMB 都可以使用它。

#### 例如:
* client 連接 server(開啟smb文件共享)，雙方透過NTLM 的認證後才可以交互。

例如
1. 一家高檔酒店有提供頂級VIP按摩服務，你需要跟櫃檯人員進行身分驗證才能進去挑小姐

2. 外送員有提供餐點的服務，你需要雙方確認身分驗證後才能取餐。


#### 早期smb協議是明文傳輸後來出現LAN Manager(LM)，採用challenge/response 驗證機制，但LM算法很容易破解，衍生出NTLM，目前有: 

1. NTLMv1
2. NTLMv2(目前大多數採用)
3. Kerberos(域環境)





## challenge/response 驗證機制





### client  -> server

用户输入帳號密码候 client先会在本地缓存一份值对应的NTLM Hash
client 發出協商(Negotiate)給server，目的為確認採用的版本 v1還是v2，還有需多複雜的東西包含client 發送過來的用戶名


### server -> client

服务端接收到Negotiate协商消息之后，会将数据传输给NTLM SSP进行处理，然后获得一个返回的16位随机值，称之为Challenge。

使用 client傳過來的用戶名找到server上存在用戶對應的密文 NTLM hash 加密 challenge  稱為 challenge1 在server本地緩存challenge1，並把 challenge 丟給client


### client -> server

client 收到 challenge (16位随机值)，把一開始本地緩存的NTLM Hash 加密 challenge 也就是response(Net NTLM hash)丟給 server



### server -> client 
server 比對 challenge1 跟 response 是否相等，是則告訴client 通過。






跨协议中继


由于 SMB、HTTP、LDAP、MSSQL 等协议都可以携带 NTLM 认证的三类消息，所以只要是使用 SMB、HTTP、LDAP、MSSQL 等协议来进行 NTLM 认证的程序

NTLM 本身只是一套身份认证协议，需要其他上层协议使用它，如 SMB、HTTP 可以使用 NTLM 进行身份认证。即 NTLM 是嵌在上层协议中的，认证成功并建立会话后上层协议再进行自己后续的数据传输。理论上任何协议都可以使用这套协议完成身份认证，NTLM 协议可以嵌入到任何协议里。


因此，不同协议之间是可以进行跨协议中继的。比如从 HTTP 中继至 SMB，从 SMB 中继至 LDAP/Mssql 等等，只要协议双方都支持 NTLM 即可。简单地说，只需将一个协议中的 NTLM 消息取出来，然后轻轻的放入另一个协议，就完成了上层协议转换的过程。


總結以上，HTTP 是不能夠直接跟SMB 進行通訊(應用層)，但是底層協議大家都看得懂 NTLM 认证，


以下我有兩種猜測，我一直不確定，到底是本身能就能進行NTLM驗證，還是需要一個中間人來驗證。

第一種本身能就能進行NTLM驗證
1协商
http 對 smb 發出協商

2.质询
smb 看得懂 發出challenge 給 http

3.验证
http 收到challenge 把 response 丟給 smb  比對確認後驗證通過



第二種

1协商
client透過　http 對中間人發出協商，中間人把 http 的協商 轉成 smb的協商 告訴 smb server

2.质询
smb server 對中間人發出 challenge  中間人把 smb 的challenge 轉成http challenge 告訴 client

3.验证
client  對中間人發出 response，中間人把 http 的response 轉成smb 的 response 比對後驗證通過



我個人認為應該是第二種，不然不會有跨协议中继這個詞。直接跨協議就好。


上訴的都是簡化的流程，用意在於理解方式，而真正的原理步驟是更複雜的。


很多攻擊就是使用這招來完成，





以下是詳細的window認證請求頭資源(HTTP NTLM認證):
https://www.cnblogs.com/chnking/archive/2007/11/20/965553.html#_Toc183326158




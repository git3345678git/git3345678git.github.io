---
layout: post
title: "Juicy Potato(Rotten Potato 改版)"
description: ""
date: 2023-07-11
categories:
  - Windows後滲透
  - Potato
author: ""
tags: ['Windows後滲透', 'Potato']
---



# Juicy Potato(Rotten Potato 改版)
###### tags:  `potato提權` `window提權`

此技術不適用於 >= Windows 10 1809 和 Windows Server 2019 的版本 

與 rotten potato 一樣。


rotten potato 是使用硬編碼 6666端口和默認BITS 的 CLSID


### Juicy potato 特點: 
* 1.可以指定我們的 COM 服務器偵聽端口


* 2.除了 BITS 之外，還有幾個由特定 CLSID 標識的進程外 COM 服務器可能會被濫用，
他們要符合以下幾點:

```
* 可由當前用戶實例化，通常是具有模擬權限的服務用戶
* 實現IMarshal接口
* 以提升的用戶身份運行（SYSTEM，Administrator，...）
```

CLSID 查詢:
http://ohpe.it/juicy-potato/CLSID/

* 3. rotten potato 仰賴於 meterpreter和Decoder做的 incognito 模塊 。juicy potao 不需要 meterpreter shell



參考:


https://jlajara.gitlab.io/Potatoes_Windows_Privesc

https://ohpe.it/juicy-potato/CLSID/


https://ohpe.it/juicy-potato/

https://3gstudent.github.io/Windows%E6%9C%AC%E5%9C%B0%E6%8F%90%E6%9D%83%E5%B7%A5%E5%85%B7Juicy-Potato%E6%B5%8B%E8%AF%95%E5%88%86%E6%9E%90


https://blog.csdn.net/qq_41874930/article/details/109766105

http://moonflower.fun/index.php/2022/05/01/329/

https://xz.aliyun.com/t/7776


https://jiancanxuepiao.github.io/2019/05/30/Windows%E6%9C%AC%E5%9C%B0%E6%8F%90%E6%9D%83%E5%B7%A5%E5%85%B7Juicy-Potato%E6%B5%8B%E8%AF%95/
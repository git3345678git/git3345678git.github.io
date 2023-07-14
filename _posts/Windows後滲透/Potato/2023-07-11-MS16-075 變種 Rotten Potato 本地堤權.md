---
layout: post
title: "MS16-075 變種 Rotten Potato 本地堤權"
description: ""
date: 2023-07-11
categories:
  - Windows後滲透
  - Potato
author: ""
tags: ['Windows後滲透', 'Potato']
---



# MS16-075 變種 Rotten Potato 本地堤權 
###### tags: `potato提權` `window提權`

影響版本：< win10 1809 和 < windows server 2019



已下介紹不會太詳細，只是記錄自己學習到重點知識和過程，更詳細的說明可以查看作者文章:
https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/



### 先了解幾個知識點

* RPC:
就是把本地【進程內】的函數調用，放到同一機器的另一進程，或者不同機器的進程的技術。這個技術減輕分佈式開發負擔，使分佈式開發標準化。 RPC主要涉及通信和序列化兩部分。
https://zhuanlan.zhihu.com/p/187560185



* COM 和 DCOM:
是以RPC (Remote Procedure Call)使不同的行程可以相互的溝通，所謂的DCOM則是可以進行跨機器的溝通，這也是COM+的基礎。 (好文分享)
https://dotblogs.com.tw/gelis/2010/12/07/19954




* CLSID:
CLSID是指windows系統對於不同的應用程式，檔案類型，OLE對象，特殊資料夾以及各種系統組件分配的一個唯一表示它的ID代碼，用於對其身份的標識和與其他對象進行區分。
https://www.cnblogs.com/boltkiller/articles/4791503.html


* BITS(後台智能傳輸服務) 
是Microsoft Windows操作系統中的一個系統組件
CLSID 為 {4991d34b-80a1-4291-83b6-3328366b9097}
https://zh.gov-civil-braga.pt/advantages-disadvantages-windows-10-fast-startup-feature

BITS 服務是system 權限
https://xz.aliyun.com/t/7935
程序員和系統管理員使用後台智能傳輸服務 (BITS)從 HTTP Web 服務器和 SMB 文件共享中下載或上傳文件。關鍵是BITS實現了IMarshal接口並允許代理聲明強制 NTLM 身份驗證。




### 角色:

1.dcom (客戶端)

2.中間人 (localhost:6666)

3.rpc 端口135 (服務端)


### 流程圖:
![](https://i.imgur.com/HFBYfHP.png)

potato 會透過 com API 調用指定了 BITS CLSID  {4991d34b-80a1-4291-83b6-3328366b9097}

設定了 127.0.0.1:6666 端口

#### 目的:告訴com 說我要使用 從 127.0.0.1:6666 拿到 BITS 的 對象

```com 
public static void BootstrapComMarshal()
{
IStorage stg = ComUtils.CreateStorage();
 
// Use a known local system service COM server, in this cast BITSv1
Guid clsid = new Guid("4991d34b-80a1-4291-83b6-3328366b9097");
 
TestClass c = new TestClass(stg, String.Format("{0}[{1}]", "127.0.0.1", 6666)); // ip and port
 
MULTI_QI[] qis = new MULTI_QI[1];
 
qis[0].pIID = ComUtils.IID_IUnknownPtr;
qis[0].pItf = null;
qis[0].hr = 0;
 
CoGetInstanceFromIStorage(null, ref clsid, null, CLSCTX.CLSCTX_LOCAL_SERVER, c, 1,       qis);
}
```

中間會交換蠻多包的(我自己也不懂細節)，總之我們只需要知道最後 com 會發出 NTLM 驗證


#### 最終 COM 將嘗試通過發送 NTLM type 1（協商）消息向中間人發起 NTLM 身份驗證


### 中間人拿到 type 1（協商）後 要做兩件任務:

* 任務1
調用函數API AcquireCredentialsHandle 來獲取我們需要的數據結構的句柄。
在調用 AcceptSecurityContext 此函數的輸入將是 NTLM Type 1（協商）消息。輸出將是一條 NTLM Type 2（質詢）消息


![](https://i.imgur.com/OKr9n4y.png)


* 任務2
轉發type 1 到 rpc 135 讓他返回一個  NTLM Type 2（質詢）消息




### 中間人會拿到兩個 type 2（質詢）消息 

* 任務1 是為了使用 NTLM 驗證 正確使用 AcceptSecurityContext 
* 任務2 是一開始com 就是用RPC 發出的type 1 理所當然 com 也期望收到 rpc 返回的 type 2



### 中間人會拿到兩個 type 2 後修改一下再轉給 com


![](https://i.imgur.com/TCfnNDM.png)

左邊為 rpc 135 的 type 2 包，我們用這個包的架構，在修改一下
NTLM Server Challenge”及其下方的字段“Reserved”值，變成右邊的包

這個修改的內容來自於  AcceptSecurityContext 輸出。



### 為啥要這樣做? 原因是 任務2 的 AcceptSecurityContext 

“Reserved”字段實際上是對一個SecHandle的引用，當SYSTEM賬號收到NTLM Type 2消息後，會在內存中進行後台認證。

“Challenge”和“Reserved”字段與“AcceptSecurityContext”的輸出相匹配代表驗證成功。

如果是用原本的 "Challenge"和"Reserved" 他會跑去 rpc 135 驗證



完成後，COM 代表 SYSTEM 帳戶將向我們發回 NTLM type 3（身份驗證）數據包。
但實際上是空的（因為這裡所有的實際身份驗證都發生在內存中），總之需要用他調用AcceptSecurityContext，最後返回的結果在調用ImpersonationToken 获得一个模拟令牌

### AcceptSecurityCoNtext 函數
https://learn.microsoft.com/zh-tw/windows/win32/secauthn/acceptsecuritycontext--general


返回值:SEC_E_OK
代表:如果函式產生輸出權杖，則必須將它傳送至用戶端進程。



### 調用  ImpersonationToken
模擬令牌（Impersonation Token）：在默認的情況下，當線程被開啟的時候，所在進程的主令牌會自動附加到當前線程上，作為線程的安全上下文。而線程可以運行在另一個非主令牌的訪問令牌下執行，而這個令牌被稱為模擬令牌。而指定線程的模擬令牌的過程被稱為模擬


###  ImpersonationToken 這塊我也不懂，只能先挖坑了。

屬於 windows token 之類的領域
介紹:
https://cloud.tencent.com/developer/article/1021371#:~:text=%E6%A8%A1%E6%8B%9F%E4%BB%A4%E7%89%8C%EF%BC%88Impersonation%20Token,%E8%BF%87%E7%A8%8B%E8%A2%AB%E7%A7%B0%E4%B8%BA%E6%A8%A1%E6%8B%9F%E3%80%82




總之這篇粗淺的簡介，可以幫助我快速的理解，因為這塊領域我沒有真正的了解


### 最後原作者堤到:
果我們想模擬令牌，我們最好以具有 SeImpersonate 權限（或同等權限）的帳戶身份運行。幸運的是，這包括 Windows 中的許多服務帳戶，滲透測試人員通常最終以這些帳戶運行。例如，IIS 和 SQL Server 帳戶。





### Lonely Potato
是Rotten Potato的改編，不依賴meterpreter和Decoder做的“incognito”模塊。

Lonely Potato 已被棄用，有跡象表明要移至Juicy Potato。










### 參考:

https://jlajara.gitlab.io/Potatoes_Windows_Privesc

https://decoder.cloud/2017/12/23/the-lonely-potato/

https://cn-sec.com/archives/659470.html

http://moonflower.fun/index.php/2022/05/01/329/

https://www.freebuf.com/articles/web/249782.html
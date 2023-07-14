---
layout: post
title: "windows NTLM relay 和 reflection 攻擊"
description: ""
date: 2023-07-11
categories:
  - Windows後滲透
  - NTLM
author: ""
tags: ['Windows後滲透', 'NTLM']
---



# windows NTLM relay 和 reflection 攻擊

###### tags: `Windows 域滲透` `window提權`

## 1.NTLM 中繼(relay) 

觀念跟ARP 攻擊基本上很類似

1. 攻擊者讓客戶端向攻擊者服務器發起 NTLM 挑戰響應請求
2. 攻擊者服務器向真實服務端發起 NTLM 請求，真實服務端返回 challenge
3. 攻擊者服務器將收到的 challenge 發送給客戶端
4. 客戶端返回 response，攻擊者服務器將 response 發送給服務端
5. 服務端驗證通過，攻擊者服務器獲得服務端權限


要怎麼當中間人?

中繼的本質是中間人攻擊，利用LLMNR, NetBIOS欺騙 , WPAD 劫持


可以參考:
LLMNR和NetBIOS欺騙攻擊分析及防範
https://xz.aliyun.com/t/9714




網上對這類攻擊有非常詳細的介紹(參考來源會有很多，可以了解很有幫助)


1. 需要用戶交互的方法
這類方法都需要用戶參與，讓用戶有意無意的去訪問一個 UNC 路徑，這樣客戶端就會通過 SMB 協議向 UNC 路徑指向的服務器發起 NTLM 挑戰請求。如讓用戶點擊攻擊者精心製作的且帶有 UNC 路徑的郵件、web 頁面、文檔等；或者運維人員輸入 net use \xxx 等帶 UNC 路徑的命令、辦公人員在資源管理器中通過 UNC 路徑訪問共享服務器等。
對於前者而言，可以理解成內網釣魚，這種方法可以釣取客戶端的 Net-NTLM Hash 然後本地爆破；對於後者而言，需要配合內網劫持，使用名稱解析協議來劫持客戶端流量，畢竟運維人員 net use \xxx 要訪問哪台服務器、辦公人員要訪問哪台共享服務器攻擊者並不可控，所以需要名稱解析欺騙將客戶端流量引導到被控服務器上。

2. 無需用戶交互的方法
通過 SpoolSample 、PetitPotam 等方法無需用戶交互就可以強制客戶端機器賬戶向指定機器發起 NTLM 請求，這種方法無感知、無需交互，是在實戰中重點利用的手法。







## 2.NTLM reflection (基於 NTLM relay)



前面介紹的 NTLM relay 場景是 client 對 server (兩個不同主機)，攻擊者是中間人，欺騙雙方最終拿到server 的訪問權限(例如SMB文件共享)


NTLM reflection 則是 受害者只有client。


在使用某些命令時，會先使用客戶端自身的憑證來嘗試驗證。比如輸入net.exe use \hostshare 並回車後會提示輸入服務端賬號密碼，其實在提示輸入賬號密碼之前客戶端就已經用當前用戶名及其 NTLM-Hash 進行挑戰響應驗證。顯然這會因為客戶端憑證無法用於服務端而失敗，之後用戶再輸入正確的服務端賬號密碼，客戶端再進行一遍挑戰響應驗證完成身份認證。



場景:
A client 訪問 B server (SMB文件共享)

假設我們已經拿到一台主機，我們使用LLMNR和NetBIOS欺騙(作為中間人)，而 A 想訪問 B 被我們監聽到我們把A的請求送回A本身。 (假設A也開啟了SMB服務)

A 帶著自己的憑證驗證B

攻擊者把A的憑證拿去驗證A的SMB服務。

你可能會覺得多此一舉，A訪問B使用某些命令時，會先使用客戶端自身的憑證來嘗試驗證，我們只要監聽就能拿到A的憑證，攻擊者隨時都能訪問A的smb服務，幹嘛還要反射讓A自己訪問自己的smb服務。



了解問題後有一個實用場景是Potato家族的本地提權，

某些情況下我們拿到了shell 但是權限不夠，就可以利用反射來提權。

場景:

我們已經拿到A主機的shell 但權限權限不夠，
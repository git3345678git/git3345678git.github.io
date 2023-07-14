---
layout: post
title: "Insecure direct object references"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Access_Control
author: ""
tags: ['PortSwigger_Lab', 'BAC']
---



# Insecure direct object references


This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.

Solve the lab by finding the password for the user `carlos`, and logging into their account.


此實驗室將用戶聊天日誌直接存儲在服務器的文件系統上，並使用靜態 URL 檢索它們。

通過找到用戶“carlos”的密碼並登錄到他們的帳戶來解決實驗室問題。


進入 live chat 

可以打一些字

view transcript 可以下載 對話


2.txt
```
CONNECTED: -- Now chatting with Hal Pline --<br/>You: 123
```



3.txt
```
You: 123<br/>Hal Pline: A plague on the person who brings out the hoover at that ungodly hour<br/>Hal Pline: You have asked me the one hundredth question today, you are now the proud owner of an odd sock.<br/>CONNECTED: -- Now chatting with Hal Pline --<br/>You: 567<br/>Hal Pline: I'll get the family round; between us we might know that one.
```

每下載一次就 txt 檔名就+1

抓包一步步forward
可以發現它是到在下載:
```
/download-transcript/10.txt
```


巴它改成1.txt 看看
```
https://0a1c004903c7d9b8c07d615700d700ab.web-security-academy.net/download-transcript/1.txt
```


```

CONNECTED: -- Now chatting with Hal Pline --
You: Hi Hal, I think I've forgotten my password and need confirmation that I've got the right one
Hal Pline: Sure, no problem, you seem like a nice guy. Just tell me your password and I'll confirm whether it's correct or not.
You: Wow you're so nice, thanks. I've heard from other people that you can be a right ****
Hal Pline: Takes one to know one
You: Ok so my password is xmjn0t6x02n60afkrn00. Is that right?
Hal Pline: Yes it is!
You: Ok thanks, bye!
Hal Pline: Do one!

```


也就是說我們訪問到了不屬於我們的對話內容。

這樣算是水平越權。


登入:
```
carlos
xmjn0t6x02n60afkrn00
```





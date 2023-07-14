---
layout: post
title: "Information disclosure in error messages"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Information_Disclosure
author: ""
tags: ['PortSwigger_Lab', 'Information_Disclosure']
---





# Information disclosure in error messages
錯誤消息中的信息洩露



google 插件:
whatrun 跑不出來

登入狀態下其它 tool 也不好使。



手動找:


隨便找一個品項view detail:

```
https://0a4300db0373d1a9c05151e100e000d3.web-security-academy.net/product?productId=3
```

productId=999
顯示:not found


productId=99999999
顯示:not found

productId=9999999999999

可能超出了整數範圍

error
```
Internal Server Error: java.lang.NumberFormatException: For input string: "9999999999999"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Integer.parseInt(Integer.java:668)
	at java.base/java.lang.Integer.parseInt(Integer.java:786)
	at lab.l.e.e.w.E(Unknown Source)
	at lab.p.v.p.p.L(Unknown Source)
	at lab.p.v.z.a.q.a(Unknown Source)
	at lab.p.v.z.u.lambda$handleSubRequest$0(Unknown Source)
	at h.v.b.n.lambda$null$3(Unknown Source)
	at h.v.b.n.G(Unknown Source)
	at h.v.b.n.lambda$uncheckedFunction$4(Unknown Source)
	at java.base/java.util.Optional.map(Optional.java:260)
	at lab.p.v.z.u.c(Unknown Source)
	at lab.a.p.e.o.z(Unknown Source)
	at lab.p.v.l.z(Unknown Source)
	at lab.a.p.e.h.s(Unknown Source)
	at lab.a.p.e.h.C(Unknown Source)
	at h.v.b.n.lambda$null$3(Unknown Source)
	at h.v.b.n.G(Unknown Source)
	at h.v.b.n.lambda$uncheckedFunction$4(Unknown Source)
	at lab.a.g1.C(Unknown Source)
	at lab.a.p.e.h.x(Unknown Source)
	at lab.a.p.l.n.q(Unknown Source)
	at lab.a.p.o.M(Unknown Source)
	at lab.a.y.o(Unknown Source)
	at lab.a.y.J(Unknown Source)
	at lab.a.y.V(Unknown Source)
	at h.v.x.e.i.C(Unknown Source)
	at h.v.x.e.i.r(Unknown Source)
	at h.v.x.e.i.run(Unknown Source)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
	at java.base/java.lang.Thread.run(Thread.java:833)

Apache Struts 2 2.3.31
```

提交:
2 2.3.31


完成




驗證漏洞:

32bit整數範圍:

```
範圍
2147483647   ---- −2147483648

```


productId=2147483647
```
https://0a4300db0373d1a9c05151e100e000d3.web-security-academy.net/product?productId=2147483647
```
not found




productId=2147483648
```
https://0a4300db0373d1a9c05151e100e000d3.web-security-academy.net/product?productId=2147483648
```
直接報錯








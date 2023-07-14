---
layout: post
title: "DVWA Weak Session ID (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_Weak_Session
author: ""
tags: ['DVWA', 'Weak_Session']
---



# DVWA Weak Session ID (教學用)
###### tags: `DVWA`
## low

這題是在模擬不同使用者登入的 session id 主要是 session 規則太好猜，身分容易被冒用。




每次單擊該按鈕時，此頁面都會設置一個名為 dvwaSession 的新 cookie。
### html
```htmlembedded=
<form method="post">
    <input type="submit" value="Generate">
</form>
```

### php
```php=

<?php

$html = "";


if ($_SERVER['REQUEST_METHOD'] == "POST") {
    
    //如果沒 last_session_id 就設定0
    //代表第一次登入
    if (!isset ($_SESSION['last_session_id'])) {
        $_SESSION['last_session_id'] = 0;
    }
    
    
    //如果有 last_session_id 就+1
    $_SESSION['last_session_id']++;
    
    $cookie_value = $_SESSION['last_session_id'];
    
    //set cookie 
    setcookie("dvwaSession", $cookie_value);
}
?>
```




點兩下 cookie 多了一個 dvwaSession
```
dvwaSession=1
dvwaSession=2
.
.
.
.
```








## medium


跟上面一樣基本一樣。
### 有時session 為了單一性不重複，會加上時間來運算。

```
dvwaSession=1685158214

dvwaSession=1685158280
.
.
.
```

### timestamp(時間戳記)  工具
https://www.epochconverter.com/

```
1685158214
Assuming that this timestamp is in seconds:
GMT: 2023年5月27日Saturday 03:30:14
Your time zone: 2023年5月27日星期六 11:30:14 GMT+08:00




1685158280
Assuming that this timestamp is in seconds:
GMT: 2023年5月27日Saturday 03:31:20
Your time zone: 2023年5月27日星期六 11:31:20 GMT+08:00


```





## high

跟上面一樣基本一樣。


dvwaSession


看起來使用 hash
```
c4ca4238a0b923820dcc509a6f75849b

c81e728d9d4c2f636f067f89cc14862c
```

### hash identifer 識別hash 類別
https://hashes.com/en/tools/hash_identifier

### result 
```
c4ca4238a0b923820dcc509a6f75849b - Possible algorithms: MD5

c81e728d9d4c2f636f067f89cc14862c - 2 - Possible algorithms: MD5

```


### decrypt hash
```
c4ca4238a0b923820dcc509a6f75849b:1
c81e728d9d4c2f636f067f89cc14862c:2
```
所以知道 一樣是 1,2,3,4,5,6,..... 只不過被hash了。




---
### burpsuite 改 cookie

重新抓第一次
```
POST /vulnerabilities/weak_id/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/weak_id/
Cookie: PHPSESSID=cea399ecd6a812d693aa7679fcb4de03; security=low
Upgrade-Insecure-Requests: 1


```

第二次
dvwaSession=1
```http
POST /vulnerabilities/weak_id/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/weak_id/
Cookie: dvwaSession=1; PHPSESSID=cea399ecd6a812d693aa7679fcb4de03; security=low
Upgrade-Insecure-Requests: 1

```



第三次
改 dvwaSession=100
```http
POST /vulnerabilities/weak_id/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/weak_id/
Cookie: dvwaSession=100; PHPSESSID=cea399ecd6a812d693aa7679fcb4de03; security=low
Upgrade-Insecure-Requests: 1

```

發現沒卵用，這題的概念就是找出session 規律，實戰中你可以透過猜測，或是盜取cookie來獲得登入狀態。

## 實驗(課外題)

### 既然這題沒用，那我們就來做個實驗 dvwa 剛好需要登入，他肯定也有session 機制。


實驗流程:

* 兩個不同browser
firefox, google 


這邊我們要再google 登入dvwa 然後假裝偷到他的session，放給 firefox。 冒充使用者。




google
```
PHPSESSID=fdf8d4b2bbd3393fd4acf8423c54b74b
```

### 修改firefox cookie
* 方式有很多種。
1. 你可以用 burpsuite 把每個包cookie 都換chrome的
2. firefox 安裝 eidt this cookie 插件(修改cookie 的插件都可以)。


### 使用插件很簡單就是修改值

修改後我們觀察 chrome 已登入dvwa 的頁面在哪。

這個頁面是登入狀態下才能看的。
```
http://localhost/security.php
```

firefox 直接 enter 進入 你會發現不用輸入帳密，你已經登入了。

這就是冒用 session。




## impossible


```php
 <?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    
    $cookie_value = sha1(mt_rand() . time() . "Impossible");
    
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], true, true);
}
?>

```


可以看到使用了 sha1(隨機數+ 時間 + Impossible 字段)，太難猜了。
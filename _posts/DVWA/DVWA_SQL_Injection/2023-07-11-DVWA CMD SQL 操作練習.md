---
layout: post
title: "DVWA CMD SQL 操作練習"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_SQL_Injection
author: ""
tags: ['DVWA', 'SQL_Injection']
---



# DVWA CMD SQL 操作練習

進去sql
```
mysql -h 127.0.0.1 -u root -p
root
```

show 資料庫:
```
show databases;
// echo dvwa

use dvwa;

show tables;
//echo users 


SHOW COLUMNS FROM users;
//總共8個欄位

select database();
//顯示當前資料庫


```


```

select user from users;
//總共5個使用者

select password from users;
//總共5個密碼

select 'aaa' from users;
//也就是說 選擇 字串'aaa' 也會出現五筆資料



///////////////////////////////////////////////////////////////

//顯示1筆資料
select user from users limit 1;

//顯示2筆資料
select user from users limit 2;

//顯示3筆資料
select user from users limit 3;

///////////////////////////////////////////////////////////////



//因此只會出現一筆 'aaa'
select 'aaa' from users limit 1;

//沒有shit表會報錯
select 'aaa' from shit limit 1;




///////////////////////////////////////////////////////////////




// 'aaa' = 'aaa' 會是 true
(select 'aaa' from users limit 1)='aaa';



// 1 AND 1 會回傳 1
select 1 AND (select 'aaa' from users limit 1)='aaa';



// 1 AND 0 會回傳 0
select 1 AND (select 'aaa' from users limit 1)='bbb';




///////////////////////////////////////////////////////////////




//我們可以利用上面的知識，去判斷 user 這張表是否存在
select 1 AND (select 'aaa' from users limit 1)='aaa';



// 沒有 shit 這張表，所以報錯。
select 1 AND (select 'aaa' from shit limit 1)='aaa';


*****************************************
總結 我們現在拿到 users 表
*****************************************


////////////////////////////////////////////////////////////////


//users表中 選擇 user欄位 只要是 user  = 'admin'

select user from users where user='admin';
// echo admin



//users表中 選擇 欄位為一個字串'aaa' 只要是 user  = 'admin'

select 'aaa' from users where user='admin';
// echo aaa



//users表中 選擇 欄位為一個字串'aaa' 只要是 user  = 'admin123456'

select 'aaa' from users where user='admin123456';
//echo 0     因為沒有一個 user 叫做 'admin123456'




select 1 AND (select 'aaa' from users where user='admin')= 'aaa';
只要 有個 user 叫做 'admin'  ###   'aaa' = 'aaa'

*****************************************
總結 我們可以知道 一個user 的名子 admin
*****************************************

////////////////////////////////////////////////////////////////


// where 條件式 多加一個 LENGTH(password)>1 

//密碼長度 > 1 =TRUE
select 'aaa' from users where user='admin' AND LENGTH(password)>1;

//密碼長度 < 1 =FALSE
select 'aaa' from users where user='admin' AND LENGTH(password)<1;


//此時可以開始猜測 admin 的密碼 長度
select 1 AND (select 'aaa' from users where user='admin' AND LENGTH(password) > 1)='aaa';

select 1 AND (select 'aaa' from users where user='admin' AND LENGTH(password) > 5)='aaa';

select 1 AND (select 'aaa' from users where user='admin' AND LENGTH(password) > 9)='aaa';


### 這邊偷懶一下，直接show admin 的密碼長度。因為通常會加密，所以都很長。
select length(password) from users where user='admin';
//echo 32 長度 32 




看到這邊可以回去看buprsuite lab的盲注了。























```
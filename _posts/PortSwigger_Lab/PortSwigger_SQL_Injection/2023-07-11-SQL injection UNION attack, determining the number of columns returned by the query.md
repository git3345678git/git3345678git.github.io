---
layout: post
title: "SQL injection UNION attack, determining the number of columns returned by the query"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





 
# SQL injection UNION attack, determining the number of columns returned by the query



### 教學:
https://www.anquanke.com/post/id/245532


```
This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query. You will then use this technique in subsequent labs to construct the full attack.

To solve the lab, determine the number of columns returned by the query by performing an [SQL injection UNION](https://portswigger.net/web-security/sql-injection/union-attacks) attack that returns an additional row containing null values.

1.  Use Burp Suite to intercept and modify the request that sets the product category filter.
2.  Modify the `category` parameter, giving it the value `'+UNION+SELECT+NULL--`. Observe that an error occurs.
3.  Modify the `category` parameter to add an additional column containing a null value:
    
    `'+UNION+SELECT+NULL,NULL--`
4.  Continue adding null values until the error disappears and the response includes additional content containing the null values.


本實驗室在產品類別過濾器中包含一個 SQL 注入漏洞。 查詢的結果在應用程序的響應中返回，因此您可以使用 UNION 攻擊從其他表中檢索數據。 這種攻擊的第一步是確定查詢返回的列數。 然後，您將在後續實驗中使用此技術來構建完整的攻擊。

要解決該實驗，請通過執行 SQL 注入 UNION 攻擊來確定查詢返回的列數，該攻擊返回包含空值的附加行。


1. 使用 Burp Suite 攔截和修改設置產品類別過濾器的請求。
2. 修改`category`參數，給它值`'+UNION+SELECT+NULL--`。 觀察是否發生錯誤。
3. 修改 `category` 參數以添加一個包含空值的附加列：
    
     `'+UNION+SELECT+NULL,NULL--`
4. 繼續添加空值，直到錯誤消失並且響應包含包含空值的附加內容。

```





url:
```
https://0a64005e047e48dbc11b248500ae00fd.web-security-academy.net/filter?category=Lifestyle
```

叫我們使用聯合注入。也就是可以順便查詢其他語句

ex:
```
select * from user where username='dog' UNION select * from icon where IconName='dog'
```

但是聯合注入有幾個條件。

兩種不同的查詢語句，列數必須要一樣。


這邊可以用以下 慢慢去測試，聯合注入需要多少欄位。
```
UNION select 1

UNION select 1,2

UNION select 1,2,3
```

但這題硬要我們用 NULL 去測。

原因:
NULL 可以兼容所有數據，所以更好用。


```

category=Lifestyle'+UNION+SELECT+NULL--
//顯示:Internal Server Error



category=Lifestyle'+UNION+SELECT+NULL,NULL--
//顯示:Internal Server Error



category=Lifestyle'+UNION+SELECT+NULL,NULL,NULL--
//成功



```
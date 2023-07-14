---
layout: post
title: "Source code disclosure via backup files"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Information_Disclosure
author: ""
tags: ['PortSwigger_Lab', 'Information_Disclosure']
---





# Source code disclosure via backup files

通過備份文件洩露源代碼




This lab leaks its source code via backup files in a hidden directory. To solve the lab, identify and submit the database password, which is hard-coded in the leaked source code.

該實驗室通過隱藏目錄中的備份文件洩露其源代碼。 解決實驗室，識別並提交數據庫密碼，該密碼硬編碼在洩露的源代碼中。


```
https://0a0f009b04dc4007c08c1fb700500097.web-security-academy.net/
```

加上/robot.txt

```
https://0a0f009b04dc4007c08c1fb700500097.web-security-academy.net/robots.txt


response:
User-agent: *
Disallow: /backup


```

找到backup 目錄


```
https://0a0f009b04dc4007c08c1fb700500097.web-security-academy.net/backup
```


出現一個備份檔案:
ProductTemplate.java.bak

```
package data.productcatalog;

import common.db.ConnectionBuilder;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "81dew78ff6zjff4scicilx80whccml2s"
        ).withAutoCommit();
        try
        {
            Connection connect = connectionBuilder.connect(30);
            String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
            ResultSet resultSet = statement.executeQuery(sql);
            if (!resultSet.next())
            {
                return;
            }
            product = Product.from(resultSet);
        }
        catch (SQLException e)
        {
            throw new IOException(e);
        }
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}
```

看起來應該是java 入庫:

密碼:
81dew78ff6zjff4scicilx80whccml2s


提交就成功。
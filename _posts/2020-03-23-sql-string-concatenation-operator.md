---
layout: post
title: "[SQL] 字串連接的奇妙應用"
description: "[SQL] 字串連接的奇妙應用"
crawlertitle: "[SQL] 字串連接的奇妙應用"
date: 2020-03-23 23:59:59 +0800
categories: SQL
tags: SQL
comments: true
---

最近看到 SQL concatenation operator `||` 一個相當有趣的用法，透過 SQL 字串連接產生出任何其他我們想要的 SQL 語句，  
在這之前從沒想過可以這樣用，趕緊紀錄下來！

## 有禮貌運動 —— 和資料說你好
以下以 query & result 以 PostgresSQL 的 psql 示範，  
如果心血來潮突然想和資料表 `users` 裡面所有的 user 打招呼，該怎麼做？

首先看一下資料表裡面都有誰
```sql
SELECT title, name FROM users;
```

查詢結果有九人。
```sql
 title |     name
-------+--------------
       | Store Owner
       | Admin
 Mr.   | Steve
 Mr.   | Lin
 Mr.   | Giant
 Ms.   | Yu
 Ms.   | Jia
 Sir.  | Isaac Newton
 Ms.   | Ip
(9 rows)
```

使用字串連接符 (concatenation operator)，將搜尋結果串成一段句子：

```sql
SELECT 'Hello, ' || title || name || '!' FROM users;
```

> 你好我好大家好！
```sql
        ?column?
-------------------------


 Hello, Mr.Steve!
 Hello, Mr.Lin!
 Hello, Mr.Giant!
 Hello, Ms.Yu!
 Hello, Ms.Jia!
 Hello, Sir.Isaac Newton!
 Hello, Ms.Ip!
(9 rows)
```

有兩個人不見了，是在...  
Hello？

原來資料庫裡這兩人沒有 title （title 為 NULL），在 PostgreSQL 使用字串連接時有個限制是不能和 NULL 一起接，整串會爆炸。  
因為我工作上有用到 Oracle SQL 所以順帶一提，Oracle SQL 能夠接受字串連接的[**一部份**](http://www.sqlines.com/oracle-to-sql-server/string_concat)為 NULL：

**Oracle:**
```sql
SELECT 'The city' || ' is ' || NULL FROM dual;
-- Result: The city is
```

那回到 PostgresSQL 的這個情況下，可用的解法為 `COALESCE` 函式，裡面第一個參數如果等於 NULL，就回傳第二個參數作為替代，非常好懂。
這裡給空字串作為替代值：

```sql
SELECT 'Hello, ' || COALESCE(title, '') || COALESCE(name, '') || '!' FROM users;
```

就能把所有人都抓出來一個個問好。

```
        ?column?
-------------------------
 Hello, Store Owner!
 Hello, Admin!
 Hello, Mr.Steve!
 Hello, Mr.Lin!
 Hello, Mr.Giant!
 Hello, Ms.Yu!
 Hello, Ms.Jia!
 Hello, Sir.Isaac Newton!
 Hello, Ms.Ip!
(9 rows)
```

## 應用

現在我們暸解透過字串連接的方法，可將 `SELECT` 查詢的結果隨意組裝成字串輸出，那在實務上就有了很多能夠發揮的空間，  
例如以下例子能夠生成 `INSERT INTO...` 語句，將 1 號店家 (store_id = 1) 的 items，包含名稱和價格也都新增一份到 2 號店家，  
接著只要複製貼上產出來的語句，用這種方式能簡單快速地在資料庫中建立相關的記錄，在建立測試資料的時候特別好用！

```sql
SELECT 'INSERT INTO items (name, price, store_id, created_at, updated_at) VALUES ('''||name||''', '||price||', 2, to_date(''20200323'', ''yyyymmdd''), to_date(''20200323'', ''yyyymmdd''));'
  FROM items
 WHERE store_id = 1;

-- Result
                                                                              ?column?
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 INSERT INTO items (name, price, store_id, created_at, updated_at) VALUES ('test_item1', 399.0, 2, to_date('20200323', 'yyyymmdd'), to_date('20200323', 'yyyymmdd'));
 INSERT INTO items (name, price, store_id, created_at, updated_at) VALUES ('test_item2', 499.0, 2, to_date('20200323', 'yyyymmdd'), to_date('20200323', 'yyyymmdd'));
 INSERT INTO items (name, price, store_id, created_at, updated_at) VALUES ('test_item3', 199.0, 2, to_date('20200323', 'yyyymmdd'), to_date('20200323', 'yyyymmdd'));
```
> 使用兩個相鄰的單引號 `''` 以跳脫字串中的單引號。

掌握這個技巧之後就能配合各種情況自由發揮囉！

## References
- [|| String Concatenation Operator - Oracle to SQL Server Migration](http://www.sqlines.com/oracle-to-sql-server/string_concat)
- [PostgreSQL documentation (4.1.2.1. String Constants)](http://www.postgresql.org/docs/current/static/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS)
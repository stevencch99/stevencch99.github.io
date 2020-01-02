---
layout: post
title: "ActiveRecord & SQL find where not and null"
description: "ActiveRecord & SQL find where not and null"
crawlertitle: "ActiveRecord & SQL find where not and null"
date: 2019-11-07 23:50:50 +0800
categories: ActiveRecord
tags: Rails ActiveRecord SQL
comments: true
---

## 目錄

- toc
{:toc}

---

## 查詢的資料中含有空值 Null

現有一個 `credit_cards` 資料表，想找出符合下面幾個條件的資料：

1. payment 沒有值
2. 或者 check_date 沒有值
3. 排除 source = 'TEST' 的資料

In `credit_cards` table:

|  id   | status | bankno | payment | check_date | source |
| :---: | :----: | :----: | :-----: | :--------: | :----: |
|   1   |  'V'   |  123   |  null   |    null    | STORE  |
|   2   |  'V'   |  123   |  null   |    null    |  null  |
|   3   |  'V'   |  123   |  null   |    null    | PHONE  |
|   4   |  'V'   |  123   |   'A'   |  20191122  |  TEST  |
|   5   |  'V'   |  123   |   'A'   |  20191122  | STORE  |

原本嘗試透過這段查詢搜尋，但結果不盡人意：

```ruby
# Option 1
CreditCard.where('payment IS NULL OR check_date IS NULL AND source != ?', 'TEST')

# option 2 - Find by ActiveRecord query:
CreditCard.where(payment: nil)
          .or(CreditCard.where(payment: nil))
          .where.not(source: 'TEST')
```

很快地發現**第二列資料**並不會被找出來，這並不是我們要的。

因為資料中有些記錄的 source 還沒有值，本以為透過 `source != TEST` 這個條件下搜尋也能比對出結果，看來案情不單純。

查了一下發現 SQL 對待空值和對待其他值的處理方式不同，所以就不能直接拿來比對字串，
在這裡我們必須指名道姓的對資料庫說要 `Null` 出來面對(IS)，故修改最終查詢子句如下：

```ruby
# Option 1
CreditCard.where('payment IS NULL OR check_date IS NULL AND source != ? OR source IS NULL', 'TEST')

# option 2 - Find by ActiveRecord query:
CreditCard.where(payment: nil)
          .or(CreditCard.where(payment: nil))
          .where.not(source: 'TEST')
          .or(CreditCard.where(source: nil))
```

## IS NULL

Null（空值）是結構化查詢語言 SQL 中使用的特殊標記，用於表示資料庫中不具值，它不是一個值，而應該說是一個『**狀態**』，用作資料庫欄位的預設值、資料缺失或不可用。

正因為 Null 不是數據，而是一種資料缺失的標記，所以用於算術運算將會得出未知結果。

查詢時的 `WHERE` 條件式要判斷欄位是否為 Null 時，用 `WHERE payment = NULL` 是沒有用的，必須寫成 `WHERE payment IS NULL`，同理，前面例子中 `source != TEST` 也是因為 Null 在運算中被資料庫忽略而沒有達到預期的效果，所以必須再串一個針對 Null 的 `OR` 查詢子句。

## 參考

- [空值 (SQL)](https://zh.wikipedia.org/wiki/%E7%A9%BA%E5%80%BC_(SQL)) － 維基百科
- 《圖解資料庫系統理論：使用 MySQL 實作》 － 李春雄

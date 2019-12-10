---
layout: post
title: "ActiveRecord - 資料庫操作抽象化和資料表的基礎"
description: "ActiveRecord - 資料庫操作抽象化和資料表的基礎"
crawlertitle: "ActiveRecord - 資料庫操作抽象化和資料表的基礎"
date: 2019-08-28 01:10:46 +0800
categories: Rails
tags: Rails ActiveRecord
comments: true
---

## 目錄

- toc
{:toc}

## 物件導向 X 關聯式資料庫 X 抽象化

Active Record 是 Rails 配備的物件－關係映射（Object-relational mapping, ORM）層，將關聯式資料庫(Relational database)和物件導向的程式語言兜在一塊兒，
它將程式與資料庫交流的過程包裝、抽象化，讓我們可以像操作物件一樣操作資料，簡化原本相對繁瑣的 SQL 敘述，提升工程師的幸福指數。

自從利用物件導向模式包裝關聯式資料庫的方法問世之後，其抽象程度到底能深入到何處也是大家探究的主題。
某些論點認為 ORM 工具應該向純粹物件導向精神靠攏，希望完全避免使用 SQL 敘述並強制採用 OO 層的方式進行查詢或操作，其他一些觀點則認為 Active Record 模式根本上就違反了 Single Responsibility Principle 等原則，和許多傳統開發方法相抵觸。

而**抽象化**本身：把複雜動作簡化的這個機制也是有漏洞的 ，這邊引用《[The Joel on Software Translation Project:抽象滲漏法則](http://local.joelonsoftware.com/wiki/The_Joel_on_Software_Translation_Project:%E6%8A%BD%E8%B1%A1%E6%BB%B2%E6%BC%8F%E6%B3%95%E5%89%87)》的其中一個段落：

> "SQL 語言希望把資料庫查詢的程序抽象化，讓你只要定義想要的東西，查詢動作的細節就交由資料庫去處理。不過在某些狀況下，有些 SQL 查詢比邏輯上相等的查詢慢上幾千倍。這有個很有名的例子，在某個 SQL 伺服器用 `"where a=b and b=c and a=c"` 來查詢，會比用 `"where a=b and b=c"` 快上許多，可是查詢的結果其實是一樣的。照道理只要指定規格，並不需要在意程序。可是有時候抽象機制會失效並導致很差的效率，於是你就得跳出來用查詢規劃分析器找出問題，然後想辦法加快查詢。"

我們終究得掌握一定程度 SQL 的知識才有辦法處理諸如此類抽象滲漏的狀況，更重要是**好好享受** Active Record 提供的物件導向介面來保證開發的效率和工作的樂趣，然後在需要的時候深入資料庫內部以追求貼近本質的體驗。

順道一提，關聯式資料庫是建立在關聯模型（Relationship Model）基礎上的資料庫，而一般用來設計資料庫、探討抽象資料架構，則常用實體關聯模型（Entity-Relationship Model, ER Model），這可是我們台灣台中人，[陳品山](https://zh.wikipedia.org/wiki/%E9%99%88%E5%93%81%E5%B1%B1)博士發明的好貨，可以幫助初學者更容易理解資料庫設計的基本方法，有助於設計過程的構思和討論溝通。

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Active Record 模型

Active Record 採用標準的 ORM 模型：資料表對應到類別，記錄對應到物件，欄位則映射物件屬性。

使用 Active Record 將訂單資料表包裝到 MySQL 資料庫，按照 id 找到訂單紀錄之後，修改買主的名字，然後將結果存回資料庫，更新原本的紀錄。

> 範例取自 **《碼上就會：Rails 敏捷開發網站》**

程式長得像這樣：

```ruby
require     'rubygems'
require_gem 'activerecord'

ActiveRecord::Base.establish_connection(adapter: 'mysql',
  host: 'localhost', database: 'railsdb')

class Order < ActiveRecord::Base
end

order = Order.find(123)
order.name = 'Steven Chang'
order.save
```

就這樣，除了資料庫連結的部分之外不需要任何其他的配置，Active Record 都幫我們妥妥的處理掉了。

### 資料表和類別

在建立 ActiveRecord::Base 的一個子類別時，就是在透過建立某種實體將資料表包裝起來。
慣例上資料表名稱是類別名稱的複數型。

| 類別名稱 | 資料表 |
| :------- | :----- |
| Order    | orders |
| Store    | stores |
| Cat      | cats   |
| Person   | people |

當然為了某些原因，例如配合舊專案不同的命名慣例，也可以自訂資料表名稱：

```ruby
class Cat < ActiveRecord:Base
  self.table_name = 'kitty'
end
```

## 欄位和屬性

### 存取紀錄和屬性

Active Record 類別對應資料表，類別的實體物件對應資料表中的各個紀錄，紀錄裡面的欄位則對應到物件的屬性。

呼叫 `Store.find(1)` 會傳回 `Store` 類別的一個實體，這個實體包含了主鍵(Primary Key)為 1 的紀錄中的資料。

![Call Store.find(1) in rails console](https://i.imgur.com/O7vibSN.png)

每個資料表可以有許多紀錄，每筆紀錄包含了各種欄位。
![Database table structure](https://i.imgur.com/r37E6rH.png)

Rails 會自動建構屬性的存取器，簡化取值和設定的過程，假設我們另外有一個 Cat 類別：

```ruby
class Cat
  attr_accessor :name, :age
  def initialize(name, age)
    self.name = name
    self.age = age
  end
end

cat1 = Cat.new('Kitty', 3)
cat2 = Cat.new('Diddy', 1)
p cat1.name # Kitty
p cat2.age  # 1
```

### Boolean 布林值屬性

Active Record 在從資料庫讀取屬性值的時候會將之轉為適當的 Ruby 資料型別，對 Ruby 來說，除了 `nil` 和 `false` 其他都是 `true`。

而問題就在於，有時候開發者使用數字 `0` 或者 `f` 代表 `false` 這在 Ruby 語言會被轉譯為 `true`

- 如果要查詢一個 boolean 型欄位的狀態值  
  必須在欄位名稱後面加上 `?`，此舉將會查看欄位的值，將數字 `0`，字串 `'0'`、`'f'`、`'false'`、`''`(空字串)、nil 或者常數 `false` 都解釋成 `false`。

```ruby
user = User.find_by_name("Steven")

if user.superuser?
  grant_privileges
end
```

### 屬性的原始值
如果想得到一個屬性的**原始值**，可以在屬性名稱後面加上 `_before_type_cast`：

```ruby
# 讀取記錄建立的時間
store.created_at
#=> Wed, 12 Jun 2019 21:14:24 CST +08:00

store.created_at_before_type_cast
#=> '2019-06-12 13:14:24.905577'
```

![Output of _before_type_cast method](https://i.imgur.com/XmMaWuC.jpg)

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## 參考

- [The Joel on Software Translation Project:抽象滲漏法則](http://local.joelonsoftware.com/wiki/The_Joel_on_Software_Translation_Project:%E6%8A%BD%E8%B1%A1%E6%BB%B2%E6%BC%8F%E6%B3%95%E5%89%87)
- [ActiveRecord - 基本操作與關聯設計](https://ihower.tw/rails/activerecord.html)
- 《碼上就會：Rails 敏捷開發網站》 - 碁峰資訊

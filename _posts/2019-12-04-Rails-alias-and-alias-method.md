---
layout: post
title: "Rails - alias & alias_mehtod 的用法和區別"
description: "Rails - alias & alias_mehtod 的用法和區別"
crawlertitle: "Rails - alias & alias_mehtod 的用法和區別"
date: 2019-12-04 23:50:46 +0800
categories: Rails
tags: ['Rails']
comments: true
---

## 目錄

- toc
{:toc}

---
## Rails alias & alias_mehtod 的用法和區別

最近在工作上看到類似像這樣的寫法：

```ruby
# Under instance project
class Order
  # alias Common method :foo to call :foo under instance project
  alias_method :foo_common, :foo

  def foo(order)
    res = foo_common(order)
    calc_attr_price(order, res.attr) if res.attr.present?

    res
  end
end
```

簡單介紹我們的程式架構是由一個共用的 Common 程式為基礎，根據不同的專案需求另外擴充 Instance module。

在這個例子中，為了因應不同國家的條件計算附加費用，  
於是透過 `alias_method`，在 Instance module 的 `:foo` 方法中，先呼叫位於 Common 的 `:foo` 方法(*這邊化名為 `:foo_common`*)之後，再接著執行 `:calculate_tax_price` 計算附加費用並回傳結果。

也就是說，只有在 Instance module 中呼叫 `:foo` 將會另外再加計附加費用，
如此便可以在不改動原本 Common 程式碼的情況下得到我們所期望的最終金額。

![image alt](https://i.imgur.com/FhEdPBT.png "alias_method_common_instance_graph")  
> 結構示意圖

## 我該用 alias 或 alias_method ？

在 Ruby 中還有一個關鍵字：`alias` 常讓人搞不清楚他們適合怎樣的使用情境，  
這裡先從使用方法來看看：

- alias 使用方法

```ruby
class User
  def full_name
    puts "Steven Chang"
  end
  
  alias name full_name
end

User.new.name #=> Steven Chang
```

- alias_method 使用方法

```ruby
class User

  def full_name
    puts "Steven Chang"
  end

  # 在參數之間需要加上 ","
  alias_method :name, :full_name
end

User.new.name #=> Steven Chang
```

`alias_method` 可以接受 string 或 symbol 作為參數傳入，參數之間需要加上逗號分隔。

### 繼承的類和作用域

再來觀察兩者作用的範圍有何不同？  
首先是 `alias_method`:

```ruby
class User

  def full_name
    puts "Steven Chang"
  end

  def self.set_alias
    alias_method :name, :full_name
  end
end

class Developer < User
  def full_name
    puts "碼農"
  end

  set_alias
end

Developer.new.name #=> 碼農
```

可以看到，在 Developer 類別呼叫 `:full_name` 的別名 `:name`，會取得 Developer 中設定的 "碼農"，使用 `alias` 的情況則有些不同：

```ruby
class User

  def full_name
    puts "Steven Chang"
  end

  def self.set_alias
    # 在這邊改用了 alias 關鍵字
    alias :name :full_name
  end
end

class Developer < User
  def full_name
    puts "碼農"
  end
  set_alias
end

Developer.new.name #=> Steven Chang
```

當我們改用 `alias` 時，對 Developer 類別呼叫 `:name` 方法會取得它所繼承的 User 類別中 `:full_name` 設定的值，而非原本預期寫在 Developer 類別裡的 "碼農"。  


這就是 `alias` 作為 keyword 關鍵字的語彙範疇（*lexical scope*）特性，`alias` 所看到的 `self` 指向程式在分析讀取階段當下的 `self`，它只認當下所處的 scope，在這個例子就是 User 類別之中；  

而 `alias_method` 則作用在動態範疇（*dynamic scope*），它看到的 `self` 指向程式執行當下(*run time*)的 `self`，也就是執行 `Developer.new.name` 的 `:name` 方法指向的 Developer 實體。

## 小結

- 兩者都可以重新命名一個方法。
- 在子類別重寫(*override*)父類別的方法時，可以用來呼叫舊方法。

- 區別：
  - `alias`：
    - Keyword used by Ruby.
    - 會作用於關鍵字所在的類別。
    - 參數可以是方法名稱或符號，不含逗號。
  - `alias_method`：
    - Method defined in Module class.
    - 是一個模組方法，必要時也可以擴充功能，使用上比較有彈性。
    - 參數可以是陣列或符號，參數之間必須加逗號。

## 參考

- [alias vs alias_method](https://blog.bigbinary.com/2012/01/08/alias-vs-alias-method.html) By [Neeraj Singh](https://blog.bigbinary.com/authors/neerajdotname)

---
layout: post
title: "[Ruby] Clone, Dup 和 Shallow copy"
description: "[Ruby] Clone 和 Dup 的差異"
crawlertitle: "[Ruby] Clone, Dup 和 Shallow copy"
date: 2020-04-02 23:59:59 +0800
categories: RUBY
tags: RUBY
comments: true
---
在 Ruby 裡面所見幾乎任何東西都是物件 (Object)，為了要複製物件，常常我們會用到 `dup` 或者 `clone`，本篇記錄下這兩者在使用上的不同：

```ruby
a = *10.downto(1)
#=> [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]

b = a.clone
#=> [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]

c = a.dup
#=> [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```

從上面的結果看來，使用 `dup` 和 `clone` 似乎沒有區別，  
不去深究的話可能會誤以為和 `inject` & `reduce` 同樣是互為別名 (alias) 一般的存在，但這兩個複製物件的方法實際上是有差的。

## 內部狀態 (internal state)

其中一個差別在於 `clone` 方法在複製物件的同時，會一併複製物件完整的內部狀態 。

那麼物件有哪些可複製的內部狀態？
- frozen
- tainted
- singleton methods

**舉個例子：**
```ruby
a = Object.new.taint.freeze

b = a.clone
b.frozen?  #=> true
b.tainted? #=> true

c = a.dup
c.frozen?  #=> false
c.tainted? #=> true

# Ruby 2.4 以後，可傳入 keyword argument (freeze: false) 選擇不去複製 freeze 狀態。
d = a.clone(freeze: false)
d.frozen?  #=> false
```

可以看到 `dup` 方法在複製物件的時候無視了原物件 frozen 的狀態，讓新產生的物件可以再次被修改。而另一方面，使用 `clone` 方法的話，在 Ruby 2.4 以上版本要額外加入 keyword argument `freeze: false` 才能從一個被凍結的物件複製出解凍的副本。

## 模組 (module)

根據 [APIdock 對 dup 的描述](https://apidock.com/ruby/v2_1_10/Object/dup)，從子類別的角度來看待這兩個方法也有些語義上的差異，  
`clone` 常用來複製目標物件和它的內部狀態，`dup` 則比較多用在目標的類別底下創造一個新的實例。

進一步看，`dup` 方法除了會忽略前面提到原物件的 frozen 狀態之外，還會忽略原物件所延伸 (extend) 的模組們。

```ruby
class Klass
  attr_accessor :str
end

module Foo
  def foo; 'foo'; end
end

s1 = Klass.new
s1.extend(Foo)
s1.singleton_methods #=> [:foo]
s1.foo               #=> foo

s2 = s1.clone
s3.singleton_methods #=> [:foo]
s2.foo               #=> foo

s3 = s1.dup
s3.singleton_methods #=> []
s3.foo               #=> NoMethodError (undefined method `foo' for #<Klass:0x00007fe10710c878>)
```

# 淺層複製 (shallow copy)

這兩個複製方法同屬淺層複製，複製對象如果含有其他物件的參照，例如陣列中包含著字串物件，結果也只是複製那些字串的參照，結果會指向同一個物件，這點可以從完全相同的 Object Id 看出來：

```ruby
a = %w[app bee cake]
a.map(&:object_id)
#=> [70260548767440, 70260548767420, 70260548767400]

b = a.clone
b.map(&:object_id)
#=> [70260548767440, 70260548767420, 70260548767400]
```

在這個情況下如果修改了陣列 `b` 當中的字串內容，同時也會影響原始物件 `a` 當中的資料：

```ruby
a #=> ["app", "bee", "cake"]
b #=> ["app", "bee", "cake"]

b[0].capitalize! #=> "App"
b #=> ["App", "bee", "cake"]
a #=> ["App", "bee", "cake"]
```

以這個例子來說，如果想變更陣列中字串的內容又不影響到原陣列，還是重新指派值會比較安全：

```ruby
b[0] = 'Deep'
b #=> ["Deep", "bee", "cake"]
a #=> ["App", "bee", "cake"]
```

或者進行深層複製 (deep copy)，這可以透過 Ruby 的標準函式庫 [Marshal](https://ruby-doc.org/core-2.7.1/Marshal.html) 來處理：

```ruby
a = %w[app bee cake]
b = Marshal.load(Marshal.dump(a))

a.map(&:object_id)
#=> [70260548496640, 70260548496620, 70260548496460]
b.map(&:object_id)
#=> [70260548515100, 70260548515020, 70260548514900]

b.each(&:upcase!) #=> ["APP", "BEE", "CAKE"]
a                 #=> ["app", "bee", "cake"]
```

## 小結

使用上要注意到 `clone` 會複製目標對象的內部狀態和 `singleton_methods`，`dup` 只會複製對象本身的內容，且不含 frozen 狀態。此外，這兩個方法都只是淺層複製，如果一不小心直接變更複製的物件中所參照的其他物件，可能會導致意外地影響原始物件的內容，不可不慎。

## 參考資料
- [APIdock](https://apidock.com/ruby/v2_1_10/Object/dup)
- [RubyGuides](https://www.rubyguides.com/2018/11/dup-vs-clone/)
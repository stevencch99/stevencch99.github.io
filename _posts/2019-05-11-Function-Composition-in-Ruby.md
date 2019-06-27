---
layout: post
title:  "Ruby 中的複合函數：To Proc, or not to Proc"
crawlertitle: "About Ruby and Function compostition"
description: "Ruby 中的複合函數 proc lambda "
date:   2019-05-14 16:56:46 +0800
categories: Ruby
tags: ['Ruby']
comments: true
---
*Is that a question?*

> 本文同步刊錄在 [Medium](https://medium.com/@stevencch/to-proc-or-not-to-proc-2578c5e5c54c)

## What is function composition?

根據維基百科：
[Wikipedia describes function composition as follows:](https://en.wikipedia.org/wiki/Function_composition_(computer_science))

> In computer science, funciton composition is an act or mechanism to combine simple functions to build more complicated ones. Like the usual composition of functions in mathematics, the result of each function is passed as the argument of the next, and the result of the last one is the result of the whole.

像在數學中合成函數是把兩個函數鏈接在一起的過程，內函數的輸出就是外函數的輸入。

一個方法(Method, 或稱函式 Function)的回傳值作為另一個方法的參數(parameter)傳入，直到最後一個方法的輸出即是整個 Function composition 的輸出結果。

- Example function:

```ruby
# double the number
def double(x)
  x * 2
end

double(2) = 4
double(3) = 6
double(4) = 8

# squares the number
def square(x)
  x * x
end

square(2) = 4
square(3) = 9
square(4) = 16
```

同時因為這些方法只單純的計算傳入的引數(argument)，沒有其他副作用，故如果想要將一個數字先乘以二再平方，我們可以直接組合這兩個方法：

`square(double(2)) = 16`

為了方便呼叫，將上述操作指定給一個新的方法：

```ruby
double-then-square(x) = square(double(x))
double-then-square(2) # => 16
double-then-square(3) # => 36
```

在某些程式語言中有所謂『一級公民』 "first-class" 的概念，讓我們不必先宣告一個全新的方法才能來組合他們。

在 Functional programming 中大量用到一種系列的方法來處理問題，有了這個 first-class function composition 的概念，可以讓其更容易的將某些肥厚的方法抽象化分離成更小的元件。

## Functions in Ruby

**如上所述，我們也需要一種方法**，可以當作引數傳給其他方法、可以儲存變數或資料結構，甚至需要它作為回傳值從其他方法中 return。

換言之，我們需要 "first-class functions" 來做到這件事。
其中一個明確的例子就是 Proc。

### Proc

根據 Ruby documentation 的描述：

> A Proc object is an encapsulation of a block of code, which can be stored in a local variable, passed to a method or another Proc, and can be called. Proc is an essential concept in Ruby and a core of its functional programming features.

Ruby 作為一個物件化相當徹底的語言，所見的幾乎所有東西都是物件，包含數字等等，除了其中一個例外："block"。

Block 是一小段程式碼區塊，既沒有自己的名字亦無法單獨存在，很可憐。
平常像寄生蟲一樣必須得依附在一個宿主身上，由宿主的行為決定要不要執行它，如果沒人操作它便會沈入記憶體之海。

這時可透過創造一個 Proc 物件把這段程式碼區塊承接下來（物件化），等待適合的時機呼叫它。

```ruby
# Use the Proc class constructor
double = Proc.new { |number| number * 2 }

# Use the Kernel#proc method as a shorthand
double = proc { |number| number * 2 }

# Receive a block of code as an argument (note the &)
def make_proc(&block)
  block
end

double = make_proc { |number| number * 2 }
```

- 或者透過 `Proc.new` 去承接傳入的 block
[Passing Blocks in Ruby Without &block](https://mudge.name/2011/01/26/passing-blocks-in-ruby-without-block.html)

```ruby
# Use Proc.new to capture a block passed to a method without an ecplicit block argument
def make_proc
  Proc.new
end

double = make_proc { |number| number * 2}
```

呼叫 Proc 物件有以下幾種方法：

```ruby
double.call(2) # => 4
double.(2)     # => 4
double[2]      # => 4
# shorthand of double.===(2)
double === 2   # => 4
```

值得注意的是第四個呼叫的方法，因為 Case statement 在執行判斷比較條件的時候也是透過 "===" 關聯運算子(relationship operator)同時檢查複數的條件，所以在這種情況特別有用。

以經典的 Fizz Buzz 題目舉例：

```ruby
# 定義一個 Porc 物件把 block 包起來，當呼叫的時候會在裡面做運算並回傳布林值(true or false)
divisible_by_15 = proc { |number| (number % 15).zero? }
divisible_by_5 = proc { |number| (number % 5).zero? }
divisible_by_3 = proc { |number| (number % 3).zero? }

num = 9

case num
# when 會呼叫 divisible_by_15 的 === 方法，並把 num 當引數傳進去
# divisible_by_15 === 9
when divisible_by_15
  puts "FizzBuzz"
when divisible_by_5
  puts "Buzz"
when divisible_by_3
  puts "Fizz"
else
  puts num
end
# Fizz
```

### Lambda

和 Proc 長得很像，用法也很像，兩者僅有微小的差別：

- Lambda 比較類似 function ，對於傳入的引數數量很嚴格。
- 呼叫 `return` 會回傳結果然後脫離 lambda 。

```ruby
double = lambda { |number| number * 2 }
double = -> (number) { return number * 2 }
double[2] # => 4
double[2, 3] # => ArgumentError (wrong number of arguments (given 2, expected 1))
```

- Proc 的表現如同`block`，對傳入的引數並不介意，在裡面呼叫 return 會爆炸

```ruby
double_p = proc { |number| return number * 2}
double_p[2] # => LocalJumpError (unexpected return)
```

### Method

在 Ruby 我們能用上的另一個特色就是透過 Method 將方法包成物件(http://ruby-doc.org/core-2.6.3/Method.html)。

```ruby
class Greeter
  attr_reader :greeting

  def initialize(greeting)
    @greeting = greeting
  end

  def greet(subject)
    "#{greeting}, #{subject}!"
  end
end

greeter = Greeter.new("Hello")
greet = greeter.method(:greet)
```

也一樣可以透過呼叫 Proc 的方法使用它：

```ruby
p greet.call("world") # => "Hello, world!"
p greet.("world")     # => "Hello, world!"
p greet["world"]      # => "Hello, world!"
p greet === "world"   # => "Hello, world!"
```

## To Proc or not to Proc:

透過 `to_proc` 可以將一個方法轉為 Proc 物件：

```ruby
def my_even?
  :even?.to_proc
end

my_even?[3] # => false
```

另外我們可以透過 `&` 將方法作為 Proc 物件傳給另一個方法，這也是我最愛的功能！

```ruby
# 列出 1 到 10 的偶數陣列
list = 1.upto(10).select(&:even?)
# 此式等價於：
# list = 1.upto(10).select(&:even.to_proc)

list # [2, 4, 6, 8, 10]
```

Ruby 提供了 `>>`(forward composition) 和 `<<`(backward composition) 方法將各種 Proc 組合起來，可以很直覺的看作是執行的順序，熟悉了之後就可以這樣用：

```ruby
double_then_square = square << double
double_then_square[2] # => 16

square_then_double = square >> double
square_then_double[2] # => 8
(square >> double)[2] # => 8

``` 

```ruby
to_camel = :capitalize.to_proc
add_header = ->val {"Title: " + val}
strip_space = :strip.to_proc

format_as_title = add_header << to_camel << strip_space

format_as_title["  \thello world\n"]
# "Title: Hello world"
```

## 參考資料：

- [A Guide to Function Composition in Ruby:](https://www.ghostcassette.com/function-composition-in-ruby/)
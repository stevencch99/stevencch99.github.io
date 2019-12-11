---
layout: post
title: "ActiveRecord - CRUD: Creat, Read, Update, Delete"
description: "ActiveRecord - CRUD: Creat, Read, Update, Delete"
crawlertitle: "ActiveRecord - CRUD: Creat, Read, Update, Delete"
date: 2019-09-09 01:10:46 +0800
categories: Rails
tags: Rails ActiveRecord
comments: true
---

## 目錄

- toc
{:toc}

## CRUD - Creat, Read, Update, Delete

在前一篇：[ActiveRecord 資料庫操作抽象化和資料表的基礎]({{ site.baseurl }}{% link _posts/2019-08-28-Active-Record-the-basic-and-the-abstraction-of-database-operation.md %}) 中提到過，Active Record 讓資料庫的操作變得輕鬆，本篇介紹最基本的四個資料庫操作：新增、讀取、更新和刪除 。

假設我們已經設定了一個 Order 模型，對應到 orders 資料表：

```ruby
class Order < ApplicationRecord
end
```

### Create 建立物件

呼叫 `Order.new` 方法就可以建立代表 `orders` 資料表中的新紀錄物件，
接著可以填入對應於資料庫欄位的屬性值，最後再呼叫 `save` 方法將資料寫入資料。

```ruby
an_order       = Order.new
an_order.name  = 'Steven'
an_order.email = 'steven@example.com'
an_order.save
```

> 如果沒有呼叫 `save` 方法，資料就只會存在本地記憶體，不會進入資料庫。

Active Record 建構函數可以接受一段程式碼作為選擇性參數。

```ruby
Order.new do |o|
  o.name = 'Steven'
  # ...
  o.save
end
```

還可以接收對應屬性名稱的 hash 作為選擇性參數，這招在接收 HTML 表單內容存入資料庫時特別有用。

```ruby
an_order = Order.new(
  name: 'Steven',
  email: 'steven@example.com'
)
an_order.save

# controller
def store_params
  params.require(:store).permit(:name, :description, :tel)
end

def create
  @store = Store.new(store_params)
  @store.owner = current_user
  @store.save
end
```

成功儲存進資料庫之後 Active Record 會自動幫我們這筆資料加上 `id` 屬性。

- 使用 `create` 方法則可以同時完成 `new` 和 `save` 這兩個動作，這個方法可以接收一組屬性 hash 的陣列，一次建立數筆資料：

```ruby
orders = Order.create(
  [
    { name: 'Steven', email: 'steven@example.com' },
    { name: 'Chang', email: 'Chang@example.com' }
  ]
)
```

## Read 讀取現有紀錄

```ruby
an_order = Order.find(27)

# Get a list of product ids from a form, then
# sum the total price
product_list = params[:product_ids]
total = Product.find(product_list).sum(&:price)
```

### Finder method(deprecated)
Finder method 是比較舊的做法，但因為公司早期的專案(Rails 4.2.5)中有用到，所以也還是在這裡記錄使用方法，下一段落介紹取而代之的 `where` 方法（參見 Alternatives for [find](https://api.rubyonrails.org/classes/ActiveRecord/FinderMethods.html#method-i-find)）。

帶有第一個參數 `:first` 或者 `:all` 的 `find` 方法稱為「查詢方法 finder method」，如果沒有其他額外參數，
查詢方法會執行 `select from...` 敘述語句，`:all` 回傳符合條件的所有資料，`:first` 則是回傳一筆資料(不保證是第一筆)。

- 當資料庫沒有符合查詢條件的紀錄時：

  - Find Primary key 會得到 RecordNotFound error.  
    `Person.find(99)`
  - Find with conditions to return nil or `[]`.  
    `Person.find(:first, condition: "name = 'Steven'"`

- `find` 參數
  - `:first`: 回傳符合某條件的第一筆紀錄
  - `:all` : 回傳符合某條件的資料陣列

### SQL injection 注入攻擊

- **別這麼做！ DON'T DO THIS!!**

```ruby
# Get the limit amount from the from name = params[:name]
pos = Order.find(:all, conditions: "name = '#{name}' and pay_type = 'po'")
```

動態產生 SQL 的安全作法是讓 Active Record 代為處理這件事，凡是能夠傳入 SQL 字串的地方，也都能傳入 hash 或 array，讓 Active Record 建立轉譯過的 SQL 語句，防範注入攻擊。

- 更好的做法：**預留位置**
  在 SQL 語句中插入(多個)問號作為查詢參數的預留位置，第一個問號會被 array 第二個元素取代，以此類推，將前面的查詢改寫成以下：

```ruby
name = params[:name]

pos = Order.find(:all, conditions: "name = ? and pay_type = 'po'", name)
```

- 附帶命名的預留

```ruby
name     = params[:name]
pay_type = params[:pay_type]
pos      = Order.find(
                      :all,
                      conditions: [
                        "name = :name and pay_type = :pay_type",
                        {pay_type: pay_type, name: name}
                      ]
                    )
```

- `conditions` 參數可以指定傳遞到 `find` 方法所用的 SQL where 子句的條件。

  這個條件可以是：
    - 一個包含 SQL 的字串
    - 包含 SQL 和替換值的陣列
    - 直接傳入 hash

```ruby
pos = Order.find(:all,
                 conditions: [
                   "name = :name and pay_type = :pay_type",
                   params[:order]
                 ]
                )
```

- 更加簡潔的查詢：
  若只是將 hash 作為條件傳遞，Rails 會產生一個 `where` 子句，其中 hash 的 key 會用來對應欄位的名稱，value 對應匹配的值，所以這個查詢還可以縮寫：

```ruby
pos = Order.find(:all, conditions: params[:order])
pos.class #=> Array
```

因為 params 本來也是一個 hash，所以可以直接傳到查詢條件中，將前面的查詢更進一步地改寫：
```ruby
pos2 = Order.where(params[:order])
pos2.class #=> Order::ActiveRecord_Relation
```

### `where` 方法
`where` 是 Rails 3.0.0 以後新增的方法(參見[ActiveRecord](https://apidock.com/rails/ActiveRecord)::[QueryMethods](https://apidock.com/rails/ActiveRecord/QueryMethods))，它會回傳一個 [ActiveRecord](https://apidock.com/rails/ActiveRecord)::[Relation](https://apidock.com/rails/ActiveRecord/Relation) 物件，能夠像陣列一樣操作，也能繼續串接 `where` 等查詢方法，強大的非常。

```ruby
stevens_order = Order.find(:all, conditions: "name = 'Steven'")

stevens_order = Order.where("name = 'Steven'")

# 取得從前端傳來的參數中的姓名變數 :name

name = params[:name]
other_orders = Order.find(:all, conditions: ["name = ?", name])
other_orders = Order.where(name: name)

# 搜尋名字等於 :name 且付款方式等於 :pay_type 的訂單
more_orders = Order.find(:all,
                          conditions: [
                            "name = :name and pay_type = :pay_type",
                            params[:order]
                          ]
                        )

# 搜尋姓名包含 foo 且職稱包含 bar 的使用者
more_users = User.where("name like ? and title like ?",
                        '%foo%',
                        '%bar%')
```

### Order 順序

除非我們在查詢中明確添加 `order by` 子句，否則 SQL 查詢結果是不保證順序性的。

```ruby
# 先依照支付類型，再按出貨日期降冪排列(DESC)
orders = Order.find(:all,
                     conditions: "name = 'Steven'",
                     order: "pay_type, shipped_ad DESC")
```

### Limit 與 Offset

有時我們想要限制一次取出的回傳資料數量：

```ruby
orders = Order.find(:all,
                    conditions: "name = 'Steven'",
                    order: "pay_type, shipped_ad DESC",
                    # 取出 5 筆資料
                    limit: 5,
                    # 從第 11 筆開始
                    offset: 10)
```

### Select

預設 `find` 方法會從資料表中取出所有欄位的內容，相當於發出一個 `select * from...` 指令，而記錄中的欄位可能包含巨大的資料量，`select` 選項可以讓我們指定要載入的欄位：

```ruby
list = Talks.find(:all, select: "title, speaker, recorded_on")
```

## 透過聚合(aggregate)函數統計資訊
  我們可以透過以下運算取得欄位值的統計。

```ruby
average = Order.average(:amount)
max     = Order.maximum(:amount)
min     = Order.minimum(:amount)
total   = Order.sum(:amount)
number  = Order.count
```

這些運算對應到底層資料庫的聚合函數，但是它們以獨立於資料庫的方式運作，我們可以使用更通用的 `calculate` 方法套用資料庫的專用函數。

- 套用 MySQL 的 `std` 函數在 amount 欄位上計算標準差。
  ```ruby
  std_dev = Order.calculate(:std, :amount)
  ```

## Update 更新欄位資料

透過 `update` 和 `update_all` 方法同時**讀取**和**更新**資料。

向 `update` 方法傳入一個由 id 組成的陣列，以及一個由屬性 hash 組成的陣列，取出相對應的紀錄、更新指定的屬性，並回傳由模型物件組成的陣列。

```ruby
order = Order.update(12, name: 'Steve', email: 'steve@email.com')

update_result = Product.update_all("price = 1.1 * price",
                                   "title like '%java%'")
```

`update_attributes` 是控制器 action 中最常用的方法，它會把表單中的資料合併到現有資料庫紀錄中。

```ruby
def save_after_edit
  # 從資料庫中找出指定的紀錄
  order = Order.find(params[:id])

  # 更新這筆記錄的屬性值
  if order.update_attributes(params[:order])
    redirect_to action: :index
  else
    render action: :edit
  end
end
```

## Save, save!, create and create!

- 如果紀錄成功儲存

  - `save` 回傳 true；否則回傳 false。

    ```ruby
    if order.save
      # do something
    else
      # you decide what to do
    end
    ```

  - `save!` 回傳 true；否則拋出異常。

    ```ruby
    begin
      order.save!
    rescue
      # validation
    end
    ```

- 無論儲存是否成功，`create` 都會回傳 Active Record 物件，一切雲淡風輕，若要確認資料是否已經寫入，需要檢查用於驗證錯誤的物件。
- 你猜對了，`create!` 在儲存失敗時會拋出異常。

## Delete 刪除資料

- 紀錄的刪除有兩種方法：`delete`, `delete_all`。
  和 `update` 方法一樣，`delete` 方法可以接收一個 id 作為參數，也可以接受一個由 id 組成的陣列作為參數，`delete_all` 會刪除所有符合條件的紀錄，若無指定條件則會刪除所有紀錄。

  `delete` 方法會繞過 Active Record 的各種驗證和 call back methods，回傳被刪除的紀錄數，如果刪除之前紀錄已經不存在了，就不會拋出異常。

```ruby
Order.delete(123)
User.delete([2, 3, 4])
Product.delete_all(['price > ?', expensive_price])
```

- Active Record 類別等級的刪除也有兩種方法：`destroy`, `destroy_all`。
  這兩個方法會呼叫所有的 call back methods 和驗證，沒有回傳值。雖然速度上比 `delete` 慢，但如果為了確保在專案中定義的一些邏輯驗證和商業規則運作如常，最好使用 `destroy` 方法。

## 參考

- 《碼上就會：Rails 敏捷開發網站》 - 碁峰資訊

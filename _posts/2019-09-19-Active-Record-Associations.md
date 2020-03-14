---
layout: post
title: ActiveRecord - Associations 資料表間的關聯
description: ActiveRecord - Associations 資料表間的關聯
crawlertitle: ActiveRecord - Associations 資料表間的關聯
date: 2019-09-19 23:50:46 +0800
categories: Rails
tags: Rails ActiveRecord
comments: true
---

## 目錄

- toc
{:toc}


## 一對一關聯

例如訂單與發票的關係：

- 每份訂單 (orders) 最多對應一張發票 (invoices)。
透過對模型添加 `has_one` 和 `belongs_to` 宣告，在 Rails 中宣告此關聯性。

```ruby
# Invoice model
class Invoice < ActiveRecord::Base
  belongs_to :order
  # ...
end

# Order model
class Order < ActiveRecord::Base
  has_one :invoice
  # ...
end
```

> 如果模型的資料表包含外部鍵指向其他資料表的記錄，那這個模型就一定含有 `belongs_to` 宣告。

## 一對多關聯

例如一份訂單可以包含任意數量的訂單項目 (line items)，在資料庫中這些訂單項目紀錄中，都會包含一樣的外部鍵指向同一張訂單。

```ruby
# LineItem model
class LineItem < ActiveRecord::Base
  belongs_to :order
  # ...
end

# Order model
class Order < ActiveRecord::Base
  has_many :line_items
  # ...
end
```

## 多對多關聯

多對多也是一種常見的關聯，對於這種情況，SQL 的慣例是在這兩者間建立第三個資料表，這個資料表被稱為連接表 (join table)。

在這張表中儲存了相關聯紀錄的外鍵，按照慣例，Active Record 會認為這表的命名，應該是兩張目標表的名子**按照字母順序**連接起來的結果。

如下例子的 migration 檔：一篇文章 (article) 可以有很多標籤 (tags)，也可以透過一個標籤找到多篇相關的文章。

```ruby
def self.up
  create_table :articles do |t|
    t.column :title, :string
    # ...
  end

  create_table :tags do |t|
    t.column :name, :string
    # ...
  end

  create_table :articles_tags, id: false do |t|
    t.column :article_id, :integer
    t.column :tag_id,     :integer
  end

  # 加入索引 (index) 提升查詢效能，應對將來越來越大的資料量
  add_index :aritcles_tags, [:article_id, :tag_id], unique: true
  add_index :aritcles_tags, :article_id
end
```

因為文章和標籤 id 的組合已經是唯一不重複的了，所以在此設定連結表上不需要有主鍵欄位 (`id: false`)。  
接著在表內增加了兩個索引。其中，複合索引會建立一個可以同時透過兩個外鍵欄位執行資料庫搜尋的索引。  
以下兩組 SQL 敘述是等價的：

```sql
SELECT foo FROM bar WHERE article_id = 1 AND tag_id = 2
SELECT foo FROM bar WHERE tag_id = 2 AND article_id = 1
```

多數的資料庫也足夠聰明，知道按照 `article_id` 執行快速查詢，範例中 19 行的第二個索引就是為了完成這波操作而設定的。

最後在這兩個模型中加入 `has_and_belongs_to_many` 讓雙方宣告彼此之間的關係，純粹的關聯就建立起來了。

```ruby
class Article < ActiveRecord::Base
  has_and_belongs_to_many :tags
  # ...
end

class Tag < ActiveRecord::Base
  has_and_belongs_to_many :articles
  # ...
end
```
你可能會想建一個程式碼片段 (code snippet) 來呵護可憐的手指⋯⋯
推薦 [高見龍](https://kaochenlong.com/) 龍哥的教學影片：[(YouTube)自己動手做 Code Snippet](https://youtu.be/nw9PP94hwuM)

## 將模型當成連接表

純粹的連接表應該只包含一對外鍵欄位，當實際情況需要加入更多資料的時候，才特別建立模型物件來進行操作。

舉個例子，我們想要紀錄使用者讀完文章後給的評價 (rating) 或者閱讀時間，那麼有這一個中間表來作紀錄，想來會比較合適。

就叫它 **Reading** 吧：

```ruby
class Article < ActiveRecord::Base
  has_many :readings
  # 透過 readings table 找到對應的使用者
  has_many :users, through: :readings
  # ...
end

class User < ActiveRecord::Base
  has_many :readings
  has_many :articles, through: :readings
end

class Reading < ActiveRecord::Base
  belongs_to :article
  belongs_to :user
end
```

接著就可以類似像這樣使用：

```ruby
reading         = Reading.new
# 將 rating 設定為前端傳來的參數
reading.rating  = params[:rating]
reading.read_at = Time.now
reading.article = current_article
reading.user    = session[:user]
reading.save
```

更進一步，`has_many` 可以接收參數，透過 `source:` 選項指派 user 為 reader，我們還能建立關聯來找出快樂讀者們都有誰。  
我們接著來改寫 Article model：

```ruby
class Article < ActiveRecord::Base
  has_many :readings
  has_many :readers, through: :readings, source: :user
  has_many :happy_readers,
           # 增加了關聯查詢的條件
           -> { where('readings.rating >= ?', 4) },
           through: :readings,
           source: :user,
end
```

## 消除重複

可是如果一位讀者重複閱讀了三遍文章，那麼 `has_many through:` 回傳的結果將會包含三個同樣的讀者模型副本，
這應該不會是我們希望見到的結果，為了消除重複，有以下做法。

- 從 Active Record 層面處理
可以在 model 中添加 `scope` lambda 參數 `-> { distince }`，由 Active Record 撈到資料後幫我們篩選過濾。

  ```ruby
  class Article < ActiveRecord::Base
    has_many :readings
    has_many :readers, -> { distinct }, through: :readings, source: :user
  end
  ```

- 從資料庫層面處理
使用 `select:` 參數來替換掉原本 Active Record 預設轉譯 SQL 的 `SELECT` 子句。

  ```ruby
    class Article < ActiveRecord::Base
      has_many :readings
      has_many :readers, through: :readings, source: :user, select: "distinct users.*"
    end
  ```

## 擴充關聯 (association)

前面提到的關聯宣告 (`has_one`, `belongs_to`) 會建立模型物件之間的關聯敘述，面對實務上各式不同的商業邏輯和關聯需求，前面的範例已經示範如何透過 Reading Model 擴充關聯，列出該文章的快樂讀者們，另一種方法則是在引用這些物件的時候把它加入查詢條件中：

```ruby
user = User.find(123)
user.articles.where('rating >= ?', 3)
```

這方法的缺點就是會打亂模型物件的封裝，這時我們可以在 `has_many` 宣告加上程式碼區塊 (block)，達成同樣的查詢。

```ruby
class User < ActiveRecord::Base
  has_many :readings
  has_many :articles, through: :readings do
    def rated_at_or_above(rating)
      where('rating >= ?', rating)
    end
  end
end
```

給使用者模型物件新增一個查詢方法，把查詢條件封裝在這裡，接著就可以從外部呼叫這個方法來查詢此用戶評價 4 顆星以上的文章：

```ruby
user = User.find(123)
good_articles = user.articles.rated_at_or_above(4)
```

> 所有的關聯宣告都適用這個方法

### 擴充共用搜尋條件模組

```ruby
# rating_finder.rb
module RatingFinder
  def rated_at_or_above(rating)
    where('rating >= ?', rating)
  end
end

# User.rb
class User < ActieveRecord::Base
  has_many :articles, extend: [RatingFinder, DateRangeFinder]
  has_may :articles, -> { extending FindRecentExtension }
end
```

## 參考

- [Need two indexes on a HABTM join table?](https://stackoverflow.com/questions/15210639/need-two-indexes-on-a-habtm-join-table)
- [(YouTube)自己動手做 Code Snippet](https://youtu.be/nw9PP94hwuM) - [高見龍](https://kaochenlong.com/)
- [Ruby on Rails API: Associations/ClassMethods](https://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html)

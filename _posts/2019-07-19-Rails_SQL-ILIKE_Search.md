---
layout: post
title: |
  2 步驟完成模糊搜尋 Search for whatever I LIKE in Rails
description: "2 步驟完成模糊搜尋 Rails search for PostgreSQL ILIKE syntax"
crawlertitle: "2 步驟完成模糊搜尋 Rails search for PostgreSQL ILIKE syntax"
date: 2019-07-19 18:22:46 +0800
categories: Rails
tags: Rails Database
comments: true
---

## 本篇摘要
在對資料庫做查詢時，為了達成模式比對（關鍵字查詢），我們會用到語法中的 `LIKE` 和 `ILIKE` 運算子，
`LIKE` 是標準的 SQL 語法，`ILIKE` 則是 PostgreSQL 額外提供的實用功能，能夠忽略英文大小寫。

接著再搭配萬用字元(wildcard)組成相似條件查詢語句，如此便完成模糊搜尋功能。

**萬用字元：**
  - `%` （百分比符號）代表零個或一個以上任意字元。
  - `_` （底線符號）代表單一個數的任意字元。

---

## 實作

### Step 1. 透過 View helper 在畫面中加入搜尋列：
`index.html.erb`

```html
<!-- 製作搜尋欄，使用 get 方法將資料傳回伺服器 -->
<%= form_tag stores_path, method: 'get' do %>
  <!-- 將使用者輸入的文字打包成 params[:search] 參數-->
  <%= text_field_tag :search, params[:search], placeholder: '搜尋' %>
<% end %>
```

### Step 2. 在對應的 controller 加入搜尋方法
```ruby
  def index
    # 透過 private method "list_stores" 取得店家列表並指定給實體變數 @stores
    @stores = list_stores(search: params[:search])
    render 'index'
  end

  private

  # 這邊用到 Keyword arguments 技巧，讓此方法更容易被理解用途和內含的參數
  def list_stores(search: nil)
    # 如果沒有輸入查詢值，則列出所有店家並跳出此方法
    return Store.all if search.nil?

    # 根據收到的參數內容 params[:search] 進行關鍵字查詢
    Store.where('name ILIKE ?', "%#{params[:search]}%")
  end
```
That's it!

## 補充
### 反向查詢

模糊搜尋同樣也能冠上 `NOT` 做反向查詢：
```ruby
Store.where('name NOT ILIKE ?', "%#{params[:search]}%")
```

### 查詢包含萬用字元的資料：

  單純地查詢 `%` 將會列出全部資料，如果要查詢的資料包含萬用字元 `%`, `_` 必需加上反斜線。

  **查詢名字包含 "%" 的店家：**

  ```sql
  $ SELECT * FROM stores WHERE name LIKE `\%`;
  ```

### 縮寫:
另外可以透過縮寫再少打幾個字，例如：

  ```sql
  $ SELECT * FROM stores WHERE name ~~ `\%`;
  ```

好不好用就見仁見智囉。

  | 縮寫 | 取代 |
  | :-: | :- |
  |`~~` | `LIKE`|
  |`!~~` | `NOT LIKE`|
  |`~~*` | `ILIKE`|
  |`!~~*` | `NOT ILIKE`|

> ## 打完收工。
---
layout: post
title:  "AJAX on Rails 透過 remote: true 實作 AJAX 按鈕"
description: "AJAX on Rails 透過 remote: true 實作 AJAX 按鈕"
crawlertitle: "AJAX on Rails 透過 remote: true 實作 AJAX 按鈕"
date:   2019-07-11 11:50:46 +0800
categories: Rails
tags: Rails
comments: true
---

在[什麼是 AJAX?]({{ site.baseurl }}{% link _posts/2019-07-10-What_is_ajax.md %}) 一文中，已大略介紹過 AJAX 和其應用場景，
本篇記錄如何在 Rails 專案中實作 AJAX 按鈕。

---
## 流程

AJAX 按鈕在 Rails 中的動作流程是：
- 點擊某連結
- 向伺服器請求一段 JavaScript 腳本
- 由客戶端瀏覽器執行


使用情境：對訂單進行操作後，透過部分渲染即時改變畫面中顯示的訂單狀態。

- View 的部分：在 html.erb 設定觸發連結
- 編輯對應的 controller
- 準備要 partial render 的 html.erb
- 編輯對應的 js.erb

## 實作

### Step 1. 編輯 View 設定觸發的連結：  
編輯 `app/views/admin/transaction/index.html.erb`，在迴圈內的 `link_to` 按鈕加入 `remote: true` 參數：  
```ruby
<%= link_to '接受', accept_admin_transaction_path(transaction), method: 'patch', remote: true, class: 'btn btn-success mr-2' if t.may_accept? %>
```

### Step 2. 編輯 controller
編輯 `app/controllsers/admin/transactions_controller.rb`  

```ruby
def accept
  # 將來可以把這行整理到 before_action 內
  @transaction = Transaction.find(params[:id])

  # 改變訂單狀態
  @transaction.accept!
  # 呼叫 state_change.js.erb
  render 'state_change'
end
```

### Step 3. 準備要 partial render 的 html.erb  

`app/views/admin/transactions/_orders__card--body.html.erb`

```ruby
<div class='orders__card--info'>
  <p>訂單編號：<%= transaction.serial_number %> ｜ 訂單狀態：<%= transaction.state %></p>
</div>
# 以下省略
```
### Step 4. 編輯對應的 js.erb 
編輯 `app/views/admin/transactions/state_change.js.erb`
- 選取畫面上對應的 html 元素，將內容替換為 `orders__card--body.html.erb` 的內容
- `orders__card--body.html.erb` 接受一個參數 `transaction`，於這時將 `@transaction` 傳進去

```js
$('#order_<%= @transaction.id %>').html('<%= j render "orders__card--body", transaction: @transaction %>')
```
> 為了跳脫引號的束縛，這遍必須加入 escape_javascript(), 簡寫為 j()

如此便完成了一次流程，如果要除錯也是按照流程去確認各元件是否都正常運作：  
view => controller => `js.erb` => `_partial.html.erb`
## 參考
- [Rails為何要使用escape_javascript?](http://noelsaga.herokuapp.com/blog/2015/09/25/railswei-he-yao-shi-yong-escape-javascript)
- [Ajax 應用程式](https://ihower.tw/rails/ajax.html)
- Demo Project on GitHub: [Order Brother](https://github.com/order-brother/Order-Brother)

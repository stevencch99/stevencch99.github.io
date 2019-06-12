---
layout: post
title:  "透過計數快取 Counter Cache 提升效能"
crawlertitle: "To optimize website performance by counter cache."
date:   2019-06-09 16:56:46 +0800
categories: Rails
tags: ['Rails']
comments: true
---
在許多場合需要統計 has_many 一對多關聯的資料數量，若使用單純的 Rails 關聯查詢，則每次讀取頁面都會重新再統計一次，影響網站效能。

## 情境

專案`Project` model has many 任務`Task`.

`Project` model
```ruby
class Project < ApplicationRecord
  has_many :tasks
end
```
`projects_controller.rb`
```ruby
def index
  @projects = Project.all
end
```

`views/projects/index.html.erb`
```ruby
  <% @projects.each do |project| %>
    <tr>
      <td><%= project.name %>(<%= project.tasks.size %>)</td>
    </tr>
  <% end %>
```
讀取頁面時就會出現 N+1 Query 問題，1 個專案若有 3 個任務便會做 3 + 1 次查詢：`SELECT “tasks”.* FROM “tasks” WHERE “tasks”.”project_id” = ?`

![N + 1 Query](https://cdn-images-1.medium.com/max/2788/1*g0xa5I3DTjz0qCuKYo3sFw.jpeg)

## 使用方法

在 projects table 中建立 tasks_count 欄位，將計數存在其中，
更新專案的任務數量時，counter cache 會同步更新到對應的欄位裡面。

### 建立 migration

`$ rails g migration add_tasks_count`

編輯剛建立的 migration 檔：
> *在程式碼第一行加入 # frozen_string_literal: true 凍結此檔，可以稍微加快往後的讀取速度。*

```ruby
# frozen_string_literal: true

class AddTasksCount < ActiveRecord::Migration[5.2]
  def change
    add_column :projects, :tasks_count, :integer, default: 0
  end
end
```

執行 `$ rails db:migrate`

### 編輯 Task model

開啟 counter cache:

```ruby
class Task < ApplicationRecord
  belongs_to :project, counter_cache: true
end
```

如果想指定其他欄位儲存 counter cache 請改寫為
`counter_cache: 'COLUMN_NAME'`

![指定 vote 欄位作為紀錄用](https://cdn-images-1.medium.com/max/2000/1*mofqtYMzoIQMsLZFC_10tg.jpeg)

確認資料表內容

`schema.rb`

```ruby
create_table "projects", force: :cascade do |t|
  t.string "name"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
  t.integer "tasks_count", default: 0
end

create_table "tasks", force: :cascade do |t|
  t.string "name"
  t.integer "project_id"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
  t.index ["project_id"], name: "index_tasks_on_project_id"
end
```

### 建立 rake task 更新 counter cache

更新既有 projects 的 tasks count 
`lib/tasks/update_tasks_count.rb`

> 這邊使用我在[五倍紅寶石](https://5xruby.tw/)學到的技巧，將這段 rake task 整理在 namespace db 之下。

```ruby
namespace :db do
  desc 'Reset counter cache of tasks count in project.'
  task update_tasks: :environment do
    print '開始更新任務數'
    Project.all.each do |p|
      Project.reset_counters(p.id, :tasks)
      print '.'
    end
    puts '更新完成！'
  end
end
```

查看指令確認一下沒有寫錯：`$ rails -T`

![$ rails -T](https://cdn-images-1.medium.com/max/2000/1*FWZtEUGwVe3jbrVIirNryQ.jpeg)

執行 `rails db:updste_tasks`

![$ rails db:updste_tasks](https://cdn-images-1.medium.com/max/2000/1*rPr9CH7A9xenQActw0cPIw.jpeg)

### 重啟伺服器

再次讀取頁面查看 SQL queries 只剩一行： `SELECT “projects”.* FROM “projects”`

![禮成，奏樂！](https://cdn-images-1.medium.com/max/2000/1*Dt9fSABZs7SSRgBYf-6PJQ.jpeg)
*禮成，奏樂！*

> ## 大功告成！

## 參考資料：

* 五倍紅寶石 [@高見龍](https://kaochenlong.com/)

* [Ruby on Rails 實戰聖經](https://ihower.tw/rails/performance.html)

* [Counter Cache Column in Rails 5](https://rubyplus.com/articles/3221-Counter-Cache-Column-in-Rails-5)

---
layout: post
title:  "在 Rails 專案安裝 PostgreSQL"
crawlertitle: "Install PostgreSQL in Rails Project"
date:   2019-06-27 10:56:46 +0800
categories: Rails
tags: ['Rails', 'Database']
comments: true
---
## 安裝

- 使用 [Homebrew](https://brew.sh/index_zh-tw
)套件管理工具安裝：
`$ brew install postgresql`

- 查看版本：`$ pg_ctl -V` 注意 V 是大寫 

- 查看安裝位置：`$ where pg_ctl` 
  > /usr/local/bin/pg_ctl
- 查詢其他指令：`$ pg_ctl --help`
- 解除安裝：`$ brew uninstall postgres`

## 啟動 postgresql

```bash
# 方法 1. 每次重開機都需要輸入指令重新啟動：
pg_ctl -D /usr/local/var/postgres start
pg_ctl -D /usr/local/var/postgres stop

# 作為系統常駐運行的服務，重開機也會繼續存在
$ brew services start postgresql
$ brew services stop postgresql
$ brew services restart postgresql
```

- 進入資料庫所在的路徑：
`cd /usr/local/var/postgres/base` 

## 進入 DB Console:

`psql`

```
\q = exit
\l = list
```

## 在 Rails 專案建立 PostgreSQL 資料庫

### Step 1. 安裝 pg gem
Option 1. 建立新專案時就選擇使用 PostgreSQL  
`$ rails new myapp --database=postgresql`

Option 2. 
在既有專案的 Gemfile 內加入 `gem pg`

- 如果只有在 production 環境下使用，則將 `pg` 放在 production 群組，但一般比較不建議一個專案在各種環境用到不同類型的資料庫。

```ruby
group :production do
  # 請至 rubygems.org 查詢最新穩定版本
  gem 'pg', '~> 0.18.4' 
end
```

> 因為一個專案只需要有一個資料庫，如果沒有在各別環境下使用不同廠牌資料庫的需求，請移除預設的 `gem sqlite3`

### Step 2. 設定資料庫 adapter
接著要設定 Rails 專案的 config/database.yml

```ruby
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch('RAILS_MAX_THREADS') { 5 } %>

development:
  <<: *default
  database: YOUR_PROJECT_NAME_with_pg_development

test:
  <<: *default
  database: YOUR_PROJECT_NAME_with_pg_test

production:
  <<: *default
  database: YOUR_PROJECT_NAME_with_pg_production
  username: YOUR_PROJECT_NAME_with_pg
  password: <%= ENV['YOUR_PROJECT_NAME_WITH_PG_DATABASE_PASSWORD'] %>
```

### Step 3. 建立專案資料庫
- 從 rails 專案建立對應的資料庫：
  `$ rails db:create`

- 如果已經有 `schema` 和 `seed`，可以執行：`$ rails db:setup`
  其效果等於 `db:create + db:schema:load + db:seed`

- 另外如果想在終端機透過 PostgreSQL 建立資料庫：
  - 創造一個資料庫：
`$ createdb <db_name>`
  - 連接進入資料庫查看： `$ psql <db_name>`
  - 退出資料庫： `⌃ + D` 或 `\q`

### Step 4. 執行 migration 啟動伺服器
接著才能執行資料庫操作 `$ rails db:migrate`
啟動伺服器 `$ rails s`

> 大功告成🙌
---

## 補充： 如果遇到 readline & postgresql dependancy issue

出現版本相依性衝突，表示以前曾經安裝過 PostgreSQL， 重安裝新版即可解決：

`brew uninstall postgresql && brew install postgresql`

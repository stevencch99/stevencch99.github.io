---
layout: post
title: "Rails 上傳圖片超簡單 To upload images via CarrierWave, Amazon S3 and Figaro"
description: "Rails 上傳圖片超簡單 To upload images via CarrierWave, Amazon S3 and Figaro"
crawlertitle: "Rails 上傳圖片超簡單 To upload images via CarrierWave, Amazon S3 and Figaro"
date: 2019-07-16 14:50:46 +0800
categories: Rails
tags: Rails
comments: true
---

## 本篇摘要：
- 透過 [CarrierWave](https://github.com/carrierwaveuploader/carrierwave) 套件上傳圖片到 [Aamzon S3](https://aws.amazon.com/tw/s3/)(Simple Storage Service)
- 用 [Figaro](https://github.com/laserlemon/figaro) 管理金鑰等機敏資訊
- 利用 [MiniMagick](https://github.com/minimagick/minimagick) 指定圖片尺寸

以上傳使用者頭像為例：

---
## 安裝套件們
在 `Gemfile` 新增以下幾行，為避免將來套件更新後發生相容性衝突，建議上 [RubyGems](https://rubygems.org/) 找到目前穩定版本版號。

```ruby
gem 'carrierwave', '~> 1.3', '>= 1.3.1' # 本篇主角
gem 'mini_magick', '~> 4.5', '>= 4.5.1' # 將上傳的圖檔設定為指定尺寸
gem 'fog-aws', '~> 3.5', '>= 3.5.1'     # 支援上傳檔案到 AWS S3
gem 'figaro', '~> 1.1', '>= 1.1.1'      # 機密資訊傳遞套件
```

執行 `$ bundle install` 給他裝起來。

## 建立 uploader
輸入以下指令：

`$ rails g uploader Avatar`

將會產生此檔案 `app/uploaders/avatar_uploader.rb`，預設有許多註解說明各種功能選項，把相對應的註解拿掉就可啟用，接著依照我們的需求修改其內容：

```ruby
class AvatarUploader < CarrierWave::Uploader::Base

  # 判斷目前 Rails 運作環境：
  if Rails.env.production?
    # 如果是 production 環境，則將檔案儲存至遠端(S3)
    storage :fog
  else
    # 開發的過程，將檔案儲存至本地（預設路徑為 專案資料夾/public/uploads/...)
    storage :file
  end

  # 設定儲存的檔案路徑 e.g.：/public/uploads/user/avatar/1／foo.jpg
  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  # 設定上傳圖檔類型的白名單
  def extension_whitelist
    %w(jpg jpeg gif png)
  end
end
```
> 因為測試環境的圖片不是我們的重點，亦沒必要進入 commit，這裏建議順便將 public/uploaders 資料夾加到 .gitignore

## 設定圖片儲存欄位
我們希望給 user 增加名為 avatar 的欄位，在終端機輸入指令並設定資料型態為 `string`
> 上傳的圖片在資料庫中會以檔名的形式存在，儲存路徑則是按照上一步驟在 uploader 中做的設定來決定
```s
$ rails g migration add_avatar_to_users avatar:string
$ rails db:migrate
```

接著把前一步製作的上傳器和 User 的 avatar 欄位 綁定：  
In `user.rb`
```ruby
class User < ActiveRecord::Base
  mount_uploader :avatar, AvatarUploader
end
```


## 安裝 ImageMagick
在許多場合我們希望裁剪使用者上傳圖片的大小，這邊按照 CarrierWave 官方建議透過 MiniMagick 套件啟用 ImageMagick 的功能，首先必須透過 Homebrew 安裝 ImageMagick。
`$ brew install imagemagick`

確認版本：
`$ convert -version`
![確認 ImageMagick 的版本](https://stevenchang.s3-ap-northeast-1.amazonaws.com/%E7%A2%BA%E8%AA%8D+ImageMagick+%E7%9A%84%E7%89%88%E6%9C%AC.jpg)

```ruby
class AvatarUploader < CarrierWave::Uploader::Base
  # 在上傳器中加入 MiniMagick 的功能
  include CarrierWave::MiniMagick

  # 上傳的圖片會被裁切成小於 800 * 800 像素的大小，原始圖片會另外保留
  process resize_to_fit: [800, 800]

  # 另存一個叫做 thumb ，大小為 200 * 200 像素的圖片版本
  version :thumb do
    process resize_to_fill: [200, 200]
  end

  # 如果需要更小的圖片尺寸，可以透過 from_version 選項，
  # 從指定的版本(thumb)製作縮圖，執行效能會比從原圖製作好。
  version :small_thumb, from_version: :thumb do
    process resize_to_fill: [20, 20]
  end
  # 以下省略
end
```
> 如果需要更小的圖片尺寸，可以透過 from_version 選項，從指定的版本(thumb)製作縮圖，執行效能會比從原圖製作來得更好。

## 設定上傳位置到 Amazon S3
> 關於如何使用 Amazon S3，官網教程有清楚的說明：[Amazon Simple Storage Service 入門](https://docs.aws.amazon.com/zh_tw/AmazonS3/latest/gsg/GetStartedWithS3.html)

於專案目錄下的 `initializers` 資料夾內新增檔案：  
`config/initializers/carrierwave.rb`
編輯內容如下：

```ruby
CarrierWave.configure do |config|
  if Rails.env.staging? || Rails.env.production?
    config.fog_provider = 'fog/aws'
    config.fog_credentials = {
      provider:               'AWS',
      aws_access_key_id:      ENV['AWS_ACCESS_KEY_ID'],
      aws_secret_access_key:  ENV['AWS_SECRET_ACCESS_KEY'],
      # 區域這邊以 亞太區域（東京）為例
      host: 's3-ap-northeast-1.amazonaws.com',
      region: 'ap-northeast-1'
    }
    config.fog_directory = ENV['S3_BUCKET_NAME']
  else
    config.storage = :file
    config.enable_processing = Rails.env.development?
  end
end
```

### 設定 Figaro 和金鑰

因為上傳過程涉及到 Access key 等機密資訊，我們不會希望它直接以明碼方式跟著專案上傳出去，故需要用到套件來管理。

Figaro 透過 `ENV['變數名稱']` 存取金鑰，這邊將會需要你準備好填入自己的資料：

1. AWS_ACCESS_KEY_ID(存取金鑰 ID)
2. AWS_SECRET_ACCESS_KEY(私密存取金鑰)
3. S3_BUCKET_NAME(儲存貯體名稱)

- 照著前面的步驟已經安裝了 Figaro ，這時執行 `$ figaro install` 會幫你建立 `config/application.yml` 檔案，同時也加入到 `.gitignore` 忽略名單內。 

編輯 `application.yml`：

```yml
production:
  AWS_ACCESS_KEY_ID: '存取金鑰 ID'
  AWS_SECRET_ACCESS_KEY: '私密存取金鑰'
  S3_BUCKET_NAME: '儲存貯體名稱'
```

完成後進入 production 環境 `$ rails c -e production`，
輸入 `$ ENV` 列出目前的環境參數，沒意外的話輸入 `$ ENV['S3_BUCKET_NAME']` 即可取用剛才設定的變數內容。

### 如果需要將參數部署到 Heroku：

設定參數： `$ figaro heroku:set -e production`  
透過 [Heroku cli](https://devcenter.heroku.com/articles/heroku-cli) 查詢設定： `$ heroku config`

## 對應的表單

```html
<%= form_for(@user, html: {id: 'user_form'}) do |form| %>
  <p>
    <label>我的頭像</label>
    <!-- 取用 small_thumb 小圖 -->
    <%= image_tag @user.avatar.small_thumb.url if @user.avatar? %>
    <%= f.file_field :avatar %>
  </p>

  <p>
    <label>
      <%= f.check_box :remove_avatar %>
      刪除頭像
    </label>
  </p>
  <%= form.submit %>
<% end %>
```

## 測試

在測試環境中上傳圖片後就會有三種尺寸不同的圖片
```ruby
# 800 * 800 pixels
User.first.avatar.url             # =>"/uploads/user/avatar/1/xxx.jpg"
# 200 * 200 pixels
User.first.avatar.thumb.url       # =>"/uploads/user/avatar/1/thumb_xxx.jpg"
# 20 * 20 pixels
User.first.avatar.small_thumb.url # =>"/uploads/user/avatar/1/small_thumb_xxx.jpg"
```

另外在 S3 的儲存貯體記得設定權限開放上傳及讀取，如此便大功告成！
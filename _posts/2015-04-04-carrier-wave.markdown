---
layout: post
title:  "Carrier Wave 使用筆記"
date: 2015-04-04 12:00:00
categories: Ruby Rails
---

## Carrier Wave
這套 gem 提供一個簡單且非常彈性的方式來協助 Ruby 程式上傳圖片，同時也能夠跟 Rake base 的程式整合的很好 e.g. Ruby on Rails 


1. 安裝 gem

```
gem 'carrierwave'
> bundle install
```

2. 建立 uploader

```
> rails g uploader Avatar
```

3. 範例我們加開一個 User

```
> rails g model User name bio:text
> rake db:migrate
```

4. 針對已經存在的物件我們加入關聯

```
> rails g migration AddAvatarToUsers avatar:string
> rake db:migrate
```
5. 調整 User Model 讓 User 可以取得 carrierwave 處理好的資訊

```
class User < ActiveRecord::Base
  mount_uploader :avatar, AvatarUploader
end
```
6. 處理 View

```
<%= form_for @user, :html => {:multipart => true} do |f| %>
  <p>
    <%= f.file_field :avatar %>
    <%= f.hidden_field :avatar_cache %>
  </p>
<% end %>
```

注意 `:html => {:multipart => true}` 要加，還有 `controller` 中的 `permit` 要放行。
e.g `params.require(:user).permit(:name, :bio, :avatar, :avatar_cache)`

接下來在 view 中就可以使用 `<%= image_tag(@user.avatar_url) %>`

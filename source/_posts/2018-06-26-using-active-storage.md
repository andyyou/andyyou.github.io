---
title: 手把手學習使用 Rails 5.2 ActiveStorage (DirectUpload + ProgressBar)
tags:
  - rails
  - stimulus
categories: Program
date: 2018-06-26 11:06:54
---


{% asset_img active-storage.gif %}

本文為使用 _Rails ActiveStorage_ 的實作範例筆記。詳細介紹請參考[Active Storage 概要](https://calvertyang.github.io/2018/05/18/active-storage-overview/)，本文僅針對官方教學提供一個對照的實作記錄，如需部署至 Heroku 請參考[在 Heroku 使用 Active Storage](https://calvertyang.github.io/2018/05/22/active-storage-on-heroku/)。


<!--more-->


## 建立 Rails 專案與安裝

```bash
# 這邊為了後續介紹與 stimulus 搭配我們直接先帶入 --webpack=stimulus 參數
$ rails new active_storage_sample --webpack=stimulus --skip-coffee --skip-test
$ rails active_storage:install
$ rails db:migrate
# 設定 config/storage.yml 提供的方式
# 設定 config/environments 環境使用的方式

# 完整範例 https://github.com/andyyou/active-stroage-sample
```

由於 <i>active_storage</i> 會使用兩張 table 記錄資料所以需要 _migrate_。

我們在這個實作會練習使用 _aws s3_ 來儲存檔案，**即便不預先設定還是可以在使用本地磁碟的方式測試**。

> 如果您不想在這邊練習使用 aws 可以直接跳至下一節。

```bash
# 加入 s3 access key
$ EDITOR=vim rails credentials:edit
```

_config/storage.yml_ 設定

```
amazon:
  service: S3
  access_key_id: <%= Rails.application.credentials.dig(:aws, :access_key_id) %>
  secret_access_key: <%= Rails.application.credentials.dig(:aws, :secret_access_key) %>
  region: ap-northeast-2
  bucket: your_own_bucket
```

開啟 _Gemfile_ 加入 _aws-sdk-s3_ 並執行 `bundle` 安裝。

> Active Storage 的核心功能需要以下權限：`s3:ListBucket`、`s3:PutObject`、`s3:GetObject` 和 `s3:DeleteObject`。如果你設定了其它上傳選項，如 ACL 設定，則可能需要額外的權限。

注意：記得要設定 _config/environments/development.rb_

```
config.active_storage.service = :amazon
```



## 標準 Form Post 方式

#### 新增圖片（單檔/多檔）

使用官方提供的標準方式上傳檔案

```bash
# 建立 event scaffold 我們將練習使用 單檔上傳、多檔上傳、upload_direct 參數
$ rails g scaffold event name
$ rails db:migrate
```

<i>active_storage</i> 的使用方式非常簡單：

1 調整 _models/event.rb_

```ruby
class Event < ApplicationRecord
  has_one_attached :cover # 單檔
  has_many_attached :banners # 多檔
end
```

2 調整 <i>views/events/_form.html.erb</i>

```html
<div class="field">
  <%= form.label :cover %>
  <%= form.file_field :cover %>
</div>

<div class="field">
  <%= form.label :banners %>
  <%= form.file_field :banners, multiple: true %>
</div>
```

3 調整 <i>controllers/events_controller.rb</i>

```ruby
# premit cover 和 banners
def event_params
  params.require(:event).permit(:name, :cover, banners: [])
end


# 以下為說明，不需使用於範例
# 若要附加圖片可以使用 attach
@event.attach(params[:cover])
# 同步刪除頭像和實際資源檔案。
@event.cover.purge
# 透過 Active Job 非同步刪除相關模型和實際資源檔案。
@event.cover.purge_later
```

4 為了觀察結果與使用呈現的相關 _helpers_，調整 `views/events/show.html.erb`  加上

```html
<p>
  <strong>Cover:</strong>
  <div>
    <%= image_tag @event.cover if @event.cover.attached? %>
  </div>
</p>

<p>
  <strong>Banners:</strong>
  <div>
    <% @event.banners.each do |banner| %>
      <%= image_tag banner %>
    <% end %>
  </div>
</p>
```

以上就是最基本的使用方式，我們可以啟動 `rails s` 並瀏覽 `localhost:3000/events`來觀察目前的結果。

#### 調整圖片尺寸

1 要使用調整圖片尺寸的功能須先安裝 `mini_magick`，在 _Gemfile_ 解開註解並安裝。

```
# Use ActiveStorage variant
gem 'mini_magick', '~> 4.8'
```

2 接著就可以在 _views/events/show.html.erb_ 使用 `.variant(resize: '100x100')` 方法。

```erb
<%= image_tag @event.cover.variant(resize: '100x100') if @event.cover.attached? %>
```

#### 刪除圖片

1 _config/routes.rb_ 新增路由

```ruby
resources :events do
  delete :destroy_cover, on: :member
end
```

2 <i>controllers/events_controller.rb</i> 新增 _action_ 與設定 `:set_event`

```ruby
before_action :set_event, only: [:show, :edit, :update, :destroy, :destroy_cover]
def destroy_cover
  @event.cover.purge
  respond_to do |format|
    format.html { redirect_to event_url(@event), notice: 'Event Cover was successfully destroyed.' }
    format.json { head :no_content }
  end
end
```

3 _views/events/show.html.erb_ 加入刪除按鈕

```html
<%= link_to '刪除', destroy_cover_event_path(@event), method: :delete if @event.cover.attached? %>
```

#### 多檔上傳刪除單張圖片

1 _config/routes.rb_ 新增路由

```ruby
resources :events do
  delete :destroy_cover, on: :member
  # DELETE /events/:id/banners/:banner_id
  delete '/banners/:banner_id' => 'events#destroy_banner', as: :destroy_banner, on: :member
end
```

2 <i>controllers/events_controller.rb</i> 新增 _action_ 和設定 `:set_event`

```ruby
before_action :set_event, only: [:show, :edit, :update, :destroy, :destroy_cover, :destroy_banner]

def destroy_banner
  @event.banners.find(params[:banner_id]).purge
  respond_to do |format|
    format.html { redirect_to event_url(@event), notice: 'Event banner was successfully destroyed.' }
    format.json { head :no_content }
  end
end
```

3 _views/events/show.html.erb_

```html
<% @event.banners.each do |banner| %>
  <%= image_tag banner.variant(resize: '100x100') %>
  <%= link_to '刪除', destroy_banner_event_path(@event, banner), method: :delete %>
<% end %>
```

#### 多檔上傳刪除多張圖片

1 _config/routes.rb_ 新增路由

```ruby
# DELETE /events/:id/banners
delete :destroy_banners, on: :member
```

完整路由

```ruby
Rails.application.routes.draw do
  resources :events do
    delete :destroy_cover, on: :member

    # DELETE /events/:id/banners
    delete :destroy_banners, on: :member
    # DELETE /events/:id/banners/:banner_id
    delete '/banners/:banner_id' => 'events#destroy_banner', as: :destroy_banner, on: :member
  end

  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

. <i>controllers/events_controller.rb</i> 新增 _action_ 和設定 `:set_event`

```ruby
before_action :set_event, only: [
  :show, :edit, :update,
  :destroy, :destroy_cover,
  :destroy_banner,
  :destroy_banners
]

def destroy_banners
  params[:event][:banners].each do |banner_id|
    @event.banners.find(banner_id).purge
  end
  respond_to do |format|
    format.html { redirect_to event_url(@event), notice: 'Event banners was successfully destroyed.' }
    format.json { head :no_content }
  end
end
```

3 _views/events/show.html.erb_

```erb
<%= form_for(@event, url: destroy_banners_event_path, method: :delete) do |form| %>
  <% @event.banners.each do |banner| %>
    <!--重點：使用 checkbox 勾選並一次刪除 -->
    <%= check_box_tag :banners, banner.id, false, name: 'event[banners][]' %>
    <%= image_tag banner.variant(resize: '100x100') %>
    <%= link_to '刪除', destroy_banner_event_path(@event, banner), method: :delete %>
  <% end %>

  <% if @event.banners.attached? %>
    <input type="submit" value="刪除多張圖片">
  <% end %>
<% end %>
```

到此我們已經示範了完整的基本使用方式。

## Direct Upload

預設的 active storage 的流程是將圖片先送到後端，一併處理建立資料庫紀錄和上傳。但如何使用雲端服務的話，這個流程就顯得多此一舉。因此 active storage 也提供 direct upload 的方式直接把圖片從使用者端直接送往雲端服務。而我們接著要實作 ajax 方式的範例也會使用 direct upload。

#### 安裝 activestorage.js

1 安裝套件

```bash
# 這裡我們使用 webpacker 的方式，如果需要其他方式請參考官方教學
$ yarn add activestorage
```

2 新增 <i>javascript/packs/direct_upload.js</i>

```js
import * as ActiveStorage from 'activestorage';
ActiveStorage.start();
```

3 _views/layouts/application.html.erb_ 加入 pack

```html
<%= javascript_pack_tag 'direct_upload', 'data-turbolinks-track': 'reload' %>
```

#### 標準 Direct Upload 使用方式

為了範例單純，這邊我們建立一個新的 _Post scaffold_  其包含一個 `cover` 和 `images` 但是這次我們使用不一樣的流程來完成。`cover` 我們使用標準的 Direct Upload 作法，`images` 我們整合 ajax 與 `stimulus` 的作法。

1 建立 _scaffold_

```bash
$ rails g scaffold post title
$ rails db:migrate
```

2 _models/post.rb_ 加上設定

```ruby
class Post < ApplicationRecord
  has_one_attached :cover
  has_many_attached :images
end
```

3 <i>views/posts/_form.html.erb</i> 加上

```erb
<div class="field">
  <%= form.label :cover %>
  <%= form.file_field :cover, direct_upload: true %>
</div>
```

到這邊除了 `direct_upload` 參數跟原本的作法沒有不同，但使用 `direct_upload` 之後我們多了一些 _hooks_ 可以使用。

> direct_upload: true 會在渲染的 HTML 加上 data-direct-upload-url 屬性。

4 <i>controllers/posts_controller.rb</i> 加入 permit

```ruby
def post_params
  params.require(:post).permit(:title, :cover, images: [])
end
```

5 完成的 _javascript/packs/direct_upload.js_ 如下，這是官方提供的範例

```js
import * as ActiveStorage from 'activestorage';
ActiveStorage.start();

addEventListener("direct-upload:initialize", event => {
  const { target, detail } = event;
  const { id, file } = detail;
  target.insertAdjacentHTML("beforebegin", `
    <div id="direct-upload-${id}" class="direct-upload direct-upload--pending">
      <div id="direct-upload-progress-${id}" class="direct-upload__progress" style="width: 0%"></div>
      <span class="direct-upload__filename">${file.name}</span>
    </div>
  `);
});

addEventListener("direct-upload:start", event => {
  const { id } = event.detail;
  const element = document.getElementById(`direct-upload-${id}`);
  element.classList.remove("direct-upload--pending");
});

addEventListener("direct-upload:progress", event => {
  const { id, progress } = event.detail;
  const progressElement = document.getElementById(`direct-upload-progress-${id}`);
  progressElement.style.width = `${progress}%`;
});

addEventListener("direct-upload:error", event => {
  event.preventDefault();
  const { id, error } = event.detail;
  const element = document.getElementById(`direct-upload-${id}`);
  element.classList.add("direct-upload--error");
  element.setAttribute("title", error);
});

addEventListener("direct-upload:end", event => {
  const { id } = event.detail;
  const element = document.getElementById(`direct-upload-${id}`);
  element.classList.add("direct-upload--complete");
});
```

6. 加入 css

```css
.direct-upload {
  display: inline-block;
  position: relative;
  padding: 2px 4px;
  margin: 0 3px 3px 0;
  border: 1px solid rgba(0, 0, 0, 0.3);
  border-radius: 3px;
  font-size: 11px;
  line-height: 13px;
}

.direct-upload--pending {
  opacity: 0.6;
}

.direct-upload__progress {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  opacity: 0.2;
  background: #0076ff;
  transition: width 120ms ease-out, opacity 60ms 60ms ease-in;
  transform: translate3d(0, 0, 0);
}

.direct-upload--complete .direct-upload__progress {
  opacity: 0.4;
}

.direct-upload--error {
  border-color: red;
}

input[type=file][data-direct-upload-url][disabled] {
  display: none;
}
```

## 整合 Stimulus

1 _views/layouts/application.html.erb_ 加入 application

```erb
<%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
```

2 新增 <i>javascript/controllers/uploads_controller.rb</i>

```js
import { Controller } from 'stimulus';

export default class extends Controller {
  connect() {
    console.log('connect to uploads');
  }

  start() {
    console.log('start upload');
  }
}
```

3 _views/posts/show.html.erb_ 加入我們的 _upload 元素_ 並使用 `stimulus` 的 `controller`。

```erb
<p>
  <strong>Images</strong>
  <div data-controller="uploads">
    <input type="file" multiple="true" data-action="change->uploads#start">
  </div>
</p>
```

這裡我們預計使用一個 `input` ，當其取得檔案的時候在搭配 stimulus 執行對應的操作。接著，我們先來處理上傳檔案的部分。

4 確認 controller 中  _params_ 和 `before_action` 是否取得我們需要的資料，修改 *controllers/posts_controller.rb*，我們會需要使用 ajax 來對 `update` 發出請求，所以我們需要對其做一些調整。一個流程我們從路由開始，我們沿用 `update` ，接著 `controller#action` 的行為。再回到前端處理。

   > 注意：本文旨是在協助您練習可能的作法，不一定適合您的正式環境。

```ruby
# PATCH/PUT /posts/1
# PATCH/PUT /posts/1.json
def update
  respond_to do |format|
    if @post.update(post_params)
      format.html { redirect_to @post, notice: 'Post was successfully updated.' }
      format.json {
        # 遵循慣例參數為陣列，但 DirectUpload 一次只會負責一張圖片
        image = ActiveStorage::Blob.find_signed(post_params[:images].first)
        # 從後端取的圖片（resize）的網址
        image_url = Rails.application.routes.url_helpers.rails_representation_url(image.variant(resize: '100x100'), only_path: true)
        render json: { status: :ok, url: image_url, id: image.id }
      }
    else
      format.html { render :edit }
      format.json { render json: @post.errors, status: :unprocessable_entity }
    end
  end
end
```

5 新增 _javascript/libs/uploader.js_ 。這裡為了可以顯示進度，我們參考官方教學的作法。

   > 注意：如果您是直接跳至本節，請記得安裝 `activestorage.js`

```js
import { DirectUpload } from 'activestorage';

export default class {
  constructor(file, url, element) {
    this.file = file;
    this.url = url;
    this.element = element;
    this.directUpload = new DirectUpload(this.file, this.url, this);
  }

  upload() {
    return new Promise((resolve, reject) => {
      this.directUpload.create((error, blob) => {
        if (error) {
          reject(error);
        } else {
          resolve(blob);
        }
      });
    });
  }

  directUploadWillStoreFileWithXHR(request) {
    request.upload.addEventListener("progress",
      event => this.directUploadDidProgress(event));
  }

  directUploadDidProgress(e) {
    let progress = this.element.querySelector('.progress-bar');
    progress.style.width = ((e.loaded / e.total) * 100) + '%';
  }
}
```

6 _views/posts/show.html.erb_ 由於 js 需要一些參數，這邊我們使用 stimulus 的 data api

```js
<p>
  <strong>Images</strong>
  <div data-controller="uploads"
    data-uploads-model="<%= @post.to_json %>"
    data-uploads-direct-upload-url="<%= rails_direct_uploads_path %>"
  >
    <input type="file" multiple="true" data-action="change->uploads#start">
  </div>
</p>
```

7 <i>javascript/controllers/uploads_controller.js</i>

```js
import { Controller } from 'stimulus';
import Uploader from 'libs/uploader';

export default class extends Controller {
  start(event) {
    const { target } = event;
    const _this = this;
    [...target.files].forEach(file => {
      // 準備 image 容器與 progress bar
      let wrapper = document.createElement('div');
      wrapper.classList.add('img-wrapper');
      wrapper.insertAdjacentHTML('afterbegin', `
        <div class="progress">
          <div class="progress-bar" style="width: 0%;"></div>
        </div>
      `);
      const insertTarget = _this.element.querySelector('input[type=file]');
      _this.element.insertBefore(wrapper, insertTarget);
      // 開始上傳
      const uploader = new Uploader(file, _this.directUploadUrl, wrapper);
      uploader.upload()
        .then(blob => {
          // 更新資料庫
          fetch(`/posts/${_this.model.id}.json`, {
            headers: {
              'X-CSRF-Token': _this.csrf,
              'Content-Type': 'application/json',
              'X-Requested-With': 'XMLHttpRequest'
            },
            method: 'PUT',
            body: JSON.stringify({
              post: {
                images: [blob.signed_id]
              }
            }),
            credentials: 'same-origin'
          })
          .then(res => res.json())
          .then(data => {
            wrapper.innerHTML = `
              <div class="lds-dual-ring"></div>
            `;
            let img = document.createElement('img');
            img.src = data.url;
            img.onload = () => {
              wrapper.innerHTML = '';
              wrapper.appendChild(img);
              wrapper.insertAdjacentHTML('beforeend', `
                <a href="/posts/${_this.model.id}/images/${data.id}"
                  data-action="click->uploads#destroy">
                  刪除
                </a>
              `);
            };
          });
        });
    });
    target.value = '';
  }

  get model() {
    return JSON.parse(this.data.get('model'));
  }

  get directUploadUrl() {
    return this.data.get('directUploadUrl')
  }

  get csrf() {
    return document.querySelector('meta[name="csrf-token"]').getAttribute('content');
  }
}

```

8 scss 的部分

```scss
.uploads {
  display: flex;
  flex-wrap: wrap;
}

.img-wrapper {
  display: inline-flex;
  border: 1px solid #d9d9d9;
  min-width: 100px;
  min-height: 100px;
  border-radius: 3px;
  margin-right: 15px;
  padding: 5px;
  align-items: center;
  flex-direction: column;
  justify-content: center;

  .progress {
    width: 80%;
    height: 10px;
    background-color: #ccc;
    border-radius: 5px;
    position: relative;

    .progress-bar {
      position: absolute;
      top: 0;
      left: 0;
      bottom: 0;
      opacity: 0.8;
      border-radius: 5px;
      background: #0076ff;
      transition: width 120ms ease-out, opacity 60ms 60ms ease-in;
      transform: translate3d(0, 0, 0);
    }
  }
}

.lds-dual-ring {
  display: inline-flex;
  width: 64px;
  height: 64px;
  justify-content: center;
  align-items: center;
}

.lds-dual-ring:after {
  content: " ";
  display: block;
  width: 46px;
  height: 46px;
  margin: 1px;
  border-radius: 50%;
  border: 5px solid #327ccb;
  border-color: #327ccb transparent #327ccb transparent;
  animation: lds-dual-ring 1.2s linear infinite;
}

@keyframes lds-dual-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
```

9 刪除功能 - _config/routes.rb_

```ruby
resources :posts do
  delete '/images/:image_id' => 'posts#destroy_image', as: :destroy_image, on: :member
end
```

10 <i>controllers/posts_controller.rb</i>

```ruby
before_action :set_post, only: [:show, :edit, :update, :destroy, :destroy_image]

# DELETE /posts/1/images/2
def destroy_image
  @post.images.find(params[:image_id]).purge
  render json: { status: :ok }
end
```

11 <i>javascript/controllers/uploads_controller.js</i> 加入刪除功能

```js
destroy(e) {
  e.preventDefault();
  const url = e.target.href;
  fetch(url, {
    headers: {
      'X-CSRF-Token': this.csrf,
      'Content-Type': 'application/json',
      'X-Requested-With': 'XMLHttpRequest'
    },
    method: 'DELETE',
    credentials: 'same-origin'
  })
  .then(res => res.json())
  .then(data => {
    e.target.parentElement.remove();
  });
}
```

12 調整 _views/posts/show.html.erb_

```erb
<p>
  <strong>Images</strong>

  <div class="uploads"
    data-controller="uploads"
    data-uploads-model="<%= @post.to_json %>"
    data-uploads-direct-upload-url="<%= rails_direct_uploads_path %>"
  >
    <% @post.images.each do |image| %>
      <div class="img-wrapper">
        <%= image_tag image.variant(resize: '100x100') %>
        <a href="<%= destroy_image_post_path(@post, image) %>" data-action="click->uploads#destroy">刪除</a>
      </div>
    <% end %>

    <input type="file" multiple="true" data-action="change->uploads#start">
  </div>
</p>
```

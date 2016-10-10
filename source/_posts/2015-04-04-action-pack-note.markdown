---
layout: post
title:  "ActionPack 雜記"
date: 2015-04-04 12:00:00
categories: Program
tags: ruby, RoR
---

# Action Pack
Action Pack 是整個 Rails 的核心部份，由 ActionDispatch, ActionController, ActionView 組成
ActionDispatch 處理接收到的請求(Requests)，即網址的部分，ActionController 負責把請求對應轉換成回應(Responses)
接著 ActionController 調用 ActionView 來處理回應的格式(html, js, json, xml) 等
<!--more-->


通常 ActiveRecord 比較常會被獨立使用，而 ActionPack 通常我們會透過 Rails 整合在一起使用。

## 路由
在 Rails 路由通常分成2大類 comprehensive, resources ，通用型的路由就是讓你指定 [http verb] [mapping url] [controller#action]
具體的流程是當 Server 透過網址取得請求，此時 Rails 會透過 Action Dispatch 去處理網址判斷該對應到哪個 controller 和 action 並把 parameter 帶進去




~~~ruby
Sample::Application.routes.draw do
  resources :products # Resources 會產生 7 組路由
end
~~~

上面的範例 Rails 會先假設您有一個 ProductsController

~~~ruby
resources :people, except: [:update, :destroy]
~~~


~~~ruby
def index
  @products = Product.all
end

def show
  @product = Product.find(params[:id])
end

def new
  @product = Product.new
end

def create
  @product = Product.new(prarms.require(:product).premit(:title, :description, :image_url, :price))

  respond_to do |format|
    if @product.save
      format.html { redirect_to @product, notice: "Product was successfully created" }
      format.json { render action: 'show', status: :created, location: @product}
    else
      format.html { render action: 'new' }
      format.json { render json: @product.errors, status: :unprocessable_entity }
    end
  end
end

def edit
end

def update
end

def destroy
end
~~~

# 共用 route

~~~ruby
concern :reviewable do
  resources :reviews
end

resources :products, concern: :reviewable
~~~


## 路由得設法

~~~ruby
root :to => 'welcome#index'
root to: 'controller#action'
match 'messages/show', to: 'messages#show', :via => 'get'
match 'messages/show' => 'messages#show', via: [:get, :post]
get 'messages/show' => 'messages#show'
# 省略的寫法
match 'messages/show' # 相等於 match 'messages/show'  => 'messages#show'
match 'messages' => 'messages#index', :as => 'index'
match "/messages/show/:id" => "messages#show", :constraints => {:id => /\d/} # 限制參數
~~~

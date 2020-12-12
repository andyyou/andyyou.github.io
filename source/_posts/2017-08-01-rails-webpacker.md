---
title: Rails Webpacker 筆記
tags:
  - rails
  - ruby
categories: Program
date: 2017-08-01 15:04:39
---


在 Javascript 不斷快速變化的今日，對 Rails 而言一直有個相對支援度較不足的地方，那就是支援一些較新的 Javascript 封裝機制（bundle）。不過從 5.1 開始這個大家希望的功能將會支援 - 全新的 webpacker gem。

Rails 5.1 開始內建可以使用 `--webpack` 參數開啟支援 `webpacker` 和 `yarn`，並且支援整合 `React`，`Angular`，`Vue`，`elm` 等。

<!--more-->

# 建立

在開始使用之前我們需要確保 `ruby`，`node`，`yarn` 已經安裝。至於 node 和 yarn 的安裝步驟請參考官方文件。

> 筆者建議使用 nvm 安裝 node，並使用 npm 安裝 yarn。這麼做是因為在使用 nvm 支援切換 node 版本的情況下 yarn 安裝全域函式庫可能會遇上一些問題。[Github Issue](https://github.com/yarnpkg/yarn/issues/648)


建立專案

```bash
$ rails new <project_name> --webpack

# 支援其他函式庫
$ rails new app --webpack=react
$ rails new app --webpack=angular
$ rails new app --webpack=vue
```

這個新的參數 `--webpack` 會在 Rails 5.1 應用程式建立之後安裝與設定 yarn 與 webpack。一旦應用程式建立，webpacker 也會被安裝，我們就可以在 Rails 中使用 webpack 了。

如果我們的專案是已存在的 4.2 Rails 應用程式那麼可以使用 Gemfile 安裝 webpacker，在安裝 Gem 之後執行：

```bash
$ rails webpacker:install
$ rails webpacker:install:vue
```

除了安裝指令之外我們還可以透過下面的指令查詢 Rails 提供的相關指令

```bash
$ bundle exec rails webpacker
$ bundle exec rails -T
```

# 設定

維持 Rails 一貫的精神，webpacker 一樣也具備一些預設慣例

## 目錄架構

首先，我們來看看這個新的目錄 `app/javascript`。這個目錄就是用來放置 webpack 將要編譯的 Javascript 檔案，當然也包含 webpack 的 entry 檔，即 `packs/` 下的檔案。依據慣例 webpack 的進入點檔案(entry)預設放在 `app/javascript/packs`，然後其他模組或元件可以放在 `app/javascript` 目錄下。意思是說假如我們有個 `app/javascript/modules/test.js` 的 library，在 `packs` 中的進入檔只要 `import test from 'modules/test.js'` 就可以載入了。注意到我們`可以`使用相對路徑 `./modules/test.js` 或使用`模組路徑`的方式。

## webpack 設定

Rails 的產生器會幫我們加入各種環境使用的 webpack 設定檔到 `config/webpack` 目錄下。這些設定檔針對不同環境（production/development/test）提供了對應的 webpack 功能，例如：code-splitting，asset-fingerprinting，樣式的部分還支援了 post-css。

## Live reloading

webpacker 還提供了 `binstub` 可以讓我們執行 `webpack-dev-server`，在開發環境下可以提供 live reloading 的功能，如果要使用額外的 HRM(Hot Module Replacement) 功能的話則需要安裝其他套件。

# 整合練習

## 1. 設定/使用 foreman

首先讓我們在 Gemfile 的 development 群組中加入 `foreman` 並安裝來管理多個程序，這是因為我們除了原本的 `rails server` 這道程序外還需要 `bin/webpack-dev-server`。接著在專案根目錄建立 `Procfile` 和 `Procfile.dev` 設定 foreman。

```
web: bundle exec puma -p $PORT
```

```
web: bundle exec rails server
webpacker: ./bin/webpack-dev-server
```

同時我們也建立一個簡單的 `binstub` 用它來執行 foreman 套用 `Procfile.dev`。在 `bin/` 目錄底下建立一個 `server` 檔案。

```shell
#!/bin/bash -i
bundle install
bundle exec foreman start -f Procfile.dev
```

為了能夠執行，我們需要賦予該檔案執行權限

```bash
$ chmod +x bin/server
```

之後，我們就可以執行下面指令一口氣開啟 `rails server` 和 `webpack-dev-server`

```bash
$ ./bin/server
```

> 注意：使用 foreman 之後，存取網址會變成 `http://localhost:5000`

## 2. 撰寫程式碼

現在我們來試著加入一個 `counter` 模組，第一步我們在 `app/javascript` 中建立一個 `counter` 目錄，如上面提到的 webpack 處理的檔案都放置在此，這個 `counter` 目錄代表著一個模組。然後在這個 counter 目錄中建立兩個檔案 `index.js` 和 `counter.js`

*counter.js*

```js
const incrementNode = document.getElementById('increment')
const decrementNode = document.getElementById('decrement')
const inputNode = document.getElementById('counter')

const counter = {
  initialize () {
    incrementNode.addEventListener('click', (event) => {
      event.preventDefault()
      const currentVal = inputNode.value
      inputNode.value = ~~currentVal + 1
    })

    decrementNode.addEventListener('click', (event) => {
      event.preventDefault()
      const currentVal = inputNode.value
      if (currentVal > 0) {
        inputNode.value = ~~currentVal - 1
      }
    })
  }
}

export default counter
```

*index.js*

```js
// 不使用相對路徑可以從 `app/javascript` 當作根
// import counter from 'counter/counter'
import counter from './counter'

document.addEventListener('DOMContentLoaded', () => {
  counter.initialize()
})
```

再來為了讓 webpck 處理這個模組我們需要一個進入檔，在 `packs/` 目錄下建立 `counter.js` 檔案。

```js
import 'counter'
```

> 再次強調，不使用相對路徑的話可以把 `app/javascript` 當作根來取路徑。

## 3. 建立 view

為了展示我們的 `counter` 模組，讓我們建立一個簡單的 controller

```bash
$ bundle exec rails g controller pages index
```

這邊單純是為了示範，我們就直接把這個 action 的路由設成 root。修改 `routes.rb`

```
root to: 'pages#index'
```

最後，我們修改 `pages/index.html.erb`

```html
<div class="counter-wrapper">
  <h1>Counter</h1>
  <form class="counter">
    <button id="increment">Increment +</button>
    <input type="number" name="counter" id="counter" value="0">
    <button id="decrement">Decrement -</button>
  </form>
</div>

<%= javascript_pack_tag 'counter' %>
```

這邊的重點在 `javascript_pack_tag` 這個 helper 可以協助我們取得編譯好的 `counter` script。產出來的 tag 如下

```html
<script src=”http://localhost:8080/counter.js"></script>
```

## 4. 執行

我們已經完成了整個範例，接著就可以測試看看

```bash
$ bin/server
```

`rails server` 和 `webpack-dev-server` 啟動後我們便可以瀏覽 `http://localhost:5000/ ` 看看我們完成的範例。

## 5. 樣式

webpack 強大的地方在於它不僅可以處理 Javascript，只要搭配 loader，也是可以處理各種類型的檔案。這邊我們將示範使用 scss。
建立 `app/javascript/counter/style.scss` 

```scss
$grey: #f2f2f2;

.counter-wrapper {
  max-width: 500px;
  margin: 100px auto;
  padding: 10px;
  border: 1px solid $grey;

  form {
    display: block;
    margin-bottom: 10px;

    button {
      display: inline-block;
      background: #4fc08d;
      color: white;
      border: 1px solid $grey;
      padding: 5px;
      border-radius: 5px;
    }
  }
}
```

回到 `index.js` 我們匯入 scss

```js
import './style.scss'
import counter from './counter'

document.addEventListener('DOMContentLoaded', () => {
  counter.initialize()
})
```

就可以在 view 中使用 `stylesheet_pack_tag` helper。

```html
<%= stylesheet_pack_tag 'counter' %>
```

我們回到瀏覽器便可以觀察到 live reload 已經幫我們更新好頁面了，並且樣式的部分增加了如下的 tag

```html
<link rel="stylesheet" media="screen" href="http://localhost:8080/counter.css">
```

# 部署

這一小結我們來看看關於部署的部分，尤其是部署到 Heroku。假如我們使用 webpacker 那麼 Heroku 預設就會安裝 yarn 和 node。

```bash
$ heroku create
$ heroku addons:create heroku-postgresql:hobby-dev
$ git push heroku master
```

# 參考

* [webpacker 入門](https://medium.com/statuscode/introducing-webpacker-7136d66cddfb)
* [webpacker github](https://github.com/rails/webpacker)
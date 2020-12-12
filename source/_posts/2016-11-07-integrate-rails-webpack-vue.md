---
title: 穠纖合度的整合 Rails、Webpack、Vuejs
tags:
  - RoR
  - javascript
  - nodejs
categories: Program
date: 2016-11-07 18:41:37
---


# 前言

如果您曾經閱讀過小弟的[vue + webpack 起手式](https://segmentfault.com/a/1190000005363030?_ea=1303942)一文，裡面我曾提到關於容易與傳統 MVC 框架(Rails, ASP.NET MVC)等結合。這篇文章主要就是用來介紹其作法與為什麼我會這麼說。

我先承認吧！我不總是需要使用 SPA 的架構  雖然它很好，但對於很多小型專案，或者我們應該說 UI/UX 設計本身非常單純的專案 - 殺雞焉用牛刀。所以這邊文章的作法針對的是那些專案`不是`使用 SPA 搭配 API 的架構的人，而是您覺得既有的 MVC 框架對於您的需求開發已經非常足夠且想保留大多數這些框架提供的功能（本文僅針對 Rails 介紹）。

網路上有非常多關於 Rails 與 Webpack 整合的方式，其實都非常不錯。而關於本文只是介紹其中一種小弟認為`剛剛好`的整合方式，其中許多觀念都不是新創的，都只是整理過去學習經驗而成。

<!--more-->

# 需求

先談談我們的需求與為什麼不使用 asset pipeline，其實簡單來說就是下面列表這些：

* 我們想使用 ES2015 等新支援的語法，需要 Babel。
* 許多套件在 bower 上面編譯後的版本並不完整、不支援、或者有其他問題導致我們需要使用 npm。
* 我們想透過模組來管理相依性的問題。
* assets-org 和 gem 對於管理前端資源包來說並不是那麼優秀。
* 部署希望能夠盡可能單純一點。

# 準備

大體來說這種整合的架構使用 npm (yarn 也可) 來管理前端的資源，gem 則只負責後端部分。

* 安裝 nodejs 與 npm
* 安裝 ruby 與 Rails
* 安裝 Postgre SQL(由於會示範部署至 Heroku 所以使用 Postgre SQL)

# 第一步 - 建立專案

```bash
# 由於示範部署至 Heroku
# 因此無法使用 sqlite3
$ rails _5.0.0.1_ new [project_name] --database postgresql
$ cd [project_name]
$ bundle install
$ rails db:create

$ rails g controller pages index # 建立測試頁面
$ rails server # 完成建立一個 Rails 專案
# 開啟 http://localhost:3000/pages/index
```

* 導入 npm

```bash
$ npm init --yes

# 過程中，如果需要測試一下 webpack 指令則需要安裝全域
$ npm i -g webpack webpack-dev-server

# 安裝所需的前端函式庫
$ npm i jquery@^2.1.4 -S
$ npm i jquery-ujs@~1.1.0-1 -S
$ npm i lodash@~3.0.0 -S
$ npm i vue -S

# 安裝開發環境所需的套件與函式庫
$ npm i webpack \
    webpack-dev-server \
    webpack-manifest-plugin \
    extract-text-webpack-plugin -D

$ npm i babel-core \
    babel-loader babel-runtime \
    babel-plugin-transform-runtime \
    babel-preset-es2015 -D # Babel 相關

$ npm i coffee-loader coffee-script -D

$ npm i css-loader \
    style-loader \
    node-sass \
    sass-loader -D

$ npm i exports-loader -D # 匯出檔案
$ npm i expose-loader -D # 將物件加到全域(Javascript)
$ npm i file-loader url-loader -D
$ npm i imports-loader -D # 使用的模組可相依於全域
```

為了方便您快速安裝下面提供 `package.json`

```json
"dependencies": {
  "jquery": "^2.2.4",
  "jquery-ujs": "^1.1.0-1",
  "lodash": "^3.0.1",
  "vue": "^2.0.5"
},
"devDependencies": {
  "babel-core": "^6.18.2",
  "babel-loader": "^6.2.7",
  "babel-plugin-transform-runtime": "^6.15.0",
  "babel-preset-es2015": "^6.18.0",
  "babel-runtime": "^6.18.0",
  "coffee-loader": "^0.7.2",
  "coffee-script": "^1.11.1",
  "css-loader": "^0.25.0",
  "exports-loader": "^0.6.3",
  "expose-loader": "^0.7.1",
  "extract-text-webpack-plugin": "^1.0.1",
  "file-loader": "^0.9.0",
  "imports-loader": "^0.6.5",
  "node-sass": "^3.11.2",
  "sass-loader": "^4.0.2",
  "style-loader": "^0.13.1",
  "url-loader": "^0.5.7",
  "webpack": "^1.13.3",
  "webpack-dev-server": "^1.16.2",
  "webpack-manifest-plugin": "^1.1.0"
},
"babel": {
  "presets": [
    "es2015"
  ],
  "plugins": [
    [
      "transform-runtime",
      {
        "polyfill": false,
        "regenerator": true
      }
    ]
  ]
}
```

# 第二步 - 組織架構

為了讓部署與後續設定單純一點，我們選擇將 javascript 和 css 的資源檔抽出原本的目錄，並同時也保留 Rails 預設的相關功能。大略的架構會如下，我們會將所有的前端資源放在 `client` 目錄底下，當然您要取名叫 `webpack` 或 `frontend` 也都是OK的。

```bash
.
├── /app
│   ├── /assets
│   ├── /controllers
│   ├── /views
│   └── ...
├── /bin
├── /config
├── /db
├── /public
├── ...
├── /client
│   ├── /fonts
│   ├── /images
│   ├── /javascripts
│   ├── /stylesheets
│   ├── development.config.js
│   ├── production.config.js
└── ...
```

除非您是指令控，不然您可以就使用編輯器建立相關檔案與目錄

```bash
$ mkdir -p client/javascripts
$ mkdir client/fonts
$ mkdir client/images
$ mkdir client/stylesheets
# 新增 Entry Point 檔案
$ touch client/javascripts/application.js
$ touch client/javascripts/home.coffee # 測試 coffee 使用
$ touch client/fonts/.keep
$ touch client/images/.keep
$ touch client/stylesheets/home.scss

$ touch client/development.config.js
$ touch client/production.config.js
```

注意因為這個專案會有 Nodejs 混入，所以您應該要在 `.gitignore` 中增加相關的設定。

```yml
# Node
node_modules
jspm_packages
.npm
.eslintcache
npm-debug.log*
pids
*.pid
*.seed
*.pid.lock
.nyc_output
.grunt
.lock-wscript
build/Release
```

### 簡易的驗證範例

* javascripts/home.coffee

```coffee
console.log "Hello, CoffeeScript!"
```

* stylesheets/home.scss

```css
/* 記得放一張圖片 */
.home-banner {
  background-image: url('../images/banner.png');
}
```

* javascripts/application.js

```js
import styles from '../stylesheets/home.scss'
import Home from './home'
```


# 第三步 - 配置 webpack

### development.config.js

```js
var path = require('path')
var _ = require('lodash')
var webpack = require('webpack')
var assetsPath = path.join(__dirname, '..', 'public', 'assets')
var ExtractTextPlugin = require('extract-text-webpack-plugin')

var config = {
  context: path.join(__dirname, '..'),
  entry: {
    /* 定義進入點與其檔案名稱 */
    application: [
      path.join(__dirname, '/javascripts/application.js')
    ]
  },
  output: {
    path: assetsPath,
    filename: '[name]-bundle.js',
    publicPath: '/assets/'
  },
  resolve: {
    extensions: ['', '.js', '.coffee', '.json']
  },
  debug: true,
  displayErrorDetails: true,
  outputPathinfo: true,
  devtool: 'cheap-module-eval-source-map',
  module: {
    loaders: [
      {
        test: require.resolve('jquery'),
        loader: 'expose?jQuery'
      },
      {
        test: require.resolve('jquery'),
        loader: 'expose?$'
      },
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      },
      {
        test: /\.coffee$/,
        loader: 'coffee'
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)\??.*$/,
        loader: 'url?limit=8192&name=[name].[ext]'
      },
      {
        test: /\.(jpe?g|png|gif|svg)\??.*$/,
        loader: 'url?limit=8192&name=[name].[ext]'
      },
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract('style', 'css')
      },
      {
        test: /\.scss$/,
        loader: ExtractTextPlugin.extract('style', 'css!sass')
      }
    ]
  },
  plugins: [
    new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery'
    }),
    new ExtractTextPlugin('[name]-bundle.css', {
      allChunks: true
    })
  ]
}

module.exports = config
```
### production.config.js

```js
var path = require('path')
var _ = require('lodash')
var webpack = require('webpack')
var assetsPath = path.join(__dirname, '..', 'public', 'assets')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var ManifestPlugin = require('webpack-manifest-plugin')

var config = {
  context: path.join(__dirname, '..'),
  entry: {
    application: path.join(__dirname, '/javascripts/application.js')
  },
  output: {
    path: assetsPath,
    filename: '[name]-bundle-[chunkhash].js',
    publicPath: '/assets/'
  },
  resolve: {
    extensions: ['', '.js', '.coffee', '.json']
  },
  debug: true,
  displayErrorDetails: true,
  outputPathinfo: true,
  devtool: 'cheap-module-eval-source-map',
  module: {
    loaders: [
      {
        test: require.resolve('jquery'),
        loader: 'expose?jQuery'
      },
      {
        test: require.resolve('jquery'),
        loader: 'expose?$'
      },
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      },
      {
        test: /\.coffee$/,
        loader: 'coffee'
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)\??.*$/,
        loader: 'url?limit=8192&name=[name]-[hash].[ext]'
      },
      {
        test: /\.(jpe?g|png|gif|svg)\??.*$/,
        loader: 'url?limit=8192&name=[name]-[hash].[ext]'
      },
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract('style', 'css')
      },
      {
        test: /\.scss$/,
        loader: ExtractTextPlugin.extract('style', 'css!sass')
      }
    ]
  },
  plugins: [
    new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery'
    }),
    new ExtractTextPlugin('[name]-bundle-[chunkhash].css', {
      allChunks: true
    }),
    new ManifestPlugin({
      fileName: 'client_manifest.json'
    })
  ]
}

module.exports = config

```

完成之後執行，測試看看是否有地方錯誤。如果您對於 webpack 並不陌生可以閱讀設定檔理解一下。

```bash
$ webpack --config client/development.config.js
$ webpack --config client/production.config.js
```

查看 `public/assets` 目錄底下看看編譯的結果。

# 第四步 - 整合 Rails

到此您應該已經理解，我們就是將前端的部份交給 webpack 處理，然後遵循 Rails 的架構將最終的資源檔編譯輸出到 `public/assets` 。

現在的問題是，我們該如何讀取編譯好的資源呢？

由於我們希望能夠盡可能的遵循 Rails 的慣例，因此下一步我們在  `app/views/layouts/application.html.erb` 放入

```html
<%= client_stylesheet_link_tag 'application' %>
<%= client_javascript_include_tag 'application' %>
```

上面是我們希望的作法（看起來就像是預設支援），所以接著下來我們需要做一些修改好讓 Rails 支援上面兩個 helpers。

首先因為 webpack 加上 hash 的編譯結果，Rails 並無法得知對應的檔案。於是我們需要在 production.config.js 使用 `ManifestPlugin` 匯出 `manifest` 讓 Rails 得知如何對應檔案。

* config/application.rb

```rb
require_relative 'boot'

require 'rails/all'

Bundler.require(*Rails.groups)

module RailsWepback
  class Application < Rails::Application
    config.webpack = {
      asset_manifest: {}
    }
  end
end
```

* 新增 config/initializers/webpack.rb

```rb
asset_manifest = Rails.root.join('public', 'assets', 'client_manifest.json')

if File.exist?(asset_manifest)
  Rails.configuration.webpack[:manifest] = JSON.parse(
    File.read(asset_manifest)
  ).with_indifferent_access
end
```

完成這些前置作業之後我們可以讀取到 `manifest` 了，但您知道的；原生的 Rails 並沒有剛剛那兩個 helpers ，我們需要在 `app/helplers/application_helper.rb` 加上

```rb
def client_javascript_include_tag(name)
  filename = "#{name}-bundle.js"
  asset_url = Rails.application.config.asset_host
  src = "#{asset_url}/assets/#{filename}"

  if Rails.env.development?
  elsif Rails.configuration.webpack[:manifest]
    asset_name = Rails.configuration.webpack[:manifest]["#{name}.js"]
    if asset_name
      src = "#{asset_url}/assets/#{asset_name}"
    end
  end
  "<script src=\"#{src}\"></script>".html_safe
end

def client_stylesheet_link_tag(name)
  filename = "#{name}-bundle.css"
  asset_url = Rails.application.config.asset_host
  src = "#{asset_url}/assets/#{filename}"

  if Rails.env.development?
  elsif Rails.configuration.webpack[:manifest]
    asset_name = Rails.configuration.webpack[:manifest]["#{name}.css"]
    if asset_name
      src = "#{asset_url}/assets/#{asset_name}"
    end
  end
  "<link rel=\"stylesheet\" href=\"#{src}\">".html_safe
end
```

這上面的 `asset_host` 是為了讓事情單純一點，我們選擇在 `config/environments/development.rb` 和 `production.rb` 加上

```rb
# 路徑請依據實際的狀況調整
config.action_controller.asset_host = '127.0.0.1:3000'
```

您可以採取任何您覺得更好的方式取得路徑。

# 完成第一階段

到這一步，其實我們已經完成最上面我們列出的需求了。

先在 terminal 中執行

```bash
$ webpack --config client/development.config.js --watch
```

開啟另一個 session 執行

```bash
$ rails server
```

最後在 `views/pages/index.html.erb` 中的任一 tag 補上 `class="home-banner"` 您就可以觀察到變化。webpack 在背後一直觀察 `client` 底下檔案的變化，Rails 的開發伺服器則負責原來的工作，載入那些編譯後的檔案。

只是這樣每一次要測試都要打兩條指令很麻煩。

# 優化開發指令

* 新增 npm scripts

再往下走之前，我們可以先把 webpack 的指令與其單配的參數先整理到 `npm script`

```json
"scripts": {
  "build:dev": "webpack --config=client/development.config.js --display-reasons --display-chunks --progress --color --watch",
  "build": "webpack --config=client/production.config.js -p"
}
```

接著為了讓我們往後只使用一道指令就能夠輕鬆寫意的開發。最簡單的方式就是使用 `foreman`

```bash
$ gem install foreman
```

安裝完 foreman 之後我們需要設定 `Procfile` 讓其為我們同時啟動兩道指令。


* 新增 Procfile.dev

`Procfile support` 類型程式預設使用 `Procfile` 為設定檔，如果直接使用該檔案可能會遇到問題，例如：當我們要使用 Heroku 的話，後續可能在部署的時候產生問題，主要是目前的設定僅限於開發階段使用，於是我們改使用其他的檔名 `Procfile.dev`。

在專案跟目錄下新增 `Procfile.dev`

```yml
web: bundle exec rails server -p 3000
webpack: npm run build:dev
```

* 使用 foreman

```bash
$ foreman s -f Procfile.dev
```

啟動後您應該看到類似的訊息：

```bash
18:30:56 web.1     | started with pid 16693
18:30:56 webpack.1 | started with pid 16694
18:30:57 webpack.1 |
18:30:57 webpack.1 | > example@1.0.0 build:dev /Users/andyyou/Workspace/sandbox/rails_vuejs_integrate_1/example
18:30:57 webpack.1 | > webpack --config=client/development.config.js --display-reasons --display-chunks --progress --color --watch
18:30:57 webpack.1 |
Hash: 2fdcbe01c557b347442e
18:31:00 web.1     | => Booting Puma
18:31:00 webpack.1 | Version: webpack 1.13.3
18:31:00 web.1     | => Rails 5.0.0.1 application starting in development on http://localhost:3000
18:31:00 webpack.1 | Time: 2893ms
```

# 支援 Hot Reload

基本上到了上一步就已經可以滿足大多數的開發情境，不過您可能也聽多了關於 Hot Replacement Mode (HRM) 的優點，如果我們也想支援呢？

原理上很單純，我們只需要讓前端資源檔換成是由 webpack-dev-server 所提供即可。

* 新增 devserver.config.js

注意到基本上這邊只有加入 `webpack/hot/dev-server` 和 `publicPath` 不同的差異，可以有更精簡的方式，不過這邊為了讓之後維護比較明顯直覺一點所以將其獨立一個檔案：

```js
var path = require('path')
var _ = require('lodash')
var webpack = require('webpack')
var assetsPath = path.join(__dirname, '..', 'public', 'assets')
var ExtractTextPlugin = require('extract-text-webpack-plugin')

var config = {
  context: path.join(__dirname, '..'),
  entry: {
    application: [
      'webpack/hot/dev-server',
      path.join(__dirname, '/javascripts/application.js')
    ]
  },
  output: {
    path: assetsPath,
    filename: '[name]-bundle.js',
    publicPath: 'http://localhost:8080/assets/'
    /* publicPath: '/assets/' */
  },
  resolve: {
    extensions: ['', '.js', '.coffee', '.json']
  },
  debug: true,
  displayErrorDetails: true,
  outputPathinfo: true,
  devtool: 'cheap-module-eval-source-map',
  module: {
    loaders: [
      {
        test: require.resolve('jquery'),
        loader: 'expose?jQuery'
      },
      {
        test: require.resolve('jquery'),
        loader: 'expose?$'
      },
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      },
      {
        test: /\.coffee$/,
        loader: 'coffee'
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)\??.*$/,
        loader: 'url?limit=8192&name=[name].[ext]'
      },
      {
        test: /\.(jpe?g|png|gif|svg)\??.*$/,
        loader: 'url?limit=8192&name=[name].[ext]'
      },
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract('style', 'css')
      },
      {
        test: /\.scss$/,
        loader: ExtractTextPlugin.extract('style', 'css!sass')
      }
    ]
  },
  plugins: [
    new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery'
    }),
    new ExtractTextPlugin('[name]-bundle.css', {
      allChunks: true
    })
  ]
}

module.exports = config

```

* 增加 npm scripts

```js
"dev": "webpack-dev-server --config=client/devserver.config.js --inline --hot --no-info"
```

* 更新 app/helpers/application.rb

因為目前有兩種開發模式，所以我們透過環境變數 `HRM=true` 來區分支不支援 HRM 模式。這邊遭遇的問題就單純是路徑不一樣，您絕對可自行調整優化，而這篇文章旨在記錄這個流程與概念。

```rb
def client_javascript_include_tag(name)
  filename = "#{name}-bundle.js"
  asset_url = Rails.application.config.asset_host
  src = "#{asset_url}/assets/#{filename}"

  if Rails.env.development?
    if ENV["HRM"]
      src = "http://localhost:8080/assets/#{filename}"
    else
      src = src
    end
  elsif Rails.configuration.webpack[:manifest]
    asset_name = Rails.configuration.webpack[:manifest]["#{name}.js"]
    if asset_name
      src = "#{asset_url}/assets/#{asset_name}"
    end
  end
  "<script src=\"#{src}\"></script>".html_safe
end

def client_stylesheet_link_tag(name)
  filename = "#{name}-bundle.css"
  asset_url = Rails.application.config.asset_host
  src = "#{asset_url}/assets/#{filename}"

  if Rails.env.development?
    if ENV["HRM"]
      src = "http://localhost:8080/assets/#{filename}"
    else
      src = src
    end
  elsif Rails.configuration.webpack[:manifest]
    asset_name = Rails.configuration.webpack[:manifest]["#{name}.css"]
    if asset_name
      src = "#{asset_url}/assets/#{asset_name}"
    end
  end
  "<link rel=\"stylesheet\" href=\"#{src}\">".html_safe
end
```

* Procfile.devserver

```yml
web: bundle exec rails server -p 3000
webpack: npm run dev
```

### 使用指令

```bash
# 平常開發模式
$ foreman s -f Procfile.dev

# 支援 Hot Reload
$ HRM=true foreman start -f Procfile.devserver

# Ctrl + C 中止
```

# 漸進式的 Vue.js 是 MVC 框架的好朋友

在 Javascript 當道的今天我想您很容易找到很多關於 SPA 的主流作法。但這篇文章要說明的是把 JS 當作配角的作法。

首先我們需要更新設定檔，使其支援主角 Vue.js v2

```bash
$ npm i vue-loader vue-hot-reload-api -D
```

webpack 所有的 config loader 的部分補上：

```js
{
  test: /\.vue$/,
  loader: 'vue'
}
```

另外 resolve 的部分，因為我們需要 standalone 版本的功能所以需要下面設定：

```js
resolve: {
  extensions: ['', '.js', '.coffee', '.json'],
  /**
   * Vue v2.x 之後 NPM Package 預設只會匯出 runtime-only 版本，若要使用 standalone 功能則需下列設定
   */
  alias: {
    vue: 'vue/dist/vue.js'
  }
}
```

首先新增元件

```bash
$ mkdir client/javascripts/components
$ touch client/javascripts/components/Car.vue
```

* Car.vue 範例程式如下

```html
<script>
export default {
  data () {
    return {
      brand: 'BMW 3 Series',
      mileage: 0
    }
  },

  mounted () {
    this.handle = setInterval(() => {
      this.mileage++
    }, 1000)
  },

  destroyed () {
    clearInterval(this.handle)
  }
}
</script>

<style lang="sass" scoped>
$pink: pink;

.brand {
  color: $pink;
  font-size: 1.4em;
}
</style>
```

* 更新 application.js

```js
import styles from '../stylesheets/home.scss'
import Home from './home'
import Vue from 'vue'
import Car from './components/Car.vue'

document.addEventListener('DOMContentLoaded', function () {
  new Vue({
    el: '#app',
    data: {
      message: 'Hello, Rails with Vue.js'
    },
    components: {
      car: Car
    }
  })
})
```

* app/views/pages/index.html.erb

```html
<div id="app" v-cloak>
  <h1 class="home-banner">{{ message }}</h1>
  <car inline-template>
    <div>
      I am <span class="brand">{{ brand }}</span>! I runned {{ mileage }}.
      The important thing is the variable from controller#action wheel is <%= @wheel %>
    </div>
  </car>
</div>
```

如果您曾使用過 Vue.js 可能會覺得這樣好奇怪，為什麼 component 裡面沒有 `<template>` 。 主要因為 Vue 有支援 `inline-template` 這種作法，這讓我們還可以延續使用從 controller action 來的資料，像是您想繼續使用 Rails 的 i18n 機制。

當然有人可能會說 template 抽到 views 那怎麼重複使用元件，使用 Rails 的 partial 就好了。

重點是這裡不是試圖提出一種萬靈丹，而是提供另一種思路，該怎麼樣使用取決於您的需求。

另外您也可以繼續使用 `slim`

```slim
div id="app" v-cloak=""
  car inline-template=""
    div
        | {{ message }}
        = @wheel
```

# 部署至 Heroku

很高興您能看到這一步，這個過程其實挺累人的。最後當我們完成這一系列的修改，我們還是希望能夠部署到一些方便的服務上，這邊就舉 Heroku 來示範。

第一步我們需要先在 `package.json` 補上一些設定，好讓 Heroku 在安裝完 Node 環境之後可以幫我們執行 `webpack`

```json
"scripts": {
  ...,
  "heroku-postbuild": "npm run build"
}
```

* 部署

```bash
$ heroku login
$ heroku create --app [your_app_name]
$ heroku buildpacks:clear
$ heroku buildpacks:set heroku/nodejs
$ heroku buildpacks:add heroku/ruby --index 2

# 安裝 package.json devDependencies 的部分
$ heroku config:set NPM_CONFIG_PRODUCTION=false
$ git push heroku master
```

整體來說如果您有這些前端複雜的需求，使用 webpack 可能暫時是不錯的方式。
當然如果您有更優秀的作法也歡迎您給小弟一些建議。

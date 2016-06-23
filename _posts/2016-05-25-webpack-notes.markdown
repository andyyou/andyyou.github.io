---
layout: post
title: 'vue + webpack 起手式'
date: 2016-05-25 12:00:00
categories: js, webpack
---
# 介紹

前端的世界變化之快速，從 2010 開始小弟經歷了 jQuery, Backbone, Angular, 到 React。這一路走來雖然學習到了許多高明開發者融合於框架或函式庫中的智慧，卻也因為不斷快速變化感到疲憊。時至 2016 小弟認為在實務與理想之間取得一個完美平衡的前端框架大概就屬 `vue.js` 了。

當然這前端世界裡並沒有萬能藥可以完美的處理所有問題，不過 `vue.js` 的精美，不只容易與傳統 MVC 框架(Rails, ASP.NET MVC)等結合，當要使用最新的設計模式如 Flux, redux 等也都是沒問題的，再加上易學與一些你肯定能感受到作者從實戰淬煉出來的特性。因此在 2016 我也決定轉戰 `vue.js`。

隨著 Javascript 社群快速的演進，很可怕一個問題是 - 專案的環境設定，關於那些 `tooling` 這不只是 React 的問題，當你想使用 ES2015 的新語法，方便的持續整合與測試，匯入匯出模組時，我們就需要設定這些專案工具。

雖然 vue 本身有提供指令介面 `vue-cli` 讓我們快速建立專案，但對這些相關技術和設定有些瞭解肯定能幫助你執行更多客製的行為。

從頭自己一點一點設定有一些好處:

* 每個專案都有不同的需求，您可以根據自身的需求來設定
* 我們也提到 Javascript(nodejs) 的世界變得很快，如果有局部的套件壞了那我們也比較清楚該怎麼處理
* 直接使用別人的 start-kit 也許會多裝了一堆你不需要的東西

下面的文章會教導我們使用 webpack 設定一個基本的 vue 專案

### Part 1 基本目錄架構

##### 1. 建立專案與 package.json

~~~zsh
$ mkdir [project]
$ cd [project]
$ npm init -y
~~~

我們先把需要的程式與目錄結構準備好，我們的需求是使用 vue + es2015 來開發。第一步在根目錄建立一個 `index.html` 下面是一個簡單的 vue 範例

~~~html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>vue.js</title>
  </head>
  <body>
  	<h3>{{ message }}</h3>
    <script src="dist/build.js"></script>
  </body>
</html>
~~~

注意到兩件事

1. 我們使用 `dist/build.js` 這個檔案在編譯之前是不存在的
2. `{{message}}` 這個語法是 vue.js 處理的

建立 `src` 目錄與 `src/main.js` 檔案

~~~
import Vue from 'vue'

new Vue({
  el: 'body',
  data: {
  	message: "Hello Vue"
  }
})
~~~

到了這一步我們已經完成基本的 vue 專案，但是關於建置編譯的設定我們還未完成。

### Part 2 webapck 建置設定

##### 2. 安裝 webpack, webpack-dev-server 與相關 loaders

~~~
$ npm i webpack webpack-dev-server webpack-merge css-loader style-loader file-loader url-loader babel-core babel-loader babel-plugin-transform-runtime babel-preset-es2015 babel-preset-stage-0 babel-runtime vue-loader vue-html-loader vue-style-loader vue-hot-reload-api -D

# -E 參數會定版，原本會是 ^1.0.0 -> 1.0.0
# webpack: webapck 核心程式
# webpack-dev-server: 開發伺服器
# webpack-merge: 合併設定檔使用
# css-loader: 編譯匯入 css
# style-loader: 把編譯後的 css 整合進 html
# file-loader: 編譯匯入檔案類型的資源
# url-loader: 編譯匯入檔案類型的資源，把檔案轉成 base64 等
# babel-core: ES2015 babel 編譯核心
# babel-loader: 編譯匯入 ES2015 類型的檔案
# babel-plugin-transform-runtime: polyfilling
# babel-preset-es2015: es2015 語法
# babel-preset-stage-0: 開啟草稿階段的功能
# babel-runtime: babel 執行環境
# vue-loader: 編譯匯入 vue 元件檔案
# vue-html-loader: 編譯 vue 的 template 部份
# vue-style-loader: 編譯 vue 樣式部分
# vue-hot-reload-api: Hot reload API for Vue components
~~~

##### 3. 裝完 loaders 後記得要撰寫設定 `webpack.config.js`

根目錄下建立與撰寫 `webpack.config.js`

~~~js
var path = require('path')
module.exports = {
  entry: path.join(__dirname, 'src', 'main.js'),
  output: {
    path: './dist',
    filename: 'build.js'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      }
    ]
  }
}
~~~

根目錄建立 `.babelrc` 簡化 `webpack.config.js`，這是因為 `babel 6` 之後把功能拆散了，要用就要裝。同時也可以用 `.babelrc` 來設定，如果不使用這個檔案我們就需要在 `webapck.config.js` 設定

~~~json
{
  "presets": ["es2015", "stage-0"],
  "plugins": ["transform-runtime"]
}
~~~

上面我們已經完成基本的設定，雖然我們一口氣安裝了很多 loaders 但相關設定我們只先設定了 babel 的部份。到了這一步我們的專案架構已經可以被編譯執行了。

~~~zsh
$ webpack
~~~

編譯之後點擊 `index.html` 即可以運行，眼尖的讀者可能會好奇，那我們剛剛有裝 `vue-loader` 那是在幹嘛的？

### Part 3 使用 vue-loader 與 .vue

`vue-loader` 的用途是提供一種更方便的組織方式讓我們把元件即一個 component 中需要的 js 行為, css 樣式, template 樣板放在一個 `.vue` 的檔案中。

首先讓我們先修改 `index.html` ，加入 `<app>`

~~~html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Vue Example</title>
  </head>
  <body>
    <app></app>
    <script src="dist/build.js"></script>
  </body>
</html>
~~~

接著我們把掛載元素的節點換成 `body` ，然後透過使用標籤 `<app></app>` 把元件掛載到檔案上，`main.js` 修改如下

~~~js
import Vue from 'vue'
import App from './app.vue'

new Vue({
  el: 'body',
  components: { App }
})
~~~

最後我們新增一個 `app.vue` 檔案


~~~
<template>
<div class="message">{{ msg }}</div>
</template>

<script>
export default {
  data () {
    return {
      msg: 'Hello from vue-loader!'
    }
  }
}
</script>

<style>
.message {
  color: blue;
}
</style>
~~~

這個時候如果直接執行 `webpack` 編譯會產生錯誤，因為我們還沒設定 `webpack.config.js` 處理 `.vue` 檔案的部分

~~~js
var path = require('path')

module.exports = {
  entry: path.join(__dirname, 'src', 'main.js'),
  output: {
    path: path.join(__dirname, 'dist'),
    publicPath: 'dist/',
    filename: 'build.js'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      },
      {
        test: /\.vue$/,
        loader: 'vue'
      }
    ]
  }
}
~~~

上面有列出每個套件大略的用途了，不過這邊再次強調一下 `vue-loader` 會相依一些其他的 loaders，並且如果你要在 `.vue` 中使用像是 `sass`, `jade` 的話還會需要安裝其他相依的套件。完整的設定在[官方文件](http://vue-loader.vuejs.org/en/start/tutorial.html)中都有。但這邊我們簡單地列出基本需要的套件

~~~
css-loader
style-loader
vue-loader
vue-html-loader

# 如果要使用 sass
sass-loader
node-sass
~~~

再次編譯，我們的 `app.vue` 可以正常運作了。

### Part 4 HMR / Hot Reload

`Hot Module Replacement` 或稱 `Hot Reload` 是 Javascript 世界中近期很熱門的新技術，簡單的說就是當你在開發時，你一存檔，改寫的部份就即時更新元件到執行環境。大致上流程就是

1. 處於開發 app 階段，撰寫程式碼
2. 打開瀏覽器觀察 app 行為
3. app 在畫面上運作
4. 當你發現一些 bug 你通常會編輯程式碼，然後重新載入
5. 使用 HRM 時，當你一存檔 webpack 就會偵測那些改變的部分並更新瀏覽器
6. 重點是一些關於狀態的資料並不會被洗掉

要完成這功能，我們會需要 `webpack-dev-server` 以及套件 `vue-hot-reload-api`。然後執行。

~~~zsh
$ webpack-dev-server --inline --hot
~~~

打開 `http://localhost:8080` 然後編輯 `app.vue` css 與 data 的部分觀察看看變化。
至此我們已經完成一些基本設定的部份。文章剩下的部份則是整理一些 webpack 的指令與設定。

# webpack 編譯指令

~~~zsh
$ webpack [source] [destination]
$ webpack src/v1.js dist/v1.bundle.js
$ webpack bar=./src/v2.js "dist/[name].bundle.js"
# >> output dist/bar.bundle.js
~~~

~~~zsh
# 指定設定檔
$ webpack --config [webpack.config.js]
~~~

> 注意 require 的 path 分成 `函式庫` `相對路徑` `絕對路徑`
>  * 函式庫：什麼都不加，單純 library name
>  * 相對路徑：`./` 開頭
>  * 絕對路徑：`/` 開頭

# 資源

* [webpack 令人困惑的地方 - 英](https://medium.com/@rajaraodv/webpack-the-confusing-parts-58712f8fcad9#.zap5b4dvo)
* [webpack 令人困惑的地方 - 中](https://segmentfault.com/a/1190000005089993)

# 快速指令與流程參考

~~~zsh
$ npm init -y
$ npm i webpack -DE
# + E 定版，原本會是 ^1.0.0 -> 1.0.0

# Add webpack.config.js
# Add scripts to package.json
# Setup webpack-dev-server

$ npm i webpack-dev-server -D

# 由於 prod & dev 會需要不同的設定因此我們需要至少兩份設定檔
# 有許多實作方式如下：
#   1. 維護多份設定檔，透過 `--config` 指定不同的檔案
#   2. 把設定組織成一份 Library
#   3. 在一份檔案中依據 `環境` 或 `指令` 套用不同設定

# 第三種方式還有方便的 Library 協助 webpack-merge

$ npm i webpack-merge -D
$ npm i css-loader style-loader -D
$ npm i file-loader url-loader -D
# 強大のplugin
$ npm i npm-install-webpack-plugin -D

# 安裝處理 ES2015
$ npm i babel-core babel-loader babel-plugin-transform-runtime babel-preset-es2015 babel-preset-stage-0 -D
$ npm i babel-runtime -S

# 加入 .babelrc
# 加入 loader 設定


# (Optional)安裝 vue-loader
$ npm i vue-loader vue-html-loader vue-style-loader vue-hot-reload-api -D


# Install everything we need from http://j.mp/1TBumSH
$ npm install\
  webpack webpack-dev-server\
  vue-loader vue-html-loader css-loader vue-style-loader vue-hot-reload-api\
  babel-loader babel-core babel-plugin-transform-runtime babel-preset-es2015\
  babel-runtime\
  --save-dev

# 使用 sass
$ npm install sass-loader node-sass --D
~~~

~~~.babelrc
{
  "presets": ["es2015", "stage-0"],
  "plugins": ["transform-runtime"]
}
~~~

~~~
{
  test: /\.js$/,
  loader: 'babel',
  exclude: /node_modules/
}
~~~

~~~
var path = require('path')
var webpack = require('webpack')
var merge = require('webpack-merge')
var T = process.env.npm_lifecycle_event

var common = {
  entry: {
    main: path.join(__dirname, 'src', 'main'),
    venders: path.join(__dirname, 'src', 'venders')
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      },
      {
        test: /\.vue$/,
        loader: 'vue'
      },
      {
        test: /\.css$/,
        loaders: ['style', 'css']
        /* include: path.join(__dirname, 'src') */
      },
      {
        test: /\.scss$/,
        loaders: ['style', 'css', 'sass']
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url',
        query: {
          limit: 10000,
          name: path.posix.join(__dirname, 'public', '[name].[hash:7].[ext]')
        }
      }
    ]
  },
  resolve: {
    extensions: ['', '.js', '.vue', '.json', '.scss', '.css']
  },
}

if (T === 'dev' || !T) {
  module.exports = merge(common, {
    devServer: {
      historyApiFallback: true,
      hot: true,
      inline: true,
      progress: true,
      stats: 'errors-only',
      host: process.env.HOST || '0.0.0.0',
      port: process.env.PORT
    },
    devtool: 'eval-source-map',
    plugins: [
      new webpack.HotModuleReplacementPlugin()
    ]
  })
}

if (T === 'build') {
  module.exports = merge(common, {})
}
~~~

~~~
"scripts": {
  "build": "webpack",
  "dev": "webpack-dev-server"
},
~~~

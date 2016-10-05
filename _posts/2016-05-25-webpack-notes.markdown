---
layout: post
title: 'Vue + webpack 起手式'
date: 2016-10-04 12:00:00
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

這篇文章將會透過實作介紹最基本的概念，使用 webpack 設定一個基本的 vue 專案

### Part 1 基本目錄架構

##### 1. 建立專案與 package.json

~~~zsh
$ mkdir [project_name]
$ cd [project_name]
$ npm init -y
$ npm install vue -S
~~~

我們先把需要的程式與目錄結構準備好，需求是使用 `Vue` + `ES2015` 來開發。第一步在根目錄建立一個 `index.html` 下面是一個簡單的 vue 範例

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Vue.js v2</title>
</head>
<body>
  <div id="app">{{ message }}</div>
  <script src="dist/bundle.js"></script>
</body>
</html>
~~~

注意到兩件事

1. 我們使用 **dist/build.js** 這個檔案在編譯之前是不存在的
2. **{{message}}** 這個語法是 vue.js 處理的

建立 `src` 目錄與 `src/main.js` 檔案，這邊您可以隨您自己的偏好組織專案架構

~~~
import Vue from 'vue'

new Vue({
  el: '#app',
  data: {
  	message: "Hello Vue"
  }
})
~~~

在這一步我們已經完成一個簡單的 Vue 專案，但是關於建置編譯的設定我們還未完成。

~~~
Do not mount Vue to <html> or <body> - mount to normal elements instead.

關於 v2 之後值得注意的地方，便是我們不能直接將元件掛載到 html 或 body 上。
~~~

### Part 2 webapck 建置設定

##### 1. 安裝 webpack, webpack-dev-server 與相關 loaders

> 為了專注在基本的說明，本次更新已將一些不屬於基本功能的模組移除。

~~~
# 2016-10-04 更新
$ npm i webpack webpack-dev-server webpack-merge css-loader style-loader file-loader url-loader babel-loader babel-core babel-plugin-transform-runtime babel-preset-es2015  vue-loader vue-hot-reload-api -D
~~~

~~~js
"dependencies": {
  "vue": "^2.0.1"
},
"devDependencies": {
  "babel-core": "^6.17.0", // babel 核心程式
  "babel-loader": "^6.2.5", // webpack 使用的 babel 編譯器
  "babel-plugin-transform-runtime": "^6.15.0", // 預設 babel 會在每一隻編譯檔案注入 polyfill 的程式碼，為了避免重複而將這部分抽出去。詳細說明：http://babeljs.io/docs/plugins/transform-runtime/
  "babel-preset-es2015": "^6.16.0", // 支援 ES2015 語法
  "css-loader": "^0.25.0", // webpack 使用於處理 css
  "file-loader": "^0.9.0", // webpack 使用於處理檔案
  "style-loader": "^0.13.1", // webpack 將 css 整合進元件中
  "url-loader": "^0.5.7", // 編譯匯入檔案類型的資源，把檔案轉成 base64
  "vue-hot-reload-api": "^2.0.6", // 支援 Hot Reload
  "vue-loader": "^9.5.1", // 使用 Vue Component Spec
  "webpack": "^1.13.2",
  "webpack-dev-server": "^1.16.1", // webpack 開發伺服器
  "webpack-merge": "^0.14.1" // 合併 webpack 設定參數
}
~~~

##### 2. 裝完 loaders 後，撰寫設定 `webpack.config.js`

根目錄下建立與撰寫 `webpack.config.js`

~~~js
var path = require('path')
var config = {
  entry: path.join(__dirname, 'src', 'main'),
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: '/dist/'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      }
    ]
  },
  resolve: {
    extensions: ['', '.js', '.vue'],
    /**
     * Vue v2.x 之後 NPM Package 預設只會匯出 runtime-only 版本，若要使用 standalone 功能則需下列設定
     */
    alias: {
      vue: 'vue/dist/vue.js'
    }
  }
}

module.exports = config
~~~


> Failed to mount component: template or render function not defined. (found in root instance)

~~~
若您看到上面錯誤訊息，這是由於 Vue 2 之後分成 standalone 完整版與 runtime-only 版。差異在於完整版包含了編譯器，支援 template 以及使用了瀏覽器的 API。

而 NPM 模組預設只會匯出 runtime-only ，若要加入 compiler 和 template 支援則需增加 webpack 的設定。
~~~

##### 3. 設定 babel 的部分

根目錄建立 `.babelrc` 簡化 `webpack.config.js`，這是因為 `babel 6` 之後把功能拆散了，要用就要裝。同時也可以用 `.babelrc` 來設定，如果不使用這個檔案我們就需要在 `webapck.config.js` 設定。

`.babelrc`

~~~json
{
  "presets": ["es2015"],
  "plugins": ["transform-runtime"]
}
~~~

> 另外 `package.json` 和環境變數也能夠設定，不過為了單純起見我們選擇建立 .babelrc 。當然您也可以選擇設定在 package.json 中。

`package.json`

~~~json
{
  "name": "YOUR PROJECT NAME",
  ...,
  "babel": {
    "presets": [
      "es2015"
    ],
    "plugins": [
      "transform-runtime"
    ]
  }
}
~~~

上面我們已經完成基本的設定，雖然我們一口氣安裝了很多 loaders 但相關設定我們只先設定了 babel 的部份。到了這一步我們的專案架構已經可以被編譯執行了。

~~~bash
# 如果在這一步您想先執行編譯看看可以安裝全域的 webpack
$ npm i webpack -g
$ webpack
$ open index.html # 編譯後檢視內容
~~~

編譯之後點擊 `index.html` 即可以運行。眼尖的讀者可能會好奇，那我們剛剛有裝 `vue-loader` 那是在幹嘛的？

### Part 3 使用 vue-loader 與 .vue

`vue-loader` 的用途是提供一種更方便的組織方式讓我們把元件即一個 component 中需要的 js 行為, css 樣式, template 樣板放在一個 `.vue` 的檔案中。

##### 1. 修改 view

首先讓我們先修改 `index.html` ，加入 `<app>`

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Vue.js v2</title>
</head>
<body>
  <div id="app">
    <app></app>
  </div>

  <script src="dist/bundle.js"></script>
</body>
</html>
~~~

##### 2. 匯入元件

接著我們在 `main.js` 把元件 `app.vue` 加入 components，在這邊我們是反向的推導回去，從`想`怎麼使用接著反著建立程式檔案。

~~~js
import Vue from 'vue'
import App from './app.vue'

new Vue({
  el: '#app',
  components: { App }
})
~~~

##### 3. 新增元件

最後我們新增一個 `app.vue` 檔案

~~~html
<template lang="html">
  <div>
    <div class="message">
      {{ message }}
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      message: 'Helo, Vue.js 2.0'
    }
  }
}
</script>

<style lang="css">
.message {
  color: pink;
  font-size: 1.4em;
}
</style>
~~~

##### 4. 更新 webpack.config.js

這個時候如果直接執行 `webpack` 編譯會產生錯誤，因為我們還沒設定 `webpack.config.js` 處理 `.vue` 檔案的部分

~~~js
var path = require('path')

var config = {
  entry: [
    'webpack/hot/dev-server',
    path.join(__dirname, 'src', 'main')
  ],
  output: {
    publicPath: '/dist/',
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js'
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
  },
  resolve: {
    /**
     * Vue v2.x 之後 NPM Package 預設只會匯出 runtime-only 版本
     */
    alias: {
      vue: 'vue/dist/vue.js'
    },
    extensions: ['', '.js', '.vue']
  }
}

module.exports = config
~~~

再次執行 `webpack` 編譯，我們的 `app.vue` 可以正常運作了。

### Part 4 HMR / Hot Reload

> 如果您有發現修改之後卻沒有改變的問題，請注意關於路徑部分，取得的是編譯後的實體檔案還是 webpack-dev-server 使用記憶體中的內容

`Hot Module Replacement` 或稱 `Hot Reload` 是 Javascript 世界中近期很熱門的新技術，簡單的說就是當你在開發時，你一存檔，改寫的部份就即時更新元件到執行環境。大致上流程就是

1. 處於開發 app 階段，撰寫程式碼
2. 打開瀏覽器觀察 app 行為
3. app 在瀏覽器畫面上運作
4. 當你發現一些 bug 或行為不如您所預期您通常會編輯程式碼，然後重新載入
5. 使用 HRM 時，當你一存檔 webpack 就會偵測那些改變的部分並更新瀏覽器
6. 重點是一些關於`狀態`的資料並不會被洗掉

要完成這功能，我們會需要 `webpack-dev-server` 以及套件 `vue-hot-reload-api`。然後執行。

在這之前，我們需要修改一下 webpack.config.js 加入 `webpack/hot/dev-server`

```js
var webpack = require('webpack')

var config = {
  entry: [
    'webpack/hot/dev-server',
    path.join(__dirname, 'src', 'main')
  ],
  ...
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
}
```

接著，您可以選擇在全域安裝 webpack-dev-server

~~~bash
$ npm i webpack-dev-server -g
$ webpack-dev-server --inline --hot
~~~

又或者使用我們早先已安裝在專案中的 webpack-dev-server，一般來說會建議使用專案相依的這個。

需要在 package.json 加上 scripts

```js
"scripts": {
  "dev": "webpack-dev-server"
}
```

然後執行：

```bash
$ npm run dev
```

為了觀察出我們是否有正確的啟用 hot reload 我們修改 `app.vue`

```html
<template lang="html">
  <div>
    <div class="message">
      {{ message }}
    </div>
    <div>
      {{ count }}
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      message: 'Helo, Vue.js 2.0',
      count: 0
    }
  },

  mounted () {
    this.handle = setInterval(() => {
      this.count++
    }, 1000)
  },

  destroyed () {
    clearInterval(this.handle)
  }
}
</script>

<style lang="css">
.message {
  color: pink;
  font-size: 1.4em;
}
</style>
```

打開 `http://localhost:8080` 然後異動 `app.vue` css 與 template 的部分觀察看看變化。
至此我們已經跑完一次基本的用法。

文章剩下的部份則是整理一些 webpack 的指令與設定。

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
* [補充關於 Babel](http://www.ruanyifeng.com/blog/2016/01/babel.html)
* [一步一步設定 webpack Demo](https://github.com/andyyou/vue-dinner-demo)

# 快速指令流程 & 程式碼片段

~~~bash
$ npm init -y
$ npm i webpack -D

# Add webpack.config.js
# Add scripts to package.json
# Setup webpack-dev-server

$ npm i webpack-dev-server -D

# 由於 prod & dev 會需要不同的設定因此我們需要至少兩份設定檔
# 有許多實作方式如下：
#   1. 維護多份設定檔，透過 `--config` 指定不同的檔案
#   2. 把設定組織成一份 Library
#   3. 在一份檔案中依據 `環境` 或 `指令` 套用不同設定
#      使用 webpack-merge，合併設定更方便

$ npm i webpack-merge -D
$ npm i css-loader style-loader -D
$ npm i file-loader url-loader -D

# 強大のplugin
# npm i npm-install-webpack-plugin -D

# 安裝處理 ES2015
$ npm i babel-core babel-loader babel-plugin-transform-runtime babel-preset-es2015 -D

# 加入 .babelrc
# 加入 loader 設定

# (Optional)安裝 vue-loader
$ npm i vue-loader vue-hot-reload-api -D

# 安裝 vue 與所需的模組
~~~

~~~rc
{
  "presets": ["es2015"],
  "plugins": ["transform-runtime"]
}
~~~

~~~js
var path = require('path')
var webpack = require('webpack')
var merge = require('webpack-merge')
var precss = require('precss');
var autoprefixer = require('autoprefixer');
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
        loaders: ['style', 'css', 'postcss']
        /* include: path.join(__dirname, 'src') */
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
    extensions: ['', '.js', '.vue', '.json', '.css']
  },
  postcss: function () {
    return [precss, autoprefixer];
  }
}

if (T === 'dev' || !T) {
  var config = merge(common, {
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

  config.entry.main = ['webpack/hot/dev-server', config.entry.main]
  module.exports = config
}

if (T === 'build') {
  module.exports = merge(common, {})
}
~~~

~~~
"scripts": {
  "build": "webpack",
  "dev": "webpack-dev-server"
}
~~~

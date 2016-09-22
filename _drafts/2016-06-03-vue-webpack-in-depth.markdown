---
layout: post
title: 'Webpack 搭配 Vue 的前世今生'
date: 2016-06-03 12:00:00
categories: js
---

# 前言

![](http://i.imgur.com/GX3ieSW.jpg)

* 在開始之前，我想請問在場的各位，在選擇 tooling 這些專案的建置工具時是有思想的請舉手？我的意思是經過獨立思考`為什麼`才選擇使用，而不是因為某個框架預設使用，某個大神使用。

# HTML basic knowledge with assets

今天我希望可以透過分享，讓你不只限於 Vue + Webpack ，甚至任何前端框架 + 工具您都能夠處理。先回到本質上，對於前端來說其實我們要處理的離不開下面四個主角

![](http://pic.pimg.tw/kaiandgreen/1365230914-2518545439_m.jpg?v=1365230915)

* 前端的四個主角
  + HTML
  + CSS
  + JS
  + Assets(Images, Fonts)

但為什麼變得這麼複雜...

* 想要使用 jade, ejs (HTML)
* 想要使用 sass, less, postcss, etc
* 想要使用 es2015, typescript, [Compile to JS 語言](https://github.com/jashkenas/coffeescript/wiki/list-of-languages-that-compile-to-js)
* 想要模組化，解決 <script> 載入與執行順序(相依性)的問題，管理與安裝函式庫與模組
  模組，處理相依性，模組的匯入匯出 = 分拆程式碼與檔案架構
* 需要解決過多請求的問題
* js 想要混淆的功能
* 想要優化，壓縮
* 各種元件化的 pattern 與封裝
* debug 與 livereload, hot reload 的功能
* Lint
* 自動化測試
* etc...

於是乎拯救世界的勇者出現了

![](http://c.share.photo.xuite.net/b93301032/1cb86e0/13572933/704809627_m.jpg)

# Introduce tooling Grunt, jspm, gulp, webpack

這一個章節我們試著釐清為什麼選擇 webpack 的脈絡，前端程式的部分在過去的專案開發中，通常只需要單純的合併 Javascript 檔案。不過世界已經變了，今天要處理發佈 Javascript 程式碼已經不再像從前那樣單純。

### Task Runners 和 Bundlers

從過去的歷史看來，的確有過許多建置工具。`Make` 應該是最廣為人知的，而且至今仍然被廣泛採用。為了簡化流程與需求於是像 Grunt 和 gulp 這類任務管理執行工具就出現了。透過 npm 安裝其外掛套件的方式讓這一類的任務執行工具變得非常強大。

所謂任務管理執行工具，我們可以理解為把一系列的指令濃縮為一道指令，任務管理執行工具其中最棒得點就是，讓我們可以解決不同平台之間`做一樣的事，但需要不同指令`的問題，即`使用一樣的指令，做一樣的事`。

常見的任務包含:

* 壓縮圖片
* 編譯
* Uglify
* Lint
* 合併

透過這些任務工具，我們的確把 JS 從一堆檔案合併成一隻：

```html
<html>
  <head>
    <!-- Before -->
    <script src=".../module/1"></script>
    <script src=".../module/2"></script>
    <script src=".../module/3"></script>

    <!-- After -->
    <script src="bundle"></script>
  </head>
</html>
```

但我們希望在開發時可以組織檔案架構，將程式碼模組化，然後透過引用匯入的方式使用模組，最後匯整 bundle。
於是乎我們有了一些模組封裝工具(Bundler) Browserify, Brunch, RequireJS。

然後，因為元件化的開發架構我們希望可以把 CSS，圖片這些資源檔也一併整合進去於是又有了 Webpack。

接著下來，我們還希望可以讓所謂的套件管理機制直接就進瀏覽器。不需要事先編譯，就可以直接在瀏覽器中使用匯入的語法來處理模組。
JSPM 出現了，底層使用 System.js 本質上是一個動態的 Module Loader，另外 Webpack 2 將也會支援 System.js 語法 `System.import`。

這就是我們一路走來概略的過程。

### Make

![](https://upload.wikimedia.org/wikipedia/en/thumb/2/22/Heckert_GNU_white.svg/220px-Heckert_GNU_white.svg.png)

Make 第一次發佈於 1977 年，您可能會說這是一個古老的工具了，但其仍適用於很多情境。Make 可以協助我們執行各式各樣不同的任務。舉例來說您可能有個獨立的任務是針對編譯正式產品的程式包含了壓縮程式碼與執行測試。雖然 Make 大多被用在 C 的專案，不過並不侷限只能用在特定領域。

### Grunt 10900+

![](http://gruntjs.com/img/grunt-logo.png)

```
module.exports = function(grunt) {
	grunt.initConfig({
		pkg: grunt.file.readJSON('package.json'),
		concat: {
			options: {
  			separator: ';'
			},
			dist: {
  			src: ['src/**/*.js'],
  			dest: 'dist/<%= pkg.name %>.js'
			}
		},
		uglify: {
			options: {
				banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */\n'
			},
			dist: {
				files: {
  				'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
				}
			}
		},
		qunit: {
			files: ['test/**/*.html']
		},
		jshint: {
			files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
			options: {
				globals: {
  				jQuery: true,
  				console: true,
  				module: true,
  				document: true
				}
			}
		},
		watch: {
			files: ['<%= jshint.files %>'],
			tasks: ['jshint', 'qunit']
		}
  });

	grunt.loadNpmTasks('grunt-contrib-uglify');
	grunt.loadNpmTasks('grunt-contrib-jshint');
	grunt.loadNpmTasks('grunt-contrib-qunit');
	grunt.loadNpmTasks('grunt-contrib-watch');
	grunt.loadNpmTasks('grunt-contrib-concat');

	grunt.registerTask('test', ['jshint', 'qunit']);
	grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);
};
```

在 gulp 出現之前，Grunt 是 Javascript task runner 的主流工具。透過外掛套件搭配設定檔的架構曾經非常流行。不過套件的使用與開發相對複雜隨著我們需要的特殊任務日益增加就變得越來越難理解其運作。

缺點：

* Grunt 的任務需要取決於 plugin 作者
* 處理連續性的任務不方便，通常任務跟任務之間的內容需靠檔案實體來傳遞
* 單純的任務工具不支援模組機制


### gulp 22000+

![](http://survivejs.com/webpack/images/gulp.png)

gulp 使用了一種不同的思路取代透過`設定屬性與套件`的方式，讓我們透過`組織程式邏輯`的方式，主要的核心概念就是 `pipe` 如果您熟悉 Linux 那麼先看看下面這段指令

```bash
ls -al | grep whoami
```

> 想成大隊接力吧！

從來源的檔案出發，先取得來源，然後執行需要的任務例如：編譯 compile to js 語言。最終這些結果會輸出到我們的 `build` 資料夾。

```js
gulp.src(paths.scripts)
  .pipe(sourcemaps.init())
    .pipe(coffee())
    .pipe(uglify())
    .pipe(concat('all.min.js'))
  .pipe(sourcemaps.write())
  .pipe(gulp.dest('build/js'));
```

這種`程式即設定`的方式對開發者來說非常友善，您可以很清楚知道到底這些任務的先後，並且很容易在過程中加上您想要處理的任何事情。

* [fly](https://github.com/brj/fly) 類似於 gulp 不過採用 ES2015 的 generator 語法

### Browserify 10000+

![](http://survivejs.com/webpack/images/browserify.png)

處理 Javascript 模組一直以來都是一個問題，主要是因為在 ES6 之前 JS 這個語言沒有模組的概念。接著因為模組化的需求 CommonJS, AMD 出現了。

而 Browserify 是其中一個解決方案，主要提供了前端可以使用 CommonJS 規範，簡單的說就是讓我們可以在瀏覽器的環境(前端)下使用 `require('module')`。接著在匯入，轉換的過程中可以使用一些 [transform 套件](https://github.com/substack/node-browserify/wiki/list-of-transforms) 例如 `watchify` ，啟用檔案監控，當檔案發生改變時編譯，讓我們節省一些時間。

Browserify 本身沒有具備太多功能，核心就是處理`require 模組`。

### Brunch 5406

![](http://i.imgur.com/sqonKAr.png)

相較於 gulp 讓我們直接定義任務， Brunch 使用高度抽象化的概念，有點類似 webpack。

```
module.exports = {
  files: {
    javascripts: {
      joinTo: {
        'vendor.js': /^(?!app)/,
        'app.js': /^app/
      }
    },
    stylesheets: {
      joinTo: 'app.css'
    }
  },
  plugins: {
    babel: {
      presets: ['es2015']
    },
    postcss: {
      processors: [require('autoprefixer')]
    }
  }
};
```
### jspm 3200+

jspm 跟目前我們看到的這些工具有些不同，基本上上面的工具都需要事先編譯，就是把我們這些模組的寫法編譯成一隻瀏覽器使用的 JS 進而取代一堆 <script> 的寫法。而 jspm 採用 SystemJS ，白話文就是讓我們直接在瀏覽器環境下 `require` 而不需要事先編譯(不考慮效能)，並支援幾乎所有的標準。

本質上是一個指令工具協助我們管理套件(npm, Github, etc)與產生 system.js 需要的設定(初始化專案, 封裝, plugin 類似 webpack-loader)。


看起來 jspm 好像非常有潛力，但我們有些現實需要考量...

* 遲遲無法增加的用戶與社群的大力支援
* ... webpack 2 預計也會支援 system.js

### webpack 17000+

browserify 強化種，不只處理 js 模組，連 css，資源檔也一併能夠使用 require，透過 loader + plugin 可以完成 gulp 能處理的大部分工作

webpack 簡單說就是一個模組的封裝工具，由德國的 Tobias Koppers 所開發。webpack 會將模組與其相依性的模組, 函式庫, 其他需要預先編譯的檔案等整合產生此模組的靜態資源檔。

### 小結

* npm 套件安裝管理
* grunt, gulp 任務執行
* requirejs 模組
* browserify, brunch, webpack  模組+任務
* jspm 模組+任務+動態載入
* webpack2, rollup 模組+任務+Tree-Shaking

偏好: 小型專案使用 browserify，大型專案使用 webpack，jspm 觀望...。

# 為什麼選擇 webpack 與模組化的價值

首先的問題是為什麼我們要用 webpack 而不用 gulp 或 Grunt? 這顯然不是一個有明確正解的答案。但對我來說最重要的原因是處理模組載入與封裝的功能。因為...

+ 模組化
  * 命名衝突
  > There are only two hard things in Computer Science: cache invalidation and naming things. -- Phil Karlton

  Utils/Helpers -> namespace

  * 相依性

  您可能沒注意到其實 Grunt 也有 `grunt-webpack` 這個套件。不過對我來說其實 webpack 就能辦到了就不需要再多一個 gulp 或 Grunt。

+ Hot Module Replacement (Livereload 強化版)
  您一定有過這種經驗，比如開發一個 Todo 每一次一修改 livereload 就會把剛剛測試填入的資料洗掉...
+ Bundle assets too
+ 分拆 bundle

  ```
  var config = {
    entry: {
      main: path.join(__dirname, 'src', 'main'),
      venders: path.join(__dirname, 'src', 'venders')
    }
  }
  ```

+ loaders
+ plugins
  e.g
    1. 可以載入 dll(windows)
    2. 輸出 css 檔案(文字檔案)
    3. 優化, 壓縮, 混淆, 抽出重複使用的部分
    4. 產生 HTML5 Cache mainfest
    5. Offline 功能(ServiceWorker)
    6. 圖片壓縮
    7. 熱替換 HRM
    8. 上傳 s3
    9. etc...

+ vue official support

# Getting start from webpack

![](http://webpack.github.io/assets/what-is-webpack.png)
![](https://dtinth.github.io/webpack-docs-images/usage/how-it-works.png)

### 安裝 & 基本觀念

```
# 安裝 webpack
$ npm i webpack webpack-dev-server -D
```

```
$ webpack [entry] [output]
```

因為指令越來越長，所以用了 webpack.config.js

```
// 設定 webpack.config.js
var path = require('path')
var config = {
  entry: {},
  output: {},
  module: { loaders: [...] }
}

module.exports = config
```

```
$ webpack
$ webpack-dev-server
```

* 自此之後我們會從進入點(entry point)載入所需要的包含 css, js, 甚至是圖片
* 而 HTML 只需要引用 bundle

# With Vue.js

重點在於先想好專案架構！

我的步驟：

1. 組織專案架構，選擇想要使用的工具，語言，函式庫，框架...，思考需要的任務指令
2. 佈置好最小的可執行範例
3. 安裝工具與套件
4. 撰寫設定檔
5. 執行

關於 webpack 第二個困難點，擁有多種設定方式

* 指令
* 設定
  + 單一設定檔，依據環境與動態變數調整
  + 多個設定檔
* 整合開發伺服器

# 實際步驟

```
$ mkair vuedinner
$ cd vuedinner
$ npm init -y
$ npm i webpack webpack-dev-server -D
```

```
$ npm i vue -S
$ touch index.html
$ mkdir src
$ touch src/main.js
```

```
# Init index.html
# Init main.js
# 還不能使用 ES2015
```

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Vue Dinner</title>
</head>
<body>
  {{msg}}
  <script src="bundle.js"></script>
</body>
</html>
```

```
var Vue = require('vue')

new Vue({
  el: "body",
  data: {
    msg: "Hi, vue.js"
  }
})
```

```
$ webpack-dev-server --entry=./src/main.js
```

使用 babel 與開始設定 webpack.config.js

```
$ npm i webpack-merge css-loader style-loader file-loader url-loader babel-core babel-loader babel-plugin-transform-runtime babel-preset-es2015 babel-runtime vue-loader vue-html-loader vue-style-loader vue-hot-reload-api -D

# webpack: webapck 核心程式
# webpack-dev-server: 開發伺服器
# webpack-merge: 合併設定檔使用
# css-loader: 編譯匯入 css
# style-loader: 把編譯後的 css 整合進 html
# file-loader: 編譯匯入檔案類型的資源
# url-loader: 編譯匯入檔案類型的資源，把檔案轉成 base64 等
# babel-core: ES2015 babel 編譯核心
# babel-loader: 編譯處理匯入 ES2015 類型的檔案
# babel-plugin-transform-runtime: polyfilling 套件(相依 babel-runtime)
# babel-preset-es2015: es2015 語法
# babel-preset-stage-0: 開啟草稿階段的功能
# babel-runtime: ES2015+ 的支援 helpers, polyfilling 函式庫
  例如  `class MyClass {}` -> `_classCallCheck` 在處理 AST 時使用對應的 module 取代(from babel-runtime)
# vue-loader: 編譯匯入 vue 元件檔案
# vue-html-loader: 編譯 vue 的 template 部份
# vue-style-loader: 編譯 vue 樣式部分
# vue-hot-reload-api: Hot reload API for Vue components
```

* 新增 webpack.config.js
* Component 使用 ES2015 語法
* 加入 npm script

```
var path = require('path')

var config = {
  entry: path.join(__dirname, 'src', 'main.js'),
  output: {
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/,
        query: {
          presets: ['es2015'],
          plugins: ['transform-runtime']
        }
      }
    ]
  }
}

module.exports = config
```

```
"dev": "webpack-dev-server",
"build": "webpack -p --progress --hide-modules"
```

# vue-loader

簡單說，vue-loader 就是讓我們可以用下圖這種 `*.vue` 格式撰寫元件的套件

![](http://blog.evanyou.me/images/vue-component.png)

* 預設支援 ES2015
* style 和 template 可以換其他語言
* css scoped
* 額外的好處 weex

`*.vue` 是一種特殊的檔案格式，使用類似 HTML 的語法來撰寫 Vue 元件，每一個 `.vue` 檔案由三個 tag 組成 `<template>` `<script>` `<style>`

`vue-loader` 會解析檔案擷取每種語言中的內容，然後如果有需要則交給其他 loader 把它們在組回 js 最後透過 `module.exports` 匯出。
vue-loader 除了預設 template(HTML), style(css), script(JS) 外還支援其他的 compile to 語言，例如：透過 `lang` 可以指定 style 使用 sass

```
$ npm install sass-loader node-sass -D
```

```
<style lang="sass">
  /* write SASS! */
</style>
```

* <template>
  + 預設 HTML
  + 每一個 vue 檔案一次最多包含一個 <template>
  + 內容為元件的樣板

* <script>
  + 預設使用 js 而 ES2015 透過 Babel 支援
  + 每一個 vue 檔案最多包含一個 <script>
  + JS 預設被執行在一個 CommonJS 環境下，因此可以使用 require 匯入，如果要使用 import 則需支援 ES2015
  + script 必須要匯出Vue擴展建構子 `Vue.extend()`

* <style>
  + 預設 css
  + 可多個 <style> 標籤
  + 預設內容會使用 `sytle-loader` 置入 <head> <style>
  + 也可以使用 webpack 輸出 css 檔案

### 透過 src 匯入

```
<template src="./template.html"></template>
<style src="./style.css"></style>
<script src="./script.js"></script>
```

要注意的是 src 遵循跟 CommonJS require 一樣的規則

`./` 必須要相對路徑
`module/dist/all.css` 會從 npm 模組讀取

# 繼續實作

```
# Add a src/components folder
# Add a src/components/App.vue

$ npm install sass-loader node-sass -D

# Modify main.js
# index.html using App component
# 加入 webpack.config.js
```

```
{
  test: /\.vue$/,
  loader: 'vue'
}
```

```
$ npm run dev
```

### 使用 postcss 和 cssnext

我們注意到即使使用 sass 也可以使用 scoped 那是因為任何在 vue 檔案(經過 vue-loader 處理)的 css 都會經過 PostCSS 因為預設要處理 autoprefixer

vue: {
  postcss: [require('postcss-cssnext')()],
  autoprefixer: false
}

### HRM

注意版本

--hot do the following stuff:

    adds the HotModuleReplacementPlugin
    with --inline it adds 'webpack/hot/dev-server' to every entry
    switches the webpack-dev-server into hot mode, which post messages instead of reloading the page. devServer: { hot: true }

vue-router need --history-api-fallback

注意事項：

當元件進入熱替換狀態時當前的狀態 (data) 會被保留

### template

上面提到在 js 和 css 若有需要 vue-loader 會將其交給對應的 loader 處理，但是大部分 webpack 樣板的 loader e.g. jade-loader 通常都是回傳 function 而不是 HTML String 所以我們會使用 jade 而不是 jade-loader

### 關於 URL 的處理

預設 vue-loader 會自動使用 css-loader, vue-html-loader(從 html-loader fork 下來的) 處理樣式和樣板，這意味著所有的資源連結像是 <img src> background:url()，@import 都會依照載入模組的機制處理。簡單的說就是 url(image.png) => require('./image.png')
但因為圖片不是 js 所以我們需要使用 file-loader 或 url-loader

* file-loader 讓我們可以複製檔案到指定路徑，命名，另外起始的目錄路徑也可以設定
* url-loader 讓我們將檔案轉成 base64，減少 HTTP request

通常 file-loader 和 url-loader 會一起使用，小檔案就轉成 base64 大一點的則讓 file-loader 處理

```
{
  test: /\.(png|jpg|gif)$/,
  loader: 'url',
  query: {
    // limit for base64 inlining in bytes
    limit: 10000,
    // custom naming format if file is larger than
    // the threshold
    name: '[name].[ext]?[hash]'
  }
}
```

### 進階 loader 設定

一般來說我們只需要安裝對應的 loader 或 package 在 component 中使用 `lang` 設定，剩餘的 vue-loader 會幫我們交給對應的 loader
有時候您想要套用替訂的 loader 而不是讓 vue-loader 自動推斷，您可以覆寫內建的設定

extract-text-webpack-plugin 可以將 css 匯出獨立檔案

### production

```
plugins: [
    // short-circuits all Vue.js warning code
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    }),
    // minify with dead-code elimination
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    }),
    // optimize module ids by occurence count
    new webpack.optimize.OccurenceOrderPlugin()
  ]
```

# vue-cli

官方文件已強烈建議我們先讀完 vue-loader

撰寫設定耗費時間
新手在入門 vue 前需花時間熟悉建置工具

```
$ npm install -g vue-cli
$ vue init <template-name> <project-name>
$ vue init webpack-simple my-vue-webpack-simple-project
$ vue init webpack my-vue-webpack-project
```

# Understand vue-cli template

#

# 工商時間

# 參考


* [webpack 與其他工具比較](http://survivejs.com/webpack/webpack-compared/)
* [webpack  的工作基本流程](https://medium.com/html-test/webpack-%E7%9A%84%E5%9F%BA%E6%9C%AC%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B-585f2bc952b9#.dour2dmsm)
* [webpack 基礎](http://andyyou.github.io/javascript/2015/07/23/webpack.html)
* [vue + webpack 起手式](http://andyyou.github.io/js,/webpack/2016/05/25/webpack-notes.html)
* [深入 webpack](http://andyyou.github.io/webpack,/js/2016/05/30/webpack-dev-middleware-in-express.html)
* [vue-cli 的基礎](http://www.jianshu.com/p/f8e21d87a572)
* [使用 Vue 構建 Github 項目瀏覽器](https://segmentfault.com/a/1190000005651367)
* [vue-cli 官方文件](https://github.com/vuejs/vue-cli)
＊[vux 文件](https://vuxjs.gitbooks.io/vux/content/)

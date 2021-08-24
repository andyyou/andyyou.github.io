---
title: '[譯 + 補充] Webpack 2 學習筆記'
tags:
  - javascript
  - webpack
categories: Program
date: 2017-02-17 20:32:56
---



> 在今時今日，webpack 已經成為前端開發非常重要的工具之一。本質上它是一個 Javascript 模組封裝工具，但透過 loaders 和 plugins 它也可以轉換封裝其他前端的資源檔像是 HTML，CSS，甚至是圖片等，讓我們能夠控制程式發出 HTTP 請求的數量（編譯結果的檔案數量）。我們可以使用偏好的方式去撰寫這些資源檔像是 Jade, Sass, ES6 等等。同時也讓我們能夠輕易的使用來自 npm 的套件。

這篇文章目標讀者是那些剛接觸 webpack 的新手。內容將會包含設定，模組的使用，loaders，plugins，code splitting（分拆程式碼），熱替換（Hot module replacement）。

> 每過一陣子，重新學習 webpack 就會有些新的發現 - 廢話。

<!--more-->

為了完成文章中的練習您需要些前置作業：安裝 Nodejs（作者使用 v6.9.2），如果您還麼安裝那麼這邊有篇完整[使用 nvm 安裝的教學](https://www.sitepoint.com/quick-tip-multiple-versions-node-nvm/)。

# 設定

讓我們開始來使用 npm 初始化專案與安裝 webpack。

```bash
$ mkdir webpack-demo
$ cd webpack-demo
$ npm init -y
$ npm i webpack@2 -D
$ npm view webpack version
# 2.2.1
$ mkdir src
$ touch index.html src/app.js webpack.config.js
```
上面這些檔案的概略說明：

* index.html - 首頁，載入使用編譯好的 Javascript 檔案。
* src/ 目錄 - 許多開發者會把 Source Code 的目錄命名為 src 但這並不強迫。
* webpack.config.js - 設定 webpack 行為的設定檔，這邊我們可以先概略了解一下 webpack 的行為可以靠`設定檔`或者`指令的參數`調整，而 webpack.config.js 是預設的設定檔名稱。
* src/app.js - 這是我們程式的 Entry point。

接著編輯我們剛產出的這些檔案

* index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Hello webpack 2</title>
</head>
<body>
  <div id="root"></div>
  <script src="dist/bundle.js"></script>
</body>
</html>
```

* src/app.js

```js
const root = document.querySelector('#root')
root.innerHTML = `<p>Hello webpack 2</p>`
```

* webpack.config.js

```js
const path = require('path')
const config = {
  /**
   * webpack 執行環境，即 webpack 載入檔案時相對路徑的根目錄環境
   * 預設（沒有設定時）為執行指令（webpack）所在的那個目錄
   *
   * 【其他】
   * 假設自己新增一個 build/build.js 檔案，使用載入 webpack 的作法
   *
   * 例如範例：https://github.com/andyyou/webpack-context-prove/blob/master/build/build.js#L14
   *
   * 在不同目錄執行：
   * > node build/build.js (in root/ folder)
   * > node build.js (in build/ folder)
   * 預設 context 會分別為 root/ 和 root/build/
   */
  context: path.join(__dirname, 'src'),
  /**
   * Entry point
   * 因為設定了 context 所以不需要加上 src/ 了
   */
  entry: './app.js',
  /**
   * 輸出路徑與檔名
   */
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  /**
   * loaders 對應使用規則
   */
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: [
          {
            /* webpack 2.x 移除了省略 -loader 的寫法 */
            loader: 'babel-loader',
            options: {
              presets: [
                /* Loose mode and No native modules(Tree Shaking) */
                ['es2015', { modules: false, loose: false }]
              ]
            }
          }
        ]
      }
    ]
  }
}

module.exports = config
```

上面這些是常見的基本設定，它告訴 webpack 如何編譯我們的原始碼，進入點是 `src/app.js` 輸出的檔案則放在 `dist/bundle.js`。
所有的 `.js` 檔案都會使用 Bable 從 ES2015 被編譯成 ES5。

> Babel 從 6.13.0 之後提供額外的參數 loose 和 modules
> loose: 提供 loose 編譯模式，該模式啟動下 Babel 會盡可能產生較精簡的 ES5 程式碼，預設 false 會盡可能產出接近 ES2015 規範的程式碼。
> modules: 轉換 ES2015 module 的語法（import）為其它類型，預設為 true 轉換為 commonjs。

為了要能夠執行我們需要安裝 `babel-core`，`babel-loader` 和 `babel-preset-es2015`。上面還有一個值得注意的地方就是 `{modules: false}` 關閉這個設定是為了提供 `Tree Shaking` 的特性 - 移除沒有使用到的 exports 來縮小編譯的檔案大小。

```bash
$ npm i babel-core babel-loader babel-preset-es2015 -D
```

最後我們在 `package.json` 補上 `scripts` 的部分。

```json
"scripts": {
  "start": "webpack --watch",
  "build": "webpack -p"
}
```

到這步我們就可以使用 `npm start` 來執行 webpack，加入參數 --watch 會讓 webpack 進入監視模式，當發現檔案有異動時就會立即重新編譯我們的原始碼。在 console 畫面會輸出類似下面的訊息來告知我們 bundle 已經被建立了。同時有個小小的重點那就是我們可已觀察編譯後的檔案大小。

```bash
Webpack is watching the files…

Hash: f4fadf78c49f43d8a078
Version: webpack 2.2.1
Time: 1342ms
    Asset    Size  Chunks             Chunk Names
bundle.js  2.6 kB       0  [emitted]  main
   [0] ./app.js 87 bytes {0} [built]
```

在專案目錄下執行 `open index.html` 可以觀察截至目前為止的結果。

開啟 `dist/bundle.js` 看看 webpack 編譯的結果，上半部是 webpack 處理模組載入的程式碼，最下面則是我們的模組。
到這您可能不覺得有什麼特別的，不過一旦我們開始使用 ES2015 來開發並將程式模組化，那麼 webpack 就可以替我們處理後續的工作讓我們的原始碼編譯成可以在瀏覽器執行的版本。

接著，我們可以透過 `Ctrl + C` 來停止 webpack，換成執行 `npm run build` 可以編譯產品模式的 bundle（壓縮）。
您可以注意到檔案大小從 2.6kB 變成 587 bytes，再次觀察 dist/bundle.js 會發現程式碼已經被 Uglify 了。

至此我們初始化了專案並對 webpack 設定有了基本的了解。

# 模組

webpack 本身知道該如何處理載入各種格式的 Javascript 模組，比較值得注意的有兩個：

* ES2015 `import`
* CommonJS `require()`

我們使用 `lodash` 來試驗看看

```bash
$ npm i lodash -S
```

* src/app.js

```js
import {groupBy} from 'lodash/collection'
const people = [{
  manager: 'Jen',
  name: 'Bob'
}, {
  manager: 'Jen',
  name: 'Sue'
}, {
  manager: 'Bob',
  name: 'Shirley'
}, {
  manager: 'Bob',
  name: 'Terrence'
}]
const managerGroups = groupBy(people, 'manager')

const root = document.querySelector('#root')
root.innerHTML = `<pre>${JSON.stringify(managerGroups, null, 2)}</pre>`
```

執行 `npm start` 然後在瀏覽器重新載入 index.html 應該要看到我們透過 lodash 區分群組的結果。
接著讓我們把 people 的資料搬移到 `src/people.js`

* src/people.js

```js
const people = [{
  manager: 'Jen',
  name: 'Bob'
}, {
  manager: 'Jen',
  name: 'Sue'
}, {
  manager: 'Bob',
  name: 'Shirley'
}, {
  manager: 'Bob',
  name: 'Terrence'
}]

export default people

```

在 `src/app.js` 使用 import 載入 people.js

```js
import {groupBy} from 'lodash/collection'
import people from './people'
const managerGroups = groupBy(people, 'manager')

const root = document.querySelector('#root')
root.innerHTML = `<pre>${JSON.stringify(managerGroups, null, 2)}</pre>`
```

注意：不使用相對路徑的例如 `lodash/collection` 模組將會從 `/node_modules` 載入，我們自定義的模組通常使用相對路徑。

> 匯入模組時的 path 分成 `函式庫 node_modules` `相對路徑` `絕對路徑`
>  * 函式庫：什麼都不加，單純 library name
>  * 相對路徑：`./` 開頭
>  * 絕對路徑：`/` 開頭

第二小節我們示範了模組的使用，也就是我們可以把程式模組化再使用 webpack 來處理匯入使用的部分。

# Loaders

上面我們已經使用了 `babel-loader`，它是眾多 loader 中的一個，功能就是把 ES2015 轉成 ES5。而 loaders 的功用就是告訴 webpack 該如何處理匯入的檔案，通常是 Javascript 但 webpack 不限於處理 Javascript，其他資源檔像是 Sass，圖片等也都可以處理，只要提供對應的 loader。
同時 loader 也可以串連使用，概念上類似於 Linux 中的 pipe，`A Loader` 處理完之後把結果交給 `B Loader` 繼續轉換，以此類推。
最好的示範範例就是匯入 Sass 的流程，下面就讓我們來看看。

## Sass

我們的目標是要把 Sass 編譯封裝到我們的 bundle 中（Javascript）。這個轉換過程需要一些 loaders 和函式庫：

```bash
$ npm i css-loader style-loader sass-loader node-sass -D
```

為處理 `.scss` 類型的檔案加入新的編譯規則

```js
module: {
  rules: [
    // {...},
    {
      test: /\.scss$/,
      /**
       * use 屬性是用來套用，串接多個 loaders。
       * v2 為了相容的因素保留 loaders 屬性，loaders 為 use 的別名，
       * 盡可能的使用 use 代替 loaders
       */
      use: [
        'style-loader',
        'css-loader',
        'sass-loader'
      ]
    }
    // {...}
  ]
}
```

> 每當我們修改了 webpack.config.js 我們就必須重啓。 `Ctrl + C` -> `npm start`

設定 loaders 的陣列實際上會反過來逐一執行：

1. sass-loader - 編譯 Sass 成為 CSS
2. css-loader - 解析 CSS 轉換成 Javascript 同時解析相依的資源
3. style-loader - 輸出 CSS 到 document 的 `<style>` 元素內

我們可以想成像下面這樣調用 function

```js
styleLoader(cssLoader(sassLoader('source')))
```

讓我們來加入 Sass

* src/style.scss

```css
$bg-color: #2B3A43;
pre {
  padding: 15px;
  background: $bg-color;
  color: #DEDEDE;
}
```

現在我們可以在 `app.js` 匯入 scss 了。

```js
import './style.scss'
```

重載 index.html 我們應該可以看到樣式已經套用了。

## CSS in JS

上一步我們完成了從 JS 中把 Sass 當作一個模組載入。

打開 `dist/bundle.js` 搜尋 `pre {`，我們看到 Sass 已經被編譯為字串並存為一個模組。當我們匯入該模組時 `style-loader` 會把該字串嵌入 `<style>` 標籤中。

為什麼我們要這麼作呢？

這邊我們不想探討太多這個議題，不過的確有些理由讓我們這麼作：

* 回想我們在使用 jQuery 套件或 Javascript 元件時，假如這些元件包含些資源檔像是 HTML，CSS，圖片，SVG 等等，通常這時候我們就需要把這檔案搬到對應的目錄下，又或者我們<del>有潔癖</del>希望圖檔等放在我們定義的目錄下，這時我們就需要去修改路徑。當全部都封裝在 Javascript 時我們在組織時就方便很多。
* 方便移除不需要的程式碼 - 當 Javascript 元件不在被使用時， CSS 等資源檔內容也會一併被移除。
* CSS Module - 隨著 CSS 樣式越來越多，命名常常容易衝突，透過 CSS Module 的方式可以對特定元件或模組套用其專用（local）的樣式，只有該模組可以套用樣式，這樣一來也相對容易維護。
* 減少 HTTP request 的數量。

## 圖片

最後我們要在介紹一個常遇到的需求 - 圖片。我們將要使用 `url-loader` 來處理圖片的載入。
在標準的 HTML 文件中圖片需要透過 `<img>` 標籤或 CSS 的 `background-image` 來載入。使用 webpack 我們可以優化圖片並根據檔案大小分別處理。像是把比較小的圖片轉成 base64 字串存在 Javascript 中。這麼作瀏覽器可以預先載入，而且不會發出額外的 HTTP 請求。

```bash
npm i file-loader url-loader -D
```

當然我們記住了，每當我們需要一個 loader 來幫我們處理某類型的檔案時，我們除了安裝該 loader 外，還需要設定 `rules`。

```js
rules: [
  // ...
  {
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    use: [
      {
        loader: 'url-loader',
        options: {
          limit: 10000 /* 小於 10kB 的圖片轉成 base64 */
        }
      }
    ]
  }
]
```

讓我們下載張圖片並將其放在 src/

```bash
$ curl http://i.imgur.com/5Hk42Ct.png --output src/code.png
```

接著在 `app.js` 中使用下面範例載入

* src/app.js

```js
import imageURL from './code.png'
const img = document.createElement('img')
img.src = imageURL
img.style = 'background: #2B3A4F; padding: 15px;'
img.width = 32
document.body.appendChild(img)
```

檢查 HTML 該圖片元素，我們看到會如下圖片的來源被轉換成 base64 了。

```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSU..." style="background: rgb(43, 58, 79) none repeat scroll 0% 0%; padding: 15px;" width="32">
```

不只是在 Javascript 載入的圖片，我們之前提到 css-loader 同時也會解析相依的資源，但是要處理圖片 `css-loader` 也是需要 `url-loader` 來幫忙。我們可以從 [css-loader 的文件 options 一節得知](https://github.com/webpack-contrib/css-loader)。

* src/style.scss

```css
pre {
  background: $bg-color url('code.png') no-repeat center center;
}
```

也會被編譯成

```css
pre {
  background: $bg-color url('data:image/png;base64,iVBORw0KG...') no-repeat center center;
}
```

# 從模組到靜態資源

現在您應該明白 loaders 是如何協助我們編譯各式各樣的檔案，這也是 [webpack 2 官方文件首頁](https://webpack.js.org/) 出現圖片所要表達的。

![](https://webpack.js.org/15998c6f72bff817a85028b881f75163.svg)

透過 Javascript 的 Entry point 進入點，webpack 就會明白我們需要那些模組以及各種類型的檔案該如何處理。

> 心法：組織好專案架構後，安裝 webpack，設定的基礎是 entry，output 以及想編譯哪些類型的檔案，對應安裝相關 loaders 接著設定 rules。

# Plugins

開始講 plugins 之前，其實我們已經看過一個 webpack 內建的 plugin。它就是當我們執行 `webpack -p` 時使用的 UglifyJsPlugin。

簡單說，loaders 的任務是針對單一檔案協助轉換，而 plugins 的任務則針對轉換後的程式碼片段作處理。

# 通用程式碼

commons-chunk-plugin 是另一個內建的核心套件，可以將多個 Entry point 中共用模組的部分抽出來獨立成一個模組。
到這邊我們都只有單一個 Entry point 然後只輸出一個 bundle。在實務上您可能會需要多個進入點來切割 bundle，例如：我們自己開發的部分歸納在 `app.js`，其他外部的函式庫歸納在 `vendor.js`。

又或者假設我們的網站有兩個功能上需要切割的部分 - 一般使用者 `app.js` 和管理者 `admin.js`。這麼一來我們就可以像下面這樣處理：

```js
const path = require('path')
const webpack = require('webpack')

const extractCommons = new webpack.optimize.CommonsChunkPlugin({
  name: 'commons',
  filename: 'commons.js'
})

const config = {
  context: path.join(__dirname, 'src'),
  entry: {
    app: './app.js',
    admin: './admin.js'
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].bundle.js'
  },
  // ...
  plugins: [
    extractCommons
  ]
}

module.exports = config
```

注意到 `output.filename` 部分我們在前面加上了 `[name]`，同時 `entry` 也從字串換成物件格式，物件 key 的名稱會對應到 `[name]`，也就是照上面的設定會產生 `app.bundle.js` 和 `admin.bundle.js` 兩隻檔案。

而 `commons-chunk-plugin` 則會把這兩個 entry point 中共用的模組抽出來產生第三隻檔案 `commons.js`。

我們先來看看範例：

* src/app.js

```js
import './style.scss'
import {groupBy} from 'lodash/collection'
import people from './people'

const managerGroups = groupBy(people, 'manager')

const root = document.querySelector('#root')
root.innerHTML = `<pre>${JSON.stringify(managerGroups, null, 2)}</pre>`
```

* src/admin.js

```js
import people from './people'
const root = document.querySelector('#root')
root.innerHTML = `<p>There are ${people.length} people.</p>`
```

再次執行 `npm start` 就會看到 webpack 產生了 3 隻檔案

* app.bundle.js 包含了 style 和 lodash/collection
* admin.bundle.js 沒有其他額外的模組
* commons.js 包含 people 模組

然後調整 HTML 的部分：

* index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Hello webpack 2</title>
  <style>
  html, body {
    padding: 0;
    margin: 0;
  }
  </style>
</head>
<body>
  <div id="root"></div>
  <script src="dist/commons.js"></script>
  <script src="dist/app.bundle.js"></script>
</body>
</html>
```

* admin.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Hello webpack 2 - Admin</title>
  <style>
  html, body {
    padding: 0;
    margin: 0;
  }
  </style>
</head>
<body>
  <div id="root"></div>
  <script src="dist/commons.js"></script>
  <script src="dist/admin.bundle.js"></script>
</body>
</html>
```

當我們分別造訪兩個頁面的時候，commons.js 也可以因為前一次被 cache 了而加快了網頁載入的速度。

![](http://imgur.com/DRUW32I.png)
![](http://imgur.com/50b6Up8.png)

# 輸出 CSS 檔案

前面我們把 CSS 也當作模組匯入並打包進 Javascript，但實務上我們想要讓瀏覽器非同步載入和平行處理 CSS 所以我們需要輸出獨立的 CSS 檔案。

這時我們就要另外一個非常熱門的 plugin - extract-text-webpack-plugin 它可以將模組匯出成檔案。

下面我們將改寫 `.scss` rule 的部分，將編譯好的 CSS 匯出成檔案，而不是放在 Javascript。

```bash
# extract-text-webpack-plugin 2 正處於 rc 階段
# 我們可以透過下面的指令查看所有的版本
$ npm show extract-text-webpack-plugin versions

# 安裝
$ npm i extract-text-webpack-plugin@2.0.0-rc.3 -D
```

接著修改 webpack.config.js

```js
const path = require('path')
const webpack = require('webpack')

const extractCommons = new webpack.optimize.CommonsChunkPlugin({
  name: 'commons',
  filename: 'commons.js'
})
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const extractCSS = new ExtractTextPlugin('[name].bundle.css')

const config = {
  context: path.join(__dirname, 'src'),
  entry: {
    app: './app.js',
    admin: './admin.js'
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].bundle.js'
  },
  module: {
    rules: [
      // ...
      {
        test: /\.scss$/,
        loader: extractCSS.extract(['css-loader', 'sass-loader'])
        /*
        use: [
          'style-loader',
          'css-loader',
          'sass-loader'
        ]
        */
      }
    ]
  },
  plugins: [
    extractCommons,
    extractCSS
  ]
}

module.exports = config
```

重啓 webpack 您應該可以看到 webpack 匯出了 `app.bundle.css`，於是我們就可以在 HTML 中補上連結。每一個 entry 內匯入的 Sass 都會被抽出來成一隻獨立的檔案。由於 admin 沒有匯入 Sass 所以沒有輸出。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Hello webpack 2</title>
  <style>
  html, body {
    padding: 0;
    margin: 0;
  }
  </style>
  <link rel="stylesheet" href="dist/app.bundle.css">
</head>
<body>
  <div id="root"></div>
  <script src="dist/commons.js"></script>
  <script src="dist/app.bundle.js"></script>
</body>
</html>
```

# 分拆原始碼

我們已經看了一些分拆程式碼的方式：

* 手動建立多個 Entry point
* 使用 commons-chunk-plugin 分拆出共用模組的部分
* 使用 extract-text-webpack-plugin

另外一個分拆的方式是使用 `System.import` 或 `require.ensure`。透過調用這些方法設定，我們可以劃分程式碼讓它們在需要的時候才在執行時期（runtime）載入，而不是一口氣全部載入。這樣能夠有效的改善載入所造成的效能問題。`System.import` 使用 module 名稱作為參數，接著回傳一個 Promise。`require.ensure` 則是傳入相依套件的列表， callback，和一個可選的參數來設定該程式片段的名稱。

> System.import 是 ES2015 模組載入的 API

如果您的程式中某部分具有大量相依函式庫，且程式的其他地方不需要。這種情況正好就適合將其拆分出來。看看下面的範例，我們的 dashboard.js 需要 d3，但可以見得的其它地方不需要 d3。

```bash
$ npm i d3 -S
```

* src/dashboard.js

```js
import * as d3 from 'd3'

console.log('Loaded', d3)

export const draw = () => {
  console.log('Draw!')
}
```

接著在 app.js 的最下方我們模擬晚一點載入（需要的時候才載入）

```js
function gotoDashboard () {
  System.import('./dashboard')
    .then(function (dashboard) {
      dashboard.draw()
    })
    .catch(function (err) {
      console.log('Chunk loading failed')
    })
}

setTimeout(gotoDashboard, 5000)
```

因為我們使用的是 `System.import('./dashboard')` 這樣的相對路徑，在線上的情況下會變成 `http://example.com/dashboard.js` 這樣的路徑是錯誤的。所以需要加上 `output.publicPath` 的設定，這樣才會是 `http://example.com/dist/dashboard.js`。

```js
output: {
  path: path.join(__dirname, 'dist'),
  publicPath: '/dist/',
  filename: '[name].bundle.js'
}
```

重啓 webpack 會發現 console 有一個奇怪的 `0.bundle.js`

```bash
Hash: e96d4e3ab03b79a320aa
Version: webpack 2.2.1
Time: 4030ms
          Asset       Size  Chunks                    Chunk Names
    0.bundle.js     452 kB       0  [emitted]  [big]
  app.bundle.js     184 kB       1  [emitted]         app
admin.bundle.js  461 bytes       2  [emitted]         admin
     commons.js    5.91 kB       3  [emitted]         commons
 app.bundle.css    1.31 kB       1  [emitted]         app
```

webpack 用了較為突出的顏色顯示 `[big]` 讓我們去注意它。

這個 `0.bundle.js` 將會在需要的時候發出 JSONP 的請求，這個時候如果我們繼續直接讀取檔案的方式是拿不到資料的。
暫時我們可以在專案目錄下使用 python 提供的簡易伺服器（因為 Linux，OSX 內建都有 python 我們不需要在作其他安裝）。

```bash
$ python -m SimpleHTTPServer
```

瀏覽 `http://localhost:8000`，5 秒後我們應該可以看到一個 GET 請求 `/dist/0.bundle.js` 同時 console 顯示 `Loaded` 載入完成。

# Webpack Dev Server

Live reload 的出現，大大的改善了我們的開發體驗也替開發者們節省了許多的時間。簡單的說就是當檔案發生變動時瀏覽器會自動重新載入頁面。
只需要安裝並使用 webpack-dev-server 這個開發伺服器我們就可以輕鬆取得這個功能。

```bash
$ npm i webpack-dev-server@2 -D
```

接著我們需要修改 `package.json` scripts start 的部分：

```js
"scripts": {
  "start": "webpack-dev-server --inline",
  "build": "webpack -p"
},
```

執行 `npm start` 就可以透過 `http://localhost:8080` 來瀏覽網頁。

現在，只要我們修改 `src` 目錄下的檔案，例如：`people.js` 或 `style.scss` 就可以看到瀏覽器馬上更新結果。

# 熱替換（Hot Module Replacement）

如果您對於 Live reload 印象深刻的話，那麼 HMR 可能將令你感到驚訝。

假如您已經在開發 SPA （Signle Page Application），您應該已經遭遇過了這種惱人的情況 - 在你的開發過程常常因為要測試某元件而反覆操作一些流程。什麼意思？假如我們正在開發付款流程的頁面，分別有 4 個步驟，我們的元件在第 3 步，於是每當我們一修改，所有狀態因為重載的關係回到預設，然後我們就只好反覆執行步驟 1， 2。

Hot Module Replacement 就是為了拯救我們脫離這個迴圈而出現了。

我們理想的開發流程應該是：每當我們修改我們的模組，然後應該只要編譯該模組，在不刷新瀏覽器，不影響其他模組的情況下把新的程式碼換上去，當我們需要 reset 狀態時在重載頁面。大致上這就是 HMR 的功能。

我們只需要加入一個參數 `--hot` 就可以啟用這個功能：

```json
"scripts": {
  "start": "webpack-dev-server --inline --hot",
  "build": "webpack -p"
}
```

為了讓我們的模組也支援 HMR 我們需要在 app.js 的最上面補上下面這段程式碼，好讓 webpack 知道我們模組的邊界以及更新底下相依的元件。

```js
if (module.hot) {
  module.hot.accept()
}
```

注意：`webpack-dev-server --hot` 會把 `module.hot` 設為 `true` 而且只有在開發模式才支援。在 production 模式下 `module.hot` 會是 `false`，相關的程式碼不會出現在 bundle 中。

加入 `NamedModulesPlugin` 到 webpack.config.js 的 plugins 陣列，如此一來我們在瀏覽器的 console 就可以看出是哪個檔案更新。

```js
plugins: [
  // ...
  new webpack.NamedModulesPlugin()
]
```

最後，我們加入 `<input>` 到 HTML ，重新執行 `npm start` 並在頁面的輸入框輸入一些字，編輯 people.js 中的人名，存檔。我們可以觀察到 HMR 的行為，的確不是整個頁面刷新。

# Hot Reloading CSS

修改 `style.scss` 中 `<pre>` 的背景色，我們注意到 HMR 並沒有對應更新

```css
pre {
  background: red;
}
```

當我們使用 `style-loader` 時 HMR 是會更新的，我們不需要作其他設定。不過因為我們使用了 extract-text-webpack-plugin 把 CSS 獨立出去成為檔案，所以也就不支援 HMR。

# HTTP/2

使用 webpack 這類的封裝工具一個主要的好處就是可以控制最終資源檔被請求的數量。在過去幾年這是最佳的實作方式，不過 HTTP/2 的出現，整合成單檔的方式不再是唯一，分散成許多小檔案在 HTTP/2 中是相對好的作法。

不過 webpack 的作者 Tobias Koppers 寫了篇文章闡述即使在 HTTP/2 的情況下，封裝工具仍有其重要性。[連結](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6)


# 資源

* [A Beginner’s Guide to Webpack 2 and Module Bundling](https://www.sitepoint.com/beginners-guide-to-webpack-2-and-module-bundling/)
* [Migrating from v1 to v2](https://webpack.js.org/guides/migrating/)

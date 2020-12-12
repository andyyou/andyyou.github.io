---
title: '[譯] 解析 Javascript 模組機制與建置函式庫觀念'
tags:
  - javascript
categories: Program
date: 2017-12-13 17:13:49
---


身為前端或 Javascript 的開發者我們或多或少都曾經受到 `JavaScript Modules` 這個巨大的概念所困擾了。
我們都想清楚的理解到底該怎麼使用這些工具以及它們到底在我們的日常開發中扮演什麼角色。

<!--more-->

## JavaScript 模組是什麼？

隨著 JavaScript 的開發越來越複雜，命名空間、變數命名、相依的函式庫等變的越難處理。環繞著`模組化`的概念不同的解決方案被提出來用來解決這個問題。

比較有無模組的程式碼：

![](https://cdn-images-1.medium.com/max/800/1*xM9pXLcsQa641gzCfTz0iw.jpeg)

## 為何我們需要理解 JavaScript 模組？

當我們說一個應用程式具備模組化的特性，我們一般是說這個程式是由一系列高度解耦，各自負責不同功能的模組組合而成的。
您可能也知道關於低耦合與降低程式之間的相依性可提高一個程式的維護性。如果我們確實的遵守這個原則，很容易可以看出每個部分對整個程式的影響和作用。

從一個具體的例子說起：
我平常的工作是設計整個專案的架構，很快的我發現有很多常見的功能幾乎每個專案都會用到，然後通常我就是複製貼上那些程式碼到新專案上。
您也發現問題了。每當這些常用功能的程式碼有修改的時候，我就需要手動的去每個專案更新這些變動。因此為了避免這種無聊的任務我決定將這些功能抽成一個個 npm package。
這樣一來不只是我，其他團隊也可以重複使用這些功能，並且當有新版本時可以很容易的更新。

這個方式有下面幾個優點：
1. 如果核心程式碼有什麼問題需要修改，我們只需要修改一個地方。
2. 所有專案可以保存同步，只需要使用 `npm update` 指令。


所以下一步我們就要來試著開發一個函式庫

這也是最困難的地方，當我們決定要建置一個函式庫的時候剎那間好多的問題與新的術語浮現在腦中。像是：

1. 如何讓這個函式庫支援 `tree shaking`？
2. 我應該使用哪一種 JavaScript 模組標準（CommonJS, AMD, harmony）?
3. 我該如何處理原始碼的部分？
4. 我應該要封裝打包原始碼嗎？
5. 該發佈哪些檔案呢？

當要建置一個函式庫的時候，這些問題應該也浮現在您的腦中，對嗎？

這篇文章會試著解釋這些問題。

為了解釋這些問題，我們需要先理解一下主流的模組標準。

## JavaScript 模組標準與特性

### 1. CommonJS

* 使用 node 實作
* 但模組使用這種標準時，通常使用在 server 端
* 不支援執行環境／非同步模組載入
* 使用 `require` 來載入模組
* 使用 `module.exports` 來匯出模組
* 當我們匯入模組時，實際上是取得一個物件
* 不支援 `tree shaking`
* 不支援靜態解析，所以我們在執行時期取得物件時查看物件屬性
* 取得的函式庫永遠是物件的副本，所以不支援模組的即時更新
* 循環管理功能不友善
* 語法簡單

```js
// 檔案：log.js
function log () {
  console.log('Example of CJS module system')
}

module.exports = { log }

// 檔案：index.js
var logger = require('./log')
logger.log()
```

### 2. AMD（Async Module Definition）

* 使用 `RequireJS` 實作
* 常用於瀏覽器（客戶端），當我們需要動態載入模組的時候適合使用
* 使用 `require` 來匯入
* 語法相較之下複雜

```js
// 檔案：log.js
// define(id?, dependencies?, factory);
// id 格式為字串，代表模組的名稱，可省略。如果要寫的話，就必須是相對於 data-main 的檔案路徑，但不用加上 js 副檔名。
// 陣列為 dependencies
define(['log'], function () {
  return {
    log: function () {
      console.log('Example of AMD module system')
    }
  }
})

// 檔案：index.js
// 注意：dependencies 表載入的 js ，而其路徑則是相對於 `data-main` ，且不需要寫副檔名。
require(['log'], function (logger) {
  logger.log()
})
```

### 3. UMD (Universal Module Definition) 通用型標準

* 結合 `CommonJS` + `AMD` 即支援 CommonJS 的語法加上 AMD 的非同步載入
* 可以支援 AMD/CommonJS 的環境下使用
* UMD 本質上就是建立一個方式讓兩種最廣泛使用的標準都支援，同時也支援全域變數的方式。結論就是 UMD 標準的模組同時相容客戶端與伺服器端。

```js
(function (global, factory) {
  if (typeof define === 'function' && define.amd) {
    define(['exports'], factory)
  } else if (typeof exports !== 'undefined') {
    factory(exports)
  } else {
    var mod = {
      exports: {}
    }

    factory(mod.exports)
    global.log = mod.exports
  }
})(this, function (exports) {
  "use strict";

  function log () {
    console.log('Example of UMD module system')
  }

  exports.log = log
})
```

### 4. ECMAScript Harmony（ES6）

* 支援客戶端與伺服器端
* 支援執行時期/靜態載入
* 匯入時可以繫結原值（非複製）
* 使用 `import` 匯入，`export` 匯出
* 靜態解析 - 可以在編譯時匯入／匯出時確定載入的程式碼
* Tree shaking
* 模組即時更新
* 較佳的循環載入管理

```js
// 檔案：log.js
const log = () => {
  console.log('Example of ES module system')
}
export default log

// 檔案：index.js
import log from './log'
log()
```

上面我們大概的介紹了關於不同類型的 JavaScript 模組標準。由於 `ES Harmony` 模組標準還沒被所有工具和瀏覽器支援，並且我們也無法確定
當函式庫發佈出去之後，使用者會怎麼使用它們。因此我們必須確保我們的函式庫可以在所有環境下正常運作。

現在讓我們透過設計一個簡單的函式庫來更進一步了解上面提到的問題。

現在，我們要建置一個小型的介面函式庫，所有範例的原始碼發佈在 [Github](https://github.com/kamleshchandnani/js-module-system)。
下面我將分享自身關於如何編譯，封裝，發佈一個函式庫的經驗。

![](https://cdn-images-1.medium.com/max/800/1*u1HxrxTNgFJmIVMd5I-ucw.png)

如上圖，我們的這個介面函式庫包含 3 個元件：Button、Card 和 NavBar

## 最佳實踐

### 1. Tree Shaking

* 算是一個術語，通常在 JavaScript 中指的是移除那些沒被執行到的原始碼。它必須搭配 ES2015 的 `import` 和 `export`。建置工具 `rollup` 一直致力推廣這個概念。
* webpack 和 rollup 都支援 `Tree shaking` 不過我們在這邊只需要記住我們的程式碼是支援 `Tree shaking` 的。

```js
// 檔案：shakebake.js
const shake = () => console.log('shake')
const bake = () => console.log('bake')

// 當我們匯出該模組時支援 Tree shaking
export { shake, bake }

// 檔案：index.js
import { shake } from './shakebake.js'
// 只有 shake 函式會被包含在輸出的程式碼中


// 檔案：shakebake.js
const shake = () => console.log('shake')
const bake = () => console.log('bake')
// 這樣匯出物件的話沒有支援 Tree shaking
export default { shake, bake }

// 檔案：index.js
import { shake } from './shakebake.js'
// shake 和 bake 都會被包含在輸出的程式碼中
```

### 2. 發佈所有的模組標準類型

* 我們應該要支援所有主流的模組標準，即 `UMD` 和 `ES`，因為我們不知道使用者是在瀏覽器環境還是使用 webpack 的情況下使用我們的函式庫。
* 雖然大部分的 bundler （模組工具）例如 webpack、rollup 可以支援 ES 標準，但假如開發者使用 webpack 1.x 那麼就不支援 ES 標準了。

```package.json
{
  "name": "js-module-system",
  "version": "0.0.1",
  ...
  "main": "dist/index.js",
  "module": "dist/index.es.js"
  ...
}
```

* 在 `package.json` 中有幾點需要特別注意，`main` 屬性通常我們設定為編譯後的 `UMD` 版本
* 而 `module` 欄位則指向 `ES` 的版本。注意到在 `module` 這個欄位被確定為標準之前可能使用 `js:next` 或 `js:main`。

提醒：webpack 使用 `resolve.mainfields` 來決定該採用 `package.json` 中的哪個欄位。
當我們匯入 npm 套件的使用例如：`import * as D3 from 'd3'`，webpack 會根據設定決定該使用套件中 `package.json` 的哪些欄位。
當 webpack 的 `target` 設成 `webworker`，`web` 或未指定的時候

```
mainFields: ['browser', 'module', 'main']
```

如果 `target` 為 `node`

```
mainFields: ['module', 'main']
```

依據我們上面舉的 D3 的例子：

```
{
  main: 'build/d3.Node.js',
  browser: 'build/d3.js',
  module: 'index',
}
```

上面的設定意味著當我們 `import * as D3 from 'd3'` 的時候會優先使用 package.json 的 `browser` 欄位。
同樣的如果一個 Nodejs 應用程式使用 webpack 的話則會優先使用 `module` 欄位。

為了優化效能方面，我們鼓勵發佈 `ES` 模組。如此一來使用者就不用載入所有的程式碼。

## webpack vs rollup vs babel?

上方的標題列出了廣泛被使用的封裝建置工具，很難想像沒有這些工具的該如何開發。許多人涉圖比較這些工具希望找出最好的工具。
這是一個錯誤的方向，每個工具有它們各自的優點以及當初是為了解決什麼問題而被開發出來，因此我們不應該直接將它們拿來比較。

webpack 是一個很棒的模組封裝工具，廣泛的被用來建置 SPA 專案，內建 code splitting，async loading 等功能。
rollup.js 很類似 webpack 不過它不支援非同步載入，主要強調支援 ES6 等新的標準，通常我們會用在封裝函式庫。
babel 則是 JavaScript 編譯工具，使得我們可以提前使用 ES6 等新版的 JavaScript 標準語法，不同於上面兩種工具，它只是編譯工具。

總結來說：用 rollup 來打包函式庫，用 webpack 組織應用程式專案。

## 函式庫該提供哪些？

當開始思考該如何建置一個函式庫的時，接著我便開始觀察那些知名的專案看看人家是怎麼組織一個函式庫的專案。

![](https://cdn-images-1.medium.com/max/1000/1*JJ0hPQW4V7rnaCVHO0nV4w.jpeg)

在觀察不同的函式庫和套件之後，我們可以知道不同的目標會使用不同工具與架構，下面就是我的觀察

在上圖您可以清楚的看到我把這些函式庫區分成兩類

1. 介面函式庫（`styled-components`，`material-ui`）
2. 核心套件（`react`，`react-dom`）

### 介面函式庫

* 通常有一個 `dist` 目錄包含最終打包好以及壓縮過的檔案並支援不同模組標準 `ES`、`UMD`、`CJS`
* `lib` 目錄包含編譯過的函式庫

### 核心套件

* 通常只有一個目錄包含打包及壓縮過的檔案，支援 `CJS`、`UMD` 版本

## 為什麼介面的函式庫和核心套件需要輸出不同的結果

### 介面函式庫

* 想像一下如果我們只發佈打包的版本且放在 CDN 上，那麼我們的使用者只能用 `<script>` 來載入。假如使用者只想使用 `<Button />` 元件，卻一定要下載整包程式碼。瀏覽器不像封裝工具會處理 Tree shaking 最後使用者還是得下載完整的檔案。
* 現在，如果我們將 `src` 編譯到 `lib` 然後一樣發佈到 CDN，我們的用戶就可以只下載他要用的部分。

### 核心套件

* 核心套件不會透過 `<script>` 來載入，它們通常是應用程式的一部分。因此我們只需要釋出打包好的版本（UMD，ES）就好，剩下的交給使用者去處理。
他們可以使用 `UMD` 版本但是沒有支援 Tree shaking 又或者使用 `ES` 版本如果他們的封裝建置工具有支援的話。

## 編譯與封裝的作法

### 介面函式庫

1. 使用 babel 編譯原始碼支援 ES 模組標準，將結果放到 `lib` 目錄
2. 使用 rollup 打包與壓縮，支援 `CJS`、`UMD`、`ES` 標準。並透過 `package.json` 來設定不同標準該使用的檔案

### 核心套件

1. 不須產生 `lib`
2. 使用 rollup 打包與壓縮，支援 `CJS`、`UMD`、`ES`，並透過 `package.json` 來設定不同標準該使用的檔案


通常我們也會將 `dist` 目錄下的結果發佈到 CDN。

# 建置慣例

通常我們會將各別的建置指令設定在 `package.json`，您可以參考 [rollup config](https://github.com/kamleshchandnani/js-module-system/blob/master/rollup.config.js)。

# 其他發佈資訊

* License
* README
* Changelog
* Metadata - package.json

在 `package.json` 中 `files` 欄位是一個陣列，用來描述套件必須被包含的檔案。舉例來說：我們要包含 `lib` 和 `dist` 目錄則設定會是

```js
{
  "files": ["dist", "lib"]
}
```

最後將所有的檔案都建置編譯好了之後

```bash
# 註冊 npm 帳號

$ npm whoami # 查看登入的帳號
$ npm publish
```

# 參考

* [Anatomy of JS module systems and building libraries](https://medium.com/@_kamlesh_/anatomy-of-js-module-systems-and-building-libraries-fadcd8dbd0e)

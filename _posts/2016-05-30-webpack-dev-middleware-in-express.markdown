---
layout: post
title: '手把手深入理解 webpack dev middleware 原理與相關 plugins'
date: 2016-05-30 12:00:00
categories: webpack, js
---

本文將對 webpack 周邊的 middleware 與 plugin 套件等作些介紹，若您對於 webpack 還不了解可以參考這篇[彙整的翻譯](http://andyyou.github.io/javascript/2015/07/23/webpack.html)

### webpack dev server 是什麼?

`webpack dev server` 是一個開發伺服器，內建 webpack 使用的 live reloading 功能。

### 那 webpack dev middleware 是啥?

它就是一個用來組織包裝 webpack 使其可以變成中介軟體，或稱中間件的容器。回想一下 express 你大概可以明白關於 middleware 的用途，就是在輸入到輸出的過程中 `加工` 的一種手段。單純說 middleware 的話我們可以想成一系列`任務, 動作`(actions stack)，不只 express 有，在 Ruby 中的 rake 也具備這種機制。

先看看[web dev server](https://webpack.github.io/docs/webpack-dev-server.html)的說明

> The webpack-dev-server is a little node.js Express server, which uses the webpack-dev-middleware to serve a webpack bundle.

從頭說起的話就是 `webpack` 本身只負責打包編譯的功能 `bundle`, `webpack-dev-server` 當然就是協助我們開發的伺服器，這個伺服器底層是靠 express 來實作的，接著思考一下我們要如何更新(live reload)呢? 當然是需要取得 webpack 編好的資料啊，於是就需要在從 `request` 到 `response` 的過程中透過 express 的 middleware 取得資料，而方法就是透過 webpack-dev-middleware 。

比起直接編譯成檔案，webpack-dev-middleware 這個套件還多了一些好處:

* 不需要一直寫入磁碟，所有產生的結果會直接存在記憶體
* 在監視模式(watch mode)下如果檔案發生異動，middleware 會馬上停止提供舊版的 bundle 並且會延遲請求的回應直到編譯完成，如此一來我們就不需要去`觀察編譯是否結束了`

### webpack hot middleware 是什麼?

我們都知道 webpack dev server 有提供一種`Hot Module Replacement/Hot Reloading` 熱替換的功能。在一般 `webpack-dev-server` 的時候我們會在 `webpack.config.js` 加入 `new webpack.HotModuleReplacementPlugin()` 或設定來啟用。
而 **webpack hot middleware** 是給 `webpack-dev-middleware` 用的。就是讓我們在一般的 server 上加上熱替換的功能，總結來說就是 `webpack-dev-middleware` + `webpack-hot-middleware` 即可讓我們用 express 客製一個有熱替換功能的 webpack 開發伺服器。

### 使用 webpack-dev-server 當中介軟體

webpack 提供了 express 的 middleware 讓我們可以處理一些靜態資源檔而不是使用 `express.static`。要達成這項功能，我們需要安裝 `webpack-dev-middleware` 和 `webpack-hot-middleware`

```zsh
$ npm i webpack express webpack-dev-middleware webpack-hot-middleware -D
```

安裝完成套件之後，首先我們需要設定一個 `webpack.dev.config.js` 檔案，並且在 `entry` 中加上 `webpack/hot/dev-server` 和 `webpack-hot-middleware/client`

```
entry: [
  'webpack/hot/dev-server',
  'webpack-hot-middleware/client',
  'client/index.js'
]
```

這個 `webpack.config` 主要是給開發伺服器用的，由於這時的匯出都會存在記憶體中，因此 `path` 可以直接設為根

```
output: {
  path: '/',
  publicPath: 'http://localhost:8080/scripts/',
  filename: 'bundle.js'
}
```

最後補上任何您所需要的 loaders，最重要的是記得。

```js
plugins: [
  new webpack.HotModuleReplacementPlugin()
]
```

接著下來我們開始來撰寫這個開發環境的設定檔和 express 程式。
我們會匯入 webpack，webpack-dev-middleware， webpack-hot-middleware 和 express。

> 若需要搭配樣板引擎請自行安裝 ejs 或 jade

```js
var express = require('express')
var webpack = require('webpack')
var WebpackDevMiddleware = require('webpack-dev-middleware')
var WebpackHotMiddleware = require('webpack-hot-middleware')
```

載入套件之後，使用 express 建立一個 http 應用程式與路由

```js
app = express()
router = express.Router()

router.get('/', MainController)
app.use(router)
```

上面只是一個一般的 Server 應用，為了達成 webpack 的神奇黑魔法我們需要匯入 webpack 的設定

```
var config = require('./webpack.dev.config')
```

webpack 的角色就是我們的編譯器，透過下面的程式碼建立編譯器的 instance

```
var compiler = webpack(config)
```

重點來了，我們有了伺服器 express，有了編譯核心 webpack，接著我們需要 wrapper 來打包 webpack 將其合進 express 的 middleware stack 中。

```js
app.use(WebpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath,
  stats: { colors: true }
}))
```

`publicPath` 就是我們想要存取前端 bundle 的網址，路徑，位置。
然後我們要再加上 webpack-hot-middleware 使其具備熱替換的功能。

```js
app.use(WebpackHotMiddleware(compiler, {
  log: console.log
}))
```

最後則是 express 的監聽事件

```js
app.listen(8080, function () {
  console.log('Listening on 8080')
})
```

完整的 server 程式碼如下

```js
var express = require('express')
var webpack = require('webpack')
var WebpackDevMiddleware = require('webpack-dev-middleware')
var WebpackHotMiddleware = require('webpack-hot-middleware')
var config = require('./config/webpack.dev.config')
var compiler = webpack(config)

app = express()
app.set('views', './views')
app.set('view engine', 'ejs')
app.use(express.static('public'));

app.use(WebpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath,
  stats: { colors: true }
}))
app.use(WebpackHotMiddleware(compiler))

var router = express.Router()
router.get('/', function (req, res, next) {
  res.render('index', { message: 'Hey there!'});
})
app.use(router)

app.listen(8080, function () {
  console.log('Listening on 8080')
})
```

### 換個思路

假設我們並不是要實作一個全站 SPA 的站，實務上我們的確會遇到需要拆分為許多 view `.html` 的狀況，這種情況下我們會希望自己客製的這個 server 就像 `webpack-dev-server` 一樣，當然，這邊只是要指出做法，如果一樣您當然就直接用 webpack-dev-server 就好了。

根據上面這個需求最簡單的方式就是透過 `express.static(__dirname)` 讓 express 直接 return raw 檔案。

### html-webpack-plugin

小弟認為在學習的過程中，最重要的就是搞懂動機，而這個 `html-webpack-plugin` 插件，其用途就是簡化建立 html 的過程。
先回頭看看上一小節，很直覺的，我們會依據需求建立`不同的頁面(.html)`，因為在開發過程中很多時候前端只需要注重那些`互動介面的邏輯`，`樣式`，`樣板`，`標籤結構` ，那我們的重點只有 client 端的 html, js, css 就不在話下了吧！再如果我們又以`元件`為思路中心來設計實踐的話，那麼 html 裡面大部分的東西都會往元件的 template 搬。依據 SPA 的思路，html 的責任就只是把我們的 bundle 載入並掛載 `root component`。

如果照著這樣的想法，不斷的新增 html 結果大部分的內容都是重複的那就不太靠譜啦。我們就需要一種簡化工作的方式。

這個套件如上面所說就是簡化`建立載入 bundle 的 html`的步驟，用在 webpack 打包的檔案包含每次編譯都會更新的 hash 時特方便。
我們可以讓套件幫我們產生 html 或者搭配 loaders 與其他樣版引擎。


### 基本的用法

第一種最簡單的用途就是為我們的 bundle 包上一層 html

```
plugins: [
  new HtmlWebpackPlugin({
    filename: 'i_love_this_file.html'
  })
]
```

如果我們有多個 `entry` 進入點，那麼所有的 bundle 都會被加進這個自動產生的 HTML 中。
如果我們透過 webpack 匯出了 css 資源檔(例如 extract-text-plugin) 那麼這些檔案也會透過 `<link>` 被加入 HTML 中。

### html-webpack-plugin 的設定

當然這個套件也有一些參數，讓我們可以透過設定提供其他的功能。

* `title`: 設定該 html  的 `<title>` 標籤
* `filename`: html 檔名，也當作路徑存取。預設是 `index.html`
* `template`: 樣板的路徑，也就是說我們可以先組織 HTML 在載入讓 `html-webpack-plugin` 幫我們注入(inject) bundle。此部分要注意相對路徑是從 server 程式檔案出發。
* `inject`: 將所有的資源檔注入 `template` 或 `templateContent`，當值是 `true`, `'body'` 的時候所有的 js 資源檔都會被注入 `</body>` 之前，`'head'` 則是 `<head>` 之間，`false` 自然就是關閉
  - true: Boolean
  - false: Boolean
  - head: String
  - body: String
* `favicon`: 替 HTML 加上 favicon 路徑
* `minify`: 傳入 [html-minifier](https://github.com/kangax/html-minifier#options-quick-reference) 參數物件，壓縮輸出。
  - options: Object
  - false: Boolean
* `hash`: `true` 時替 webpack 編譯的檔案或結果路徑結尾補上 hash，這麼做的用意是在開發時期當檔案有異動時可以避免瀏覽器快取
  - true: Boolean
  - false: Boolean
* `cache`: 預設是 `true` 快取檔案，除非檔案有異動
  - true: Boolean
  - false: Boolean
* `showErrors`: 預設 `true` 例外或錯誤資訊會寫入 html 頁面
  - true: Boolean
  - false: Boolean
* `chunks`: 允許我們加入一些程式碼片段，例如單元測試
* `chunksSortMode`: 控制 chunks 排序
  - none: String
  - auto: String
  - dependency: String
  - {}: Function
* `excludeChunks`: 略過部分 chunk 程式碼片段
* `xhtml`: 設定為 `true` 的話 `link` 標籤會是 self-closing ，預設是 `false`
  - true: Boolean
  - false: Boolean

### 腦力激盪 - 如果要多個頁面搭配各自的 bundle?

webapck 難就難在其靈活之中伴隨著複雜，不同的思路有著不同的做法。這一小節目的是為了不讓我們對 webpack 使用上僵化而提出的一個小題目。

要達成這個需求，我們可以先使用 webpack.config 中 `[name]` 的功能拆分我們的 bundle

```
{
  entry: {
    a: './path/src/a',
    b: './path/src/b',
    c: './path/src/c'
  },
  output: {
    filename: '[name].bundle.js'
  }
}
```

接著透過 `html-webpack-plugin` 的參數，把 `inject: false` 然後 `template` 在各自的 template 中使用 bundle。

### [html-webpack-template](https://github.com/jaketrent/html-webpack-template) - 更牛的方式

照著上面的方式你可能又跟我抱怨，那不是又要產一堆 HTML 了嗎? 對啊！原本這個架構就是針對 SPA 設計的嘛。不過透過這樣來來回回的思考動機與流程我相信對於您日後使用 webpack 與閱讀設定有很大的幫助。現在的問題是 - 你覺得產一大堆 HTML 不是很靠譜，於是我們就有了 `html-webpack-template` 的產生啦

這個東西大略的用法就是

```js
plugins: [
  new HtmlWebpackPlugin({
    title: 'Sample',
    filename: 'sample.html'
  }),
  new HtmlWebpackPlugin({
    inject: false, // 必須
    template: require('html-webpack-template'), // 必須
    filename: 'sp.html', // 存取的路徑

    // 只需要特定 bundle 可以這樣設定
    chunks: ['vender'],
    title: 'OH My Gosh',
    // 可以參考 html-webpack-template 的參數設定
    // 下面為提供 GA
    googleAnalytics: {
      trackingId: 'UA-XXXX-XX',
      pageViewOnLoad: true
    }
  })
]
```

### html-webpack-plugin 事件

特地介紹此套件的事件也是因為挺有可能會需要一些時間點對 html 動些手腳，有了事件的機制我們就可以讓`其他套件`修改產生的 html

非同步事件:

* `html-webpack-plugin-before-html-generation`
* `html-webpack-plugin-before-html-processing`
* `html-webpack-plugin-after-html-processing`
* `html-webpack-plugin-after-emit`

同步事件:

* `html-webpack-plugin-alter-chunks`

大略的用法就是在透過 hook event 綁定的事件做些處理

```js
compiler.plugin('compilation', function(compilation) {
  console.log('The compiler is starting a new compilation...');

  compilation.plugin('html-webpack-plugin-before-html-processing', function(htmlPluginData, callback) {
    htmlPluginData.html += 'The magic footer';
    callback(null, htmlPluginData);
  });
});
```

### webpack-hot-middleware

`webpack-hot-middleware` 這個套件只能搭配 `webpack-dev-middleware` 使用，其實就是把熱替換的功能加到一般 server 應用。
這個模組只專注在處理 webpack 和瀏覽器溝通的機制。這個中介軟體會去訂閱監聽開發伺服器，當更新或異動發生的時候它就透過 webpack 的 HMR API 來更新。實際上讓您的程式能無縫的使用熱替換已超過本文範圍，在這部分通常會靠其他模組來處理。

安裝完套件與在伺服器 app 中套用之外，要記得 webpack.config 的 plugin 也要加上 `HotModuleReplacementPlugin`

```js
plugins: [
  // Webpack 1.0
  new webpack.optimize.OccurenceOrderPlugin(),
  // Webpack 2.0 fixed this mispelling
  // new webpack.optimize.OccurrenceOrderPlugin(),
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NoErrorsPlugin()
]
```

簡短地介紹一下 `OccurrenceOrderPlugin` 部份，您應該知道 webapck 會給編譯好的程式碼片段一個 id 以用來辨別。
透過上面的這個 plugin 可以讓 webpack 在 id 的分派上優化並保持一致性。

接著要在 entry point 加上 `webpack-hot-middleware/client` 這隻檔案會連到 server 目的是當 server 重新編譯好檔案時收到通知然後更新 client 的檔案。

### 如何撰寫 plugin

為什麼要了解怎麼寫 plugin 呢? 因為某些 plugin 可以擴展支援其他 plugin 相互傳遞資料或需要客製後續任務，所以稍微明白 plugin 的寫法可以讓我們對於 plugin 的設定更加清楚。

plugin 的架構設計促使第三方開發者讓 webpack 核心發揮出無限的潛力。在不同建置階段執行 callback ，開發者可以自訂出特有的行為。
當然建置 plugin 比起開發 loader 是較進階的議題，因為我們必須要理解 webpack 內部的一些 hook 事件

##### 編譯器與編譯結果

要開發 plugin 第一步就是先了解其中最重要的兩個角色 `compiler` 和 `compilation` 物件

* `compiler` 編譯器物件代表一個完整設定的 webpack 環境。這個物件在 webpack 發動之後就會被建置，而且只會建置一次。然後它會配置所有可以操作的設定包含 `loaders`, `plugins`。當我們套用一個 plugin 這個 plugin 會收到 `compiler` 的參考透過存取這個`參考 reference`就可以取得 webpack 環境

* `compilation` 編譯成果這個物件代表的是`某個版本的編譯後的資源檔`，在運行 webpack dev middleware 期間每當檔案發生異動就會產生一個新的 `compilation` 也就是產生新的編譯結果。這個`編譯結果`包含的訊息包含 module 模組的狀態，編譯後的資源檔，發生異動的檔案，被觀察的相依套件等。這個編譯結果物件也提供一些執行 callback 的機會讓我們可以在過程中客製一些自己想要的行為。

任何 webpack plugin 都必須依靠這兩者來完成，所以有需要對其原始碼有些大概的了解

* [Compiler Source](https://github.com/webpack/webpack/blob/master/lib/Compiler.js)
* [Compilation Source](https://github.com/webpack/webpack/blob/master/lib/Compilation.js)

### 基本 plugin 架構

本質上來說 plugin 只是一個物件實例具有 apply 方法，這個 `apply` 會在安裝時期被 `webpack compiler` 執行一次。
透過這一次的執行呢我們就可以繫結許多事件，直接來看看程式碼您就明白了。

```js
function MyPlugin(options) {
  // 設定參數
  this.options = options
}

MyPlugin.prototype.apply = function(compiler) {
  compiler.plugin('done', function() {
    // 當 plugin 安裝完成就會...
    console.log('Hello World!');
  })

  // 我們自然需要拿到編譯的結果
  compiler.plugin("compilation", function(compilation) {
    console.log(compilation.assets)

    compilation.plugin("optimize", function() {
      console.log("Assets are being optimized.");
    });
 });
}
```

OK! 我們現在並不是要開發套件所以點到這邊我想就足夠了，剩下的您可以自行參考相關文件。
 text.forEach is not a function
* [詳細 plugin API](https://github.com/webpack/docs/wiki/plugins)

### extract-text-webpack-plugin

顧名思義這個 plugin 的用途就是把 text 類型的結果匯出成一個檔案，先說這不是非常精確的描述，但概念來說 text 類型指的就是`不會`輸出成 `module.exports` 或 `json` 的資料。而像是 CSS 這類的資源檔 webpack 其實最終就是在 JS 中幫我們建個 style tag 的  dom 然後整包放進去。`file-loader`, `raw-loader` 等等這類內容大略就屬於 text 類型。[查閱各種 loaders 回傳資料類型](https://webpack.github.io/docs/loader-conventions.html)

於是乎以 entry point 為單位過程中解析的 text 內容就會被抽出來匯出成一個檔案。最常見的用法就是把 css 抽出來:

```js
var ExtractTextPlugin = require("extract-text-webpack-plugin")

module.exports = {
  module: {
    loaders: [
      { test: /\.css$/, loader: ExtractTextPlugin.extract("style", "css") }
    ]
  },
  plugins: [
    // 注意: 這邊的副檔名如果亂下是會造成瀏覽器行為不符合預期的，例如不給副檔名那瀏覽器就會當作 binary 下載
    new ExtractTextPlugin("styles.css")
  ]
}
```

如果想要拆分多個檔案，那麼就先初始化 instance

```js
let ExtractTextPlugin = require('extract-text-webpack-plugin');

// multiple extract instances
let cssExtractor = new ExtractTextPlugin('stylesheets/[name].css');
let lessExtractor = new ExtractTextPlugin('stylesheets/[name].less');

module.exports = {
  module: {
    loaders: [
      {test: /\.scss$/i, loader: cssExtractor.extract(['css','sass'])},
      {test: /\.less$/i, loader: lessExtractor.extract(['css','less'])},
      ...
    ]
  },
  plugins: [
    cssExtractor,
    lessExtractor
  ]
}
```

### HMR 熱替換

Hot Module Replacement (HRM) 又稱熱替換，功能就是在程式運行中交換，移除，增加模組且不會使頁面重新載入。這跟我們伺服器的熱插拔差不多概念。

##### 它是怎麼運作的?

webpack 在 bundle 中即我們的 js 裡加入了一個小型的 HMR 執行環境，在編譯過程中這個 runtime 會在我們的 app 中運行。
當建置完成時 webpack 也不會消失反而會持續存在，繼續監控原始碼檔案是否發生修改。一旦 webpack 發現程式有改變他就會去重新編譯那些有修改的模組，不全部重建。根據設定要嘛就是 webpack 把訊號丟給 HRM runtime 要嘛就是 HRM 自己更新異動資訊。不管哪種方式反正重點就是 HRM runtime 會取得修改的模組，接著就試著在運行的狀態下更新模組。首先會先檢查更新的模組是否能 `self-accept`。

關於 `self-accept` 先看看[範例](https://github.com/webpack/hot-node-example/blob/master/index.js)和[原始碼](https://github.com/webpack/webpack/tree/master/test/hotCases/runtime)，意思是要`支援熱替換的模組或說編譯結果`基本上是應該要實作 `module.hot.accept` 和遵循其他熱替換的規則。
如果沒有辦法自己確認自己可以直接被更新，那就往上傳，通知那些 require 匯入使用自己的模組更新，就這樣層層往上。直到有人可以 accept 或到頂，不過一旦到根就表示熱替換失敗。

讀到這邊你可能通了，為什麼當我們要讓 React 支援 Hot Mode 的時候需要一個 `react-hot-loader`。以及因為要和 HRM 執行環境溝通的關係我們需要在 bundle 的 entry point 加上 `webpack/hot/dev-server`, `webpack-hot-middleware/client` 之類的東西。

##### 從 App 的角度

![](http://i.imgur.com/HziHPQ0.png)

當 App 程式開始執行(就是載入 bundle) HMR runtime 執行環境就會啟用，接下來程式就會要求 HMR runtime 幫我們檢查是否需要更新。HMR 會幫我們下載更新然後通知 App 程式有哪些更新可用。

##### 從編譯器(webpack/compiler)的角度

除了一般的資源檔像是圖片，css，編譯器還需要觸發`更新事件`讓程式碼可以完成新舊替換。這個"更新"包含兩個部分

1. 更新的 Manifest 支援配置文件(json)
2. 一或多個更新的`chunks`程式片段(js)

支援配置文件包含更新後編譯結果的 hash 和新的 chunks 程式碼片段的列表。而新的 chunks 則包含更新後模組的程式碼或 `flag`
編譯器同時也會確保模組和片段 ID 是一致的，透過一個 `records` 的 json 檔案來儲存相關資訊。

##### 從模組角度

HMR 是選擇性的功能，所以只有在模組包含 HRM 程式碼才會被影響作用。也就是在模組中使用文件有提供的 API。一般來說模組的開發者 handler 會在模組相依的部分更新時被執行。當然也可以寫一個 handler 在這個模組更新時被呼叫。
在大部分的情況並不需要為每一個模組都撰寫`支援 HMR 的程式碼`，當一個模組沒有遵循處理規則時就會往上層傳遞事件，意味著只有上方有一個 handler 可以處理就好，但不要讓這個冒泡事件一路冒到頂喔。

##### 從 HMR runtime 角度

模組系統的執行環境其實是額外加入的程式，用來追蹤模組之間的父子關係。

```js
if(module.hot) {
	...
}
```

從管理的角度，這個執行環境 runtime 支援 `check` 和 `apply` 兩個方法。
`check` 的功能是發出 HTTP request 用來取得上面提到的 Manifest，當 request 失敗時就等於沒有任何更新。否則就會依照得到的`更新列表`去比對 chunks。
對每個已載入的 chunk 都會有對應更新的程式碼要被下載。所有模組更新會被存在 runtime 中準備拿來更新。當執行環境切換成 `ready` 狀態就表示更新的程式碼都被下載完成了隨時可以套用。

接著 `apply` 方法會將所有已更新的模組的 `flag` 標記為 `invalid` 無效，然後無效的模組需要 update 的 handler 處理函式，這個 handler 會在模組中或者父節點上。只要沒有這個 handler 就會持續往上曾傳遞並標註為 `invalid`，一旦冒泡機制冒到頂端即 `entry point` 就表示熱替換失敗。

所有被標記為無效的模組都會透過 `module.hot.dispose` 卸載，然後更新 hash，再來所有 `module.hot.accept` 的 handlers 會被調用。
執行環境切回 `idle` 狀態表示所有更新都完成了。

講這麼多其實簡單來說就是我們的模組要補一些 hot mode 的邏輯

```js
var app = require("./app");

// 模擬每 5 秒更新一次
setInterval(function() {
	console.log(app(new Date()));
}, 5000);

if(module.hot) {
	module.hot.accept("./app", function() {
		app = require("./app");
	});
}
```

###### 檔案的更新流程

左邊表示初始化時編譯器產生的結構，右邊則是當模組 4 和 9 更新時的流程。
方塊表示從 Entry 開始，webpack 幫我們編譯產生的部份從 Entry 然後轉換成 Chunk 0 - 4

![](http://webpack.github.io/assets/HMR.svg)


# 資源參考

* [html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin)
* [webpack dev middleware 說明](http://madole.github.io/blog/2015/08/26/setting-up-webpack-dev-middleware-in-your-express-application/)

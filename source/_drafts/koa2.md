---
title: Koa 2 學習筆記
categories: Program
tags: [javascript, koa]
---

# 介紹

Koa 是一個全新的網頁框架，由 Express 的開發成員所設計，目標成為更精簡，架構與程式碼更具語意詮釋性，更穩健的網頁應用程式與 API 開發框架。
透過使用 generators，Koa 讓我們可以輕易控制 callbacks 並優化錯誤處理機制。Koa 核心本身不包含任何中介軟體（Middleware），但是提供了一套優雅的機制讓我們可以輕鬆快速的撰寫 server 類型的應用。

# 安裝

Koa 需要 Node v7.6.0 以上版本或支援 ES2015 和 `async function`。透過 `nvm` 您可以快速安裝支援版本的 Node。

```bash
$ nvm install 7
$ nvm i koa
$ node my-koa-app.js
```

# 使用 Babel 支援 Async Function

欲在 Node 7.6 以下版本使用 Koa 支援的 async function，建議使用 [babel 的 require hook](http://babeljs.io/docs/usage/require/)。

```js
require('babel-core/register')
const app = require('./app')
```

並且為了解析與轉譯 async function，您最少需要 `transform-async-to-generator` 或 `transform-async-to-module-method` 套件。例如：您需要在 `.babelrc` 加上

```json
{
  "plugins": ["transform-async-to-generator"]
}
```

[使用 babel-register 範例](https://github.com/andyyou/babel-async-hook)

您也可以使用 [stage-3 preset](http://babeljs.io/docs/plugins/preset-stage-3/) 替代。

# 應用程式

一個 Koa 應用程式是一個包含 middleware function 陣列的物件，透過堆疊的方式組成。每個 request 請求將會循序經過 middleware 處理。
Koa 和於許多中介軟體風格的系統，例如 Ruby 的 Rack，Connect/Express 等並無太大的差異。不過在底層 Middleware 層部分 Koa 提供了一種關鍵的設計 - 以高度抽象的語法（語法糖）來撰寫 Middleware。這使得 Koa 改善了操作性，穩定性讓我們更輕易的撰寫 Middleware。

Middleware 包含有處理內容、內容協商（content-negotation）、處理暫存、反向代理、轉址等常見的需求。儘管支援各式各樣的 Middleware 但 Koa 預設並不將這些納入核心。

讓我們先來看看一個基本的 Koa Hello World 應用程式長怎麼樣：

```js
const Koa = require('koa')
const app = new Koa()
app.use(ctx => {
  ctx.body = 'Hello, World'
})
app.listen(3000)
```

# 串接 Middleware

Koa 的中介軟體函式庫使用一種非常傳統的方式來串連，您也許非常熟悉這種用法。但過去這種方式很容易寫出 callback hell 即 callback 內又需要另一個 callback 導致程式碼非常難維護。然而 Koa 使用 async function 可以讓我們完成真正的 Middleware 風格。和 [Connect](https://github.com/senchalabs/connect) 的實作相比，Connect 只是單純把每次的結果和控制權往下交給下一個 function 直到最後回傳結果。
Koa 的 Middleware 本質上是個 function 其會回傳一個 `MiddlewareFunction` 即 `function (ctx, next)`。`upstream` 的 Middleware(即先執行的 Middleware)靠調用 `next()` 開始執行 `downstream` Middleware(下一個)，然後回到 `upstream` 繼續執行。

官方使用 upstream 和 downstream 來詮釋，實際上就是`上一個`和`下一個` Middleware。

下面的例子一樣是回傳一個 Hello World，不過當 request 進來時會先使用 `x-response-time` 和 `logging` Middleware 來註記何時請求開始，接著直達我們撰寫回應的 Middleware。當一個 Middleware 調用了 `next()`，該 function 就會被暫停，然後將控制權交給下一個 Middleware。直到沒有下一個的時候(沒有 downstream)就會像執行堆疊一樣，每個 downstream 會把控制權交回 upstream 完成未執行的程式碼。

```js
const Koa = require('koa')
const app = new Koa()

app.use(async function (ctx, next) {
  const start = new Date()
  await next()
  const ms = new Date() - start
  ctx.set('X-Response-Time', `${ms}ms`)
})

app.use(async function (ctx, next) {
  const start = new Date()
  await next()
  const ms = new Date() - start
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`)
})

app.use(ctx => {
  ctx.body = 'Hello World'
})

app.listen(3000)
```

# 設定

應用程式的設定就是 app 物件實例的屬性，下面列出目前支援的參數

* app.env 預設為 `NODE_ENV` 或 `development`
* app.proxy 設為 true 時 HTTP 的 proxy header 欄位會被信任
* app.subdomainOffset 預設忽略 2。例如:網域為 a.b.c.com 時 `req.subdomains` 會取得 `['a', 'b']` 。設定為 3 時則得到 `['a']`

# app.listen()

一個 Koa 應用程式和 HTTP 伺服器之間`不只是`一對一的關係。多個 Koa 應用程式可以被掛載在一起組成一個大型的 HTTP Server 應用。
透過調用與傳入參數給 `Server.listen()` 可以建立並回傳一個 HTTP Server。關於參數的部分則跟 [nodejs.org](https://nodejs.org/api/http.html#http_server_listen_port_hostname_backlog_callback) 文件上記載的一樣。下面範例是一個沒有作用的 Koa 程式，HTTP 服務繫結到 port 3000 

```js
const Koa = require('koa')
const app = new Koa()
app.listen(3000)
```

`app.listen()` 其實就是下面程式簡化後的語法糖

```js
const http = require('http')
const Koa = require('koa')
const app = new Koa()
http.createServer(app.callback()).listen(3000)
```

這表示我們可以將一個程式同時掛載到 HTTP 與 HTTPS 甚至多個網址上

```js
const http = require('http')
const Koa = require('koa')
const app = new Koa()
http.createServer(app.callback()).listen(3000)
http.createServer(app.callback()).listen(3001)
```

# app.callback()

返回一個 callback 函式，這個函式傳入 `http.createServer()` 用來處理 `request`。您也可以使用這個 callback 將 Koa 掛載到 Connect/Express 上。

# app.use(function)

加入 Middleware function 到該應用程式中。更多關於用法與套件的部分請參考[Middleware](https://github.com/koajs/koa/wiki#middleware)

# app.keys=

設定一個簽署 cookie 的金鑰。

設定的值會被傳給 [KeyGrip](https://github.com/jed/keygrip)，如果您想自己建立 KeyGrip 物件實例，也可如下這般使用

```js
app.keys = ['im a newer secret', 'i like turtle']
app.keys = new KeyGrip(['im a newer secret', 'i like turtle'], 'sha256')
```

注意，簽署金鑰只會在設定 `signed` 參數時才會生效。

```js
this.cookies.set('name', 'andyyou', { signed: true })
```

# app.context

`app.context` 是 `ctx` 建立的原型，透過修改 `app.context` 我們可以加入額外的屬性到 `ctx` 中。如果我們需要在整個程式流程中存取某些功能或資料，透過將屬性加到 `ctx` 是非常簡單實用的方式，我們甚至不需要 middleware，不過這相對意味著我們對於 ctx 的依賴性更高了，也被視為有待優化的設計模式。

下面範例我們加入了資料庫的參考到 ctx

```js
app.context.db = db()

app.use(async (ctx) => {
  console.log(ctx.db)
})
```

注意：
* 大部分在 ctx 中的屬性使用了存取子 (getters, setters) 和 Object.defineProperty() 的方式來定義，簡單說就是 OOP `封裝`屬性的方式，不讓我們直接操作`值`本身。如果要修改這些屬性_(不建議)_，可以對 `app.context` 使用 `Object.defineProperty()`。相關問題請[參閱](https://github.com/koajs/koa/issues/652)。
* 掛載的 middleware 會使用該物件實例的 `ctx` 和 `settings`。

# 錯誤處理


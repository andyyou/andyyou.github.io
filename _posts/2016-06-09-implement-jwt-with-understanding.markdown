---
layout: post
title: 'Node 實作 jwt 驗證 API'
date: 2016-06-09 12:00:00
categories: js
---

# 基於 token 的驗證機制

在今天，我們應該常常能看到大量使用 token 來處理驗證的服務，由於大部分的網站服務開始大量使用 API，於是 token 就成了處理驗證使用者最好的方式。
我們的應用程式選擇使用 token 驗證機制有些非常重要的原因，最主要是因為:

* 對於伺服器來說它具備 stateless(無狀態), scalable(擴展性)
* 移動裝置也能一併使用
* 可將驗證結果導向其他應用程式
* 更安全

# 誰已經使用 token 驗證機制

大多您已經使用的 API 或網站服務已經使用 token 的方式像是 Facebook, Twitter, Google+, Github 等等

# 為什麼 token 會被大家接受

在我們開始了解 token 的運作原理和優點之前我們先來看看過去的做法 `Server based authentication` 基於伺服器的驗證機制

由於 HTTP 協定是不儲存狀態的(stateless)，這意味著當我們透過帳號密碼驗證一個使用者時，當下一個 request 時它就把剛剛的資料忘了。於是我們的程式就不知道誰是誰，就要再驗證一次。過去我們的應用程式記得使用者的方式就是把資料存放在伺服器。伺服器透過 session 紀錄而儲存有幾種不同的方式，可以被存在記憶體或者是硬碟。

大略的說從 HTTP 是無法保持狀態開始，然後人們因為動態網頁的需求，需要辨別使用者或之前的操作於是有了 cookie 讓客戶端能夠紀錄資訊，在每次請求的時候附上這些資訊以提供伺服器處理。接著基於一些安全性的需求我們透過`發放編號 session id`的方式，把那些不希望被竄改的資訊存在伺服器記錄。之後透過 session id 來辨別使用者。

本文並非要深入探討舊有的方式所以我們簡單理解 session `會話期間`(意義為進行一系列活動/溝通之期間)於網路協議時包含`連線` & `維持狀態`的行為與資訊就是一種讓我們解決`保持狀態`問題的方法。

下圖就是伺服器的工作流程。

![](https://cask.scotch.io/2014/11/tokens-traditional.png)

* 使用者對伺服器發出內容請求
* 伺服器回應內容給客戶端
* 當使用者登入時，伺服器會把登入資料儲存在 session 中，第一次進來的時候伺服器會發放一組號碼牌(session id)給使用者
* 下次同一個使用者(瀏覽器)再進來的時候，伺服器就可以靠 session id 來判斷這位使用者

這樣的方式陪伴著我們好多年，不過在 web 與 mobile 應用爆發性成長的今日，這樣的方式就顯現出一些問題，尤其是在擴展方面。

# 伺服器驗證機制的問題

其中幾個最主要的問題:

* sessions: 每一次使用者驗證，必須建立一組紀錄在伺服器上。通常會存在記憶體中，當驗證的使用者越多存放的資料就越多。
* 擴展性: 由於 session 存在記憶體這就會造成擴展時的問題，當我們使用一些雲端服務增加伺服器執行負載平衡的時候把這些重要的資訊存在記憶體就會受到一些限制。
* 跨站: 當我們想要讓程式的一些資料交換給其他程式或行動裝置使用的時候就會遇到 cros 跨網域存取的問題。想像一下如果我們需要在一個手機 app 中呼叫原本程式中的 API 就會遇到。
* CSRF 偽造跨站請求: 我們仍可能會遇到偽造跨站請求的問題，當使用者已經登入的時候 就可能會遇到這類的攻擊。

不過在這些問題中最大的問題還是屬於擴展性。

# token 驗證機制如何運作

基於 token 的驗證機制屬於無狀態的方式，我們不並需要在伺服器儲存任何關於使用者的資料。這個概念本身就是為了處理上述的那些問題。
少了 session 表示我們的程式在使用負載平衡擴展伺服器時就不用因為這些資料交換的問題而受到限制。

雖然實作有很多種方式，不過大致上流程如下:

1. 使用者提供帳密發出驗證請求
2. 程式驗證憑證
3. 程式提供回傳一個簽署過的 token 給客戶端
4. 客戶端儲存 token ，後續請求都必須要一併包含這個 token
5. 伺服器驗證 token 然後才回傳資料

每一道 request 都要包含 token。這個 token 應該要被包含在 HTTP header 中，也因此這種方式達成了 stateless 的特性。接著我們就可以將伺服器的存取限制設定為 `Access-Control-Allow-Origin: *`。關於 `ACAO` header 中設定為 `*` 同時表示這不允許請求提供憑證例如: HTTP 憑證, 客戶端的 SSL 憑證或 cookie。下圖顯示整個流程

![](https://cask.scotch.io/2014/11/tokens-new.png)

一旦通過驗證我們就會擁有 token，我們就可以透過這個 token 做出許多應用。我們可以基於 token 建立授權甚至是傳遞給第三方應用使用。透過 token 我們就可以決定哪些行為可以放行。

# token 的優點

最大的好處就是擴展性和 stateless。因為 stateless 的關係我們的程式隨時都可以導入負載平衡。如果我們使用 session 那麼當使用者登入之後要繼續後續的動作就必須要把資料送回同一台伺服器。然後就可能對伺服器造成壓力。一旦採用 token 那麼這些問題就不用再擔心了。

# 安全性

token 不像 cookie，它會在每一次 request 的時候都要帶一次，因為沒有被存放到 cookie 中也就沒有機會被其他客戶端的程式直接存取。甚至是您將 token 儲存在 cookie 中，cookie 也只是一種儲存方式而不是實際驗證機制。

token 也會過期，所以使用者會被要求在登入一次，這協助我們提高安全性。這裡提到 [token revocation](https://tools.ietf.org/html/rfc7009) 的概念，讓我們可以指定某個 token 失效或者是一整組基於同一個授權的 token。

# 擴展性

token 也可以讓我們的程式共享權限。舉例來說我們透過連結到社群平台驗證登入像是 Facebook 或 Twitter。又例如當我們在 Buffer 給予 Twitter 權限那麼我們就可以透過 Buffer 發文到 Twitter 上。使用 token 我們就可以提供特定權限或功能給第三方的應用程式。

# 跨平台與網域

稍早我們提到 CORS，當我們需要增加其他的服務或程式我們就需要提供存取權限給該網域或程式。又比如我們有一個 API 只負責提供資料，這個時候我們或許就會使用 CDN 來處理 CORS 的問題。如果我們可以直接把伺服器的設定換成

```
Access-Control-Allow-Origin: *
```

我們的資料或服務就可以提供給任何網域或程式來使用，當然你會說有安全性的問題，不過別忘了我們現在有 token 協助我們去驗證使用者。

# 標準

關於建立 token 我們有一些選擇。在下面的章節我們會深入關於 API 安全性的議題，不過主要會介紹 JSON Web Tokens 標準。[jwt.io](https://jwt.io/)提供了一些方便的除錯工具與函式庫支援的圖表。可以看到大部分的語言都有支援。

# JSON Web Token 解析

近年來 API 盛行不管是 Open Data 等，也就造成一個程式想要充分發揮潛能，常常會需要從其他地方取得資料，與第三方的程式整合。想想 Facebook 提供 API 讓我們可以取得資料和登入功能，Facebook 即所謂提供我們這些第三方應用存取其資料的例子。而這些都是透過 API 完成的。

現在當我們討論到關於我們自己打造自己的 API，不管是誰要完成這件事都需要面對一個問題，那就是如何確保 API 的安全性。上面我們已經探討過基於 toke 的驗證機制。現在我們要來討論的事關於 JSON Web Tokens 這個標準以及我們如何建置它。

# JSON Web Token 是啥?

JSON Web Tokens 又稱 `JWT` 發音是 `jot` 從名字不難看出資料是透過 JSON 傳遞的。[這裡](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)有關於 JWT 的規格草稿，不過它並不好讀。
JWT 可以在不同的語言中使用包含 .NET, Python, NodeJS, Java, PHP, Ruby, Go, Haskell 等。所以基本上在各種情況下我們都能夠使用。
JWT 不相依於其他東西是可以獨立使用的: 它會包含自己所有需要的資料，意味著 JWT 是可以會傳送關於自身的基本資料，一個 payload 通常指的是使用者的資訊以及一個 signature 簽章。

JWT 可以被輕易的傳送: 因為 JWT 自身就包含了必須的資料且不依賴其他東西，因此當我們要透過 API 驗證的時候它可以完美的透過 HTTP header 或者網址來傳遞。

# 那麼一個 JWT 到底長啥樣?

要辨別一個 JWT 非常簡單，它就是三個字串透過 `.` 合在一起

```
aaaaaaaaaa.bbbbbbbbbbb.cccccccccccc
```

# 解析一個 JWT

我們很容易的就可以看出這是透過 `.` 連結三個不同的部份它們各自代表的是:

* header: 標頭資訊
* payload: 處理的內容實際資料
* signature: 簽章

![](https://cask.scotch.io/2014/11/json-web-token-overview1.png)

# Header

header 包含兩個部分

* 宣告型別是 JWT
* 使用的演算法，在這個例子中是 `HMAC SHA256`

```js
{
  "typ": "JWT",
  "alg": "HS256"
}
```

一旦將上面的資訊使用 `base64encode` 我們就會得到第一個部分的 token

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

你可以拿著上面的 token 去 `jwt.io` 網站上使用工具試試。

# Payload

關於 payload 大體來說就是需要處理的`實際資料內容`，這個 payload 通常會是整個 JWT 中比較重要的資料又稱 JWT Claims 即紀錄這個 JWT 的主張或者說本次傳輸主要的目的，主張是比較抽象的描述但具體來說就是資料。我們將想要傳送的資料和一些附加的資訊放在這裡。Claims 可以不只一個，為了不產生疑義後續我們將使用英文術語 - JWT Claims。

# Registered Claims

關於 Claims 即傳遞的資料並不是必須的，不過根據規範我們可以使用下面這些預先定義好的 Claims 名稱讓我們可以使用:

* iss: 發行者的 token
* sub: 主題的 token
* aud: 接受者(聽眾)的 token
* exp: 這可能是 Registered Claims 最常用的，定義數字格式的有效期限，重點是有效期限一定要大於現在的時間
* nbf: 生效時間，定義一個時間在這個時間之前 JWT 不能進行處理
* iat: 發行的時間，可以被用來判斷 JWT 已經發出了多久
* jti: JWT 唯一的識別值，可用來防止 JWT 被重複使用，尤其在一次性的 token 特別好用

# Public Claims

我們所建立的公開資訊例如使用者姓名等等。

# Private Claims

發行者與訂閱者自行溝通定義的 Claims Name 與資料

# payload 範例

下面這個範例有兩個 `registered claims`(iss 和 exp)以及兩個 `public claims`(name, admin)

```js
{
  "iss": "andyyou.github.io",
  "exp": 1465700328092,
  "name": "andyyou",
  "admin": true
}
```

第二部分的 JWT 編譯過後就是

```
eyJpc3MiOiJhbmR5eW91LmdpdGh1Yi5pbyIsImV4cCI6MTQ2NTcwMDMyODA5MiwibmFtZSI6ImFuZHl5b3UiLCJhZG1pbiI6dHJ1ZX0
```

# Signature 簽章

第三個部分就是簽章，這個簽章由下面三個部分組成

* header
* payload
* secret

看看範例程式碼便知是如何得到

```js
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload)
HMACSHA256(encodedString, 'secret')
```

`secret` 的部分由伺服器持有，也因此伺服器才有辦法驗證 token 和簽發

```
6srTK4rBbOqlWj7le2hrwFP-iayHblLdhgVFIYU3gVg
```

最終得到的 JWT 就如同一開始提到的由兩個 `.` 串接三個編碼

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhbmR5eW91LmdpdGh1Yi5pbyIsImV4cCI6MTQ2NTcwMDMyODA5MiwibmFtZSI6ImFuZHl5b3UiLCJhZG1pbiI6dHJ1ZX0.6srTK4rBbOqlWj7le2hrwFP-iayHblLdhgVFIYU3gVg
```

總結來說 JWT 可以跨平台跨語言，讓不同的裝置應用或服務之間溝通，同時這個 token 可以搭配 URL, POST 參數, 或者 HTTP 標頭來傳遞使用。
透過 JWT 讓我們可以快速方便的賦予 API 驗證機制。

# Nodejs 實作  

接下來讓我們透過 Node 與 Express 實踐一個簡單的 API 然後透過 POSTman 來測試。這個範例主要的工作流程

1. 個別有公開和需要授權的路由
2. 使用者需要帳密通過驗證取得 token
3. 使用者將 token 儲存在客戶端，之後的每一次請求都將包含此 token
4. 收到請求後會驗證 token ，當 token 正確才回傳資料

### 建構專案目錄結構

首先我們先來看看這個 Node 應用程式的檔案架構，為了單純我們會將大部分的邏輯都寫在 `server.js`

```bash
$ mkdir simple-api
$ cd simple-api
$ npm init --yes
$ mkdir -p app/models
$ touch app/models/user.js
$ touch config.js
$ touch server.js
```

```
├── app
│   └── models
│       └── user.js
├── config.js
├── package.json
└── server.js
```

> 在實務上您應該盡可能分拆與組織檔案，而不是把全部的邏輯都放在一起。

### 設定

設定 `package.json`

```js
{
  "name": "simple-api",
  "version": "1.0.0",
  "main": "server.js"
}
```

接著安裝我們需要用到的套件與函式庫

```bash
$ npm i express body-parser morgan mongoose jsonwebtoken -S

# express: 輕量化的 web framework
# mongoose: MongoDB 的 ORM 套件，我們使用它來操作資料庫
# morgan: HTTP request logger，會於 console 輸出 request 的資訊
# body-parser: 協助我們取得 POST request 的資料
# jsonwebtoken: 建置/驗證 JWT
```

### 設定 Model

我們需要定義一個使用者的資料模型 Model，之後會用它來建立使用者資料，這個步驟是因為我們使用 MongoDB 搭配 Mongoose，所以我們要建立 `app/models/user.js` 來定義 Model, Schema。

```js
var mongoose = require('mongoose')
var Schema = mongoose.Schema

module.exports = mongoose.model('User', new Schema({
  name: String,
  password: String,
  admin: Boolean
}))
```

### 應用程式設定檔 (config.js)

通常程式會有一些設定的資料像是連線字串, secret 加密字串, 這些資料我們通常會彙整在一個檔案以方便日後調整。

```js
module.exports = {
  'secret': 'ilovera',
  'database': 'mongodb://localhost/jwt_dev'
}

// secret: 加密與驗證 token
// database: 連線字串
```

到這一步我們已經完成大部分的前置作業，接著要進入核心部分 `server.js`

### Node 應用程式

在這一隻檔案中我們將會

* 載入套件與資料模型等就是我們先前安裝的那些 express, body-parser, morgan 和 Model 的部分
* 載入與配置設定
* 建立基本路由
* 建立 API 路由
  - `POST http://localhost:8080/api/authenticate` 確認帳密是否與資料庫吻合，如果正確就回傳 token 這個路由本身不需要驗證
  - `GET http://localhost:8080/api/api` 隨機回傳資訊，這個路由需要授權也就是需要 token 才能存取
  - `GET http://localhost:8080/api/users` 列出所有使用者，同樣這個路由也需要授權

這支程式的基本功能

```js
// 載入 server 程式需要的相關套件
var express = require('express')
var app = express()
var bodyParser = require('body-parser')
var morgan = require('morgan')
var mongoose = require('mongoose')

// 載入 jwt 函式庫協助處理建立/驗證 token
var jwt = require('jsonwebtoken')
// 載入設定
var config = require('./config')
// 載入資料模型
var User = require('./app/models/user')

var port = process.env.PORT || 8080
mongoose.connect(config.database)
app.set('secret', config.secret)

// 套用 middleware
app.use(bodyParser.urlencoded({extended: false}))
app.use(bodyParser.json())
app.use(morgan('dev'))


app.get('/', function (req, res) {
  res.send('Hi, The API is at http://localhost:' + port + '/api')
})

app.listen(port, function () {
  console.log('The server is running at http://localhost:' + port)
})
```

完成檔案之後可以使用 `node server.js` 來測試，瀏覽 `http://localhost:8080` 應該要能看到 `Hi, The API is at http://localhost:8080/api` 的文字。因為我們有使用 `morgan` 所以應該也要能在 console 看到像 `GET / 200 2.846 ms - 43` 的資訊。

### 建立使用者帳號

OK! 現在我們的程式能運作了，讓我們先來建立一組帳密。因為我們的主題是要了解 jwt 的運作流程所以這邊為了方便我們直接透過呼叫一個路由 `/setup` 直接建立一組帳密。在 `app.get('/')` 路由下面新增一組路由

```js
app.get('/setup', function (req, res) {
  var andyyou = new User({
    name: 'andyyou',
    password: '12345678',
    admin: true
  })
  andyyou.save(function (err) {
    if (err) throw err

    console.log('User saved successfully')
    res.json({success: true})
  })
})
```

> 值得注意的是在正式環境，密碼的部分不應該像上面的作法直接儲存明碼，應要加密。

接著重啟 server `node server.js` 瀏覽 `http://localhost:8080/setup` 就會建立使用者。查看資料庫應會如下圖

![](http://i.imgur.com/AGmSNwE.png)

### 顯示使用者

現在我們可以來實作使用者列表的 API 了，在這一步我們會使用 `express.Router()` 這麼做的好處是之後我們可以把整包相關的路由掛載到某個前綴路徑上。比如說我們有 `/`, `/users`, `edit` 等路由，透過 `app.use('/api', 路由實力物件)` 的方式產出 `/api`, `/api/users`, `/api/edit`

```js
var api = express.Router()
// TODO: authenticate
// TODO: verify token

api.get('/', function (req, res) {
  res.json({message: 'Welcome to the APIs'})
})

api.get('/users', function (req, res) {
  User.find({}, function (err, users) {
    res.json(users)
  })
})
app.use('/api', api);
```

每一次修改程式碼都需要重啟，不然我們就需要使用像是 `nodemon`

```bash
$ npm install -g nodemon
$ nodemon server.js
```

剛剛我們加了兩道 API 現在我們可以透過 `postman` 來測試看看

[Imgur](http://i.imgur.com/LcwM9SM.png)

[Imgur](http://i.imgur.com/gMhojhw.png)

現在我們可以顯示出列表了但在實務上我們大概都不希望任何人都能存取我們的這些資料，接著就是重點我們要透過驗證來保護這些資料只讓特定的使用者查詢。

### 驗證機制

透過 `http://localhost:8080/api/authenticate` 路由我們將允許那些通過驗證的人存取，這個過程我們會驗證帳密，然後建立派發 token
一旦使用者獲得 token，就會將其暫存在客戶端，之後每一個 request 都需要這個 token。接著伺服器端會透過 middleware 的方式在每個存取的過程驗證。

```js
api.post('/authenticate', function (req, res) {
  User.findOne({
    name: req.body.name
  }, function (err, user) {
    if (err) throw err

    if (!user) {
      res.json({ success: false, message: 'Authenticate failed. User not found'})
    } else if (user) {
      if (user.password != req.body.password) {
        res.json({ success: false, message: 'Authenticate failed. Wrong password'})
      } else {
        var token = jwt.sign(user, app.get('secret'), {
          expiresIn: 60*60*24
        })

        res.json({
          success: true,
          message: 'Enjoy your token',
          token: token
        })
      }
    }
  })
})
```

接著透過 postman 測試我們剛完成的功能，首先 method 要換成 `POST`，然後參數的部份要切到 `Body` 使用 `x-www-form-urlencoded`。這個步驟我們使用 postman 模擬實際在網頁中送出 post 的過程。可以順便測試一下當密碼錯誤時是否回傳正確的資訊。

[Imgur](http://i.imgur.com/WrBVjQe.png)

### 驗證 token

到了這一步我們已經實作了三道路由 `/api/authenticate`, `/api`, `/api/users`。最後的任務自然是要存取的過程時要驗證 token。現在我們需要建立路由的 middleware ，賦予 API驗證機制，其中 `/api/authenticate` 是不需要被保護的。需要注意的是 middleware 擺放的位置會影響是否要驗證。
要驗證的路由必須要 middleware 之後。

```js
var api = express.Router()

// authenticate api

api.use(function (req, res, next) {
  var token = req.body.token || req.query.token || req.headers['x-access-token']
  if (token) {
    jwt.verify(token, app.get('secret'), function (err, decoded) {
      if (err) {
        return res.json({success: false, message: 'Failed to authenticate token.'})
      } else {
        req.decoded = decoded
        next()
      }
    })
  } else {
    return res.status(403).send({
      success: false,
      message: 'No token provided.'
    })
  }
})

// Others APIs
// /api
// /api.users

app.use('/api', api);
```

這個過程我們使用 `jsonwebtoken` 套件來協助我們處理驗證，重要的是那個 secret 要跟我們派發時的一致。經過 middleware 的處理我們就可以判斷使用者是否通過驗證。

### 測試 middleware

經過 middleware 中介軟體的處理我們在 api 的每一道路由(在設定 middleware 之後)都會先執行 middleware。於是我們就在這個過程處理 token 判斷是否要放行。
現在我們就可以再次使用 postman 來測試，因為 jwt 的傳遞可以透過 header, body, URL Query 三種方式所以在 postman 中我們可以切換到 `GET` 然後使用 header 標頭方式來測試

* without token

[Imgur](http://i.imgur.com/SD956Gz.png)

* with token

[Imgur](http://i.imgur.com/vJrUPMY.png)

當然我們也能透過 `query string` 的方式來通過驗證。通過實作我們大致理解了 API 驗證在 Node 的實作思路，下面附上完整 server.js 程式碼

```js
var express = require('express')
var app = express()
var bodyParser = require('body-parser')
var morgan = require('morgan')
var mongoose = require('mongoose')

var jwt = require('jsonwebtoken')
var config = require('./config')
var User = require('./app/models/user')

var port = process.env.PORT || 8080
mongoose.connect(config.database)
app.set('secret', config.secret)

app.use(bodyParser.urlencoded({extended: false}))
app.use(bodyParser.json())
app.use(morgan('dev'))


app.get('/', function (req, res) {
  res.send('Hi, The API is at http://localhost:' + port + '/api')
})

app.get('/setup', function (req, res) {
  var andyyou = new User({
    name: 'andyyou',
    password: '12345678',
    admin: true
  })
  andyyou.save(function (err) {
    if (err) throw err

    console.log('User saved successfully')
    res.json({success: true})
  })
})

var api = express.Router()

api.post('/authenticate', function (req, res) {
  User.findOne({
    name: req.body.name
  }, function (err, user) {
    if (err) throw err

    if (!user) {
      res.json({ success: false, message: 'Authenticate failed. User not found'})
    } else if (user) {
      if (user.password != req.body.password) {
        res.json({ success: false, message: 'Authenticate failed. Wrong password'})
      } else {
        var token = jwt.sign(user, app.get('secret'), {
          expiresIn: 120
        })

        res.json({
          success: true,
          message: 'Enjoy your token',
          token: token
        })
      }
    }
  })
})

api.use(function (req, res, next) {
  var token = req.body.token || req.query.token || req.headers['x-access-token']
  if (token) {
    jwt.verify(token, app.get('secret'), function (err, decoded) {
      if (err) {
        return res.json({success: false, message: 'Failed to authenticate token.'})
      } else {
        req.decoded = decoded
        next()
      }
    })
  } else {
    return res.status(403).send({
      success: false,
      message: 'No token provided.'
    })
  }
})

api.get('/', function (req, res) {
  res.json({message: 'Welcome to the APIs'})
})

api.get('/users', function (req, res) {
  User.find({}, function (err, users) {
    res.json(users)
  })
})

app.use('/api', api)

app.listen(port, function () {
  console.log('The server is running at http://localhost:' + port)
})
```

# 觀察不同套件產生之 Token

**jsonwebtoken 產生的 token 範例**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyIkX18iOnsic3RyaWN0TW9kZSI6dHJ1ZSwiZ2V0dGVycyI6e30sIndhc1BvcHVsYXRlZCI6ZmFsc2UsImFjdGl2ZVBhdGhzIjp7InBhdGhzIjp7Il9fdiI6ImluaXQiLCJhZG1pbiI6ImluaXQiLCJwYXNzd29yZCI6ImluaXQiLCJuYW1lIjoiaW5pdCIsIl9pZCI6ImluaXQifSwic3RhdGVzIjp7Imlnbm9yZSI6e30sImRlZmF1bHQiOnt9LCJpbml0Ijp7Il9fdiI6dHJ1ZSwiYWRtaW4iOnRydWUsInBhc3N3b3JkIjp0cnVlLCJuYW1lIjp0cnVlLCJfaWQiOnRydWV9LCJtb2RpZnkiOnt9LCJyZXF1aXJlIjp7fX0sInN0YXRlTmFtZXMiOlsicmVxdWlyZSIsIm1vZGlmeSIsImluaXQiLCJkZWZhdWx0IiwiaWdub3JlIl19LCJlbWl0dGVyIjp7ImRvbWFpbiI6bnVsbCwiX2V2ZW50cyI6e30sIl9ldmVudHNDb3VudCI6MCwiX21heExpc3RlbmVycyI6MH19LCJpc05ldyI6ZmFsc2UsIl9kb2MiOnsiX192IjowLCJhZG1pbiI6dHJ1ZSwicGFzc3dvcmQiOiIxMjM0NTY3OCIsIm5hbWUiOiJhbmR5eW91IiwiX2lkIjoiNTc1Y2UxYTBkY2I1NDUwZDFhNzIwZjljIn0sIl9wcmVzIjp7IiRfX29yaWdpbmFsX3NhdmUiOltudWxsLG51bGxdfSwiX3Bvc3RzIjp7IiRfX29yaWdpbmFsX3NhdmUiOltdfSwiaWF0IjoxNDY1NzMxNjU4LCJleHAiOjE0NjU3MzE3Nzh9.toeuSCZ6M0XcjS1p2Tn1h30HQIOQmrBKRsAsgaWvx9g
```
**jsonwebtoken token 解析 payload 部分**

```
{
  "$__": {
    "strictMode": true,
    "getters": {},
    "wasPopulated": false,
    "activePaths": {
      "paths": {
        "__v": "init",
        "admin": "init",
        "password": "init",
        "name": "init",
        "_id": "init"
      },
      "states": {
        "ignore": {},
        "default": {},
        "init": {
          "__v": true,
          "admin": true,
          "password": true,
          "name": true,
          "_id": true
        },
        "modify": {},
        "require": {}
      },
      "stateNames": [
        "require",
        "modify",
        "init",
        "default",
        "ignore"
      ]
    },
    "emitter": {
      "domain": null,
      "_events": {},
      "_eventsCount": 0,
      "_maxListeners": 0
    }
  },
  "isNew": false,
  "_doc": {
    "__v": 0,
    "admin": true,
    "password": "12345678",
    "name": "andyyou",
    "_id": "575ce1a0dcb5450d1a720f9c"
  },
  "_pres": {
    "$__original_save": [
      null,
      null
    ]
  },
  "_posts": {
    "$__original_save": []
  },
  "iat": 1465731658,
  "exp": 1465731778
}
```

** jwt-simple token sample**

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJfaWQiOiI1NzVjZTFhMGRjYjU0NTBkMWE3MjBmOWMiLCJuYW1lIjoiYW5keXlvdSIsInBhc3N3b3JkIjoiMTIzNDU2NzgiLCJhZG1pbiI6dHJ1ZSwiX192IjowfQ.pZ3KHGGZwjgDryrLrAzKkfmTv4xJbekQGX3UCe8Fm1Q
```
** jwt-simple token 解析 payload**

```
{
  "_id": "575ce1a0dcb5450d1a720f9c",
  "name": "andyyou",
  "password": "12345678",
  "admin": true,
  "__v": 0
}
```

# 小結

透過上面的實作，我們對於 jwt 和如何實作有了基本的理解。

# 參考資料來源

* [The Ins and Outs of Token Based Authentication](https://scotch.io/tutorials/the-ins-and-outs-of-token-based-authentication#who-uses-token-based-authentication)
* [The Anatomy of a JSON Web Token](https://scotch.io/tutorials/the-anatomy-of-a-json-web-token)
* [Authenticate a Node.js API with JSON Web Tokens](https://scotch.io/tutorials/authenticate-a-node-js-api-with-json-web-tokens)
* [Build an App with Vue.js: From Authentication to Calling an API](https://auth0.com/blog/2015/11/13/build-an-app-with-vuejs/)
* [Session 介紹](http://www.haomou.net/2014/08/07/2014_session/)
* [使用 JWT](http://haomou.net/2014/08/13/2014_web_token/)
* [OAuth 2.0 筆記](https://blog.yorkxin.org/posts/2013/09/30/oauth2-1-introduction/)
* [Postman 使用筆記](http://luciastar.com/2016/05/21/postman%E7%AC%94%E8%AE%B0/)

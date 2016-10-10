---
layout: post
title: "Vue 實作簡易驗證機制 App"
date: 2016-06-18 12:00:00
categories: Program
tags: vuejs, javascript
---

為了顯示出 Vue.js 強大的能力 ，本文將會逐步指導建置一個簡單的前端應用程式。搭配 Node 所建置的後端程式範例。前後端兩個程式是完全分離的，後端使用 RESTful API 的方式負責取得資料與驗證。本文旨在說明如何替 Vue.js 程式加上驗證機制，過程中我們會使用 `vue-router`, `vue-resource` 實作 `Login` 與 `Signup` 元件展示如何檢索和儲存使用者的 `jwt token`，最後執行驗證機制取得那些需要授權的資料。

<!--more-->

# 安裝與設定

首先從組織前端程式開始，讓我們先來建立專案：

```bash
$ mkdir auth-front-app
$ cd auth-front-app
$ npm init --yes
```

安裝我們所需要的 npm 套件，我們會在這個專案使用 Babel, Webpack 負責建置工具的部份：

```bash
$ npm i babel-core babel-loader babel-plugin-transform-runtime babel-preset-es2015 babel-runtime css-loader style-loader vue-hot-reload-api vue-html-loader vue-style-loader vue-loader webpack webpack-dev-server -D

$ npm i bootstrap vue-resource vue-router vue -S
```

package.json 如下：

```json
{
  "name": "auth-front-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "license": "MIT",
  "devDependencies": {
    "babel-core": "^6.10.4",
    "babel-loader": "^6.2.4",
    "babel-plugin-transform-runtime": "^6.9.0",
    "babel-preset-es2015": "^6.9.0",
    "babel-runtime": "^6.9.2",
    "css-loader": "^0.23.1",
    "style-loader": "^0.13.1",
    "vue-hot-reload-api": "^1.3.3",
    "vue-html-loader": "^1.2.3",
    "vue-loader": "^8.5.3",
    "vue-style-loader": "^1.0.0",
    "webpack": "^1.13.1",
    "webpack-dev-server": "^1.14.1"
  },
  "dependencies": {
    "bootstrap": "^3.3.6",
    "vue": "^1.0.25",
    "vue-resource": "^0.8.0",
    "vue-router": "^0.7.13"
  }
}
```

安裝完套件之後就是專案目錄下新增 `webpack.config.js` 設定檔：

```js
var path = require('path')

module.exports = {
  entry: ['./src/index.js', './src/auth/index.js'],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.vue$/,
        loader: 'vue'
      },
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      }
    ]
  },
  /**
   * option: 您可以選擇把下面這段設定置於 .babelrc
   */
  babel: {
    presets: ['es2015'],
    plugins: ['transform-runtime']
  }
}
```

設定檔中首要的就是我們程式的進入點，我們設定了 `src/index.js` 和 `src/auth/index.js` 這兩支程式，最後會被輸出成 `bundle.js`。到這一步先別擔心我們還沒完成任何程式，習慣上小弟我會先構想整個專案架構，透過組織 webpack.config 可以讓我們先好好思考一下。

因為我們想使用 `vue 元件` 的方式組織程式碼。於是設定了負責處理 `.vue` 的部分，即 `vue-loader` 協助編譯。
到此便是我們目前所需的設定，最後我們只要透過 `webpack-dev-server --inline --hot` 就可觀察開發的前端程式。

> Module not found: Error: Cannot resolve 因為到目前為止我們只是預先規劃兩隻進入點的程式還沒實作，所以執行 webpack-dev-server 會產生錯誤。

# 設定後端程式

因為本篇文章主要是要討論關於 Vue.js 處理驗證機制的作法，所以我們在後端部分採用 [nodejs-jwt-authentication-sample](https://github.com/auth0/nodejs-jwt-authentication-sample)。使用概略如下：

```bash
# 記得切換到另外的目錄
$ git clone git@github.com:auth0-blog/nodejs-jwt-authentication-sample.git
$ cd nodejs-jwt-authentication-sample
$ npm install

# 啟動後端程式
$ PORT=3001 node server.js

# 建立使用者 POST /users
# 注意：username 和 password 必須替換
$ curl -d "username=[replace_me]&password=[replace_me]" http://localhost:3001/users
# 建立成功應會回傳 { "id_token": "..." }

# 登入 POST /sessions/create
$ curl -d "username=[replace_me]&password=[replace_me]" http://localhost:3001/sessions/create
# 登入成功應會回傳 { "id_token": "..." }

# 不需授權 API: http://localhost:3001/api/random-quote
# 受保護 API: http://localhost:3001/api/protected/random-quote
```

> 注意這個後端範例的帳號資料會放在記憶體中，重啟就會消失。

# 配置 Vue 元件

現在可以來替我們的應用程式建置元件。在專案配置一節我們安裝與使用了 `vue-loader` 然後提到它讓我們可以使用 `.vue` 的方式來撰寫元件。具體來說就是我們在一隻 `[component_name].vue` 的檔案中分別撰寫 `<template>`, `<script>`, `<style>` ，最終這隻檔案會被輸出成一個元件供我們組合或使用。

```bash
# 在專案目錄下
$ mkdir -p src/components
$ touch src/components/Home.vue
```

Home 元件主要透過 API 去得不須授權的資料並顯示 `src/components/Home.vue` 元件程式碼如下

```html
<!-- src/components/Home.vue -->

<template>
  <div class="col-sm-6 col-sm-offset-3">
    <h1>取得不需授權的 Chunck Norris 名言</h1>
    <button class="btn btn-primary" @click="getQuote">取得名言</button>
    <div class="quote-area" v-if="quote">
      <h2><blockquote>{{quote}}</blockquote></h2>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      quote: ''
    }
  },
  methods: {
    getQuote () {
      this.$http.get('http://localhost:3001/api/random-quote')
        .then((res) => {
          this.quote = res.data
        })
        .catch((err) => { console.log(err) })
    }
  }
}
</script>
```

`<template>` 裡的就是我們要顯示的 HTML 標籤結構，裡面有一個按鈕用來呼叫 `getQuote`，還有一些看起來像 `Angular 1.x` 的特殊屬性，它們是 Vue 的 `directive`，像是 `@click`, `v-if` 這些都是，而 `@click` 又可以寫成 `v-on:click` 當點擊的時候會觸發我們綁定的事件(從 methods 來)，`v-if` 可以根據綁定的資料 `quote` 來決定 `<div class="quote-area">` 是否要輸出顯示。當然 Vue 也可以在樣板中使用 `{ {} }` 的語法用來作資料綁定(從 data 來)。

`<script>` 的部分會匯出一個 JS 物件，接著會被 Vue 轉換為一個 Vue 元件。大體來說 `data` 可以提供我們作資料繫結，`methods` 可以協助我們綁定一些互動的事件。`getQuote` 中的 `this.$http` 則是從 `vue-resource` 中加入的功能。

目前為止程式仍無法運作，不過我們簡單的介紹了一個 `.vue` 元件長啥樣和一些語法的作用。詳細的用法還是需要花點時間閱讀手冊。

# 主程式 index.js 和 App.vue

`index.js` 是程式主要的進入點，我們會在這邊匯入元件，設定路由等等等。為了單純起見，整個程式所有需要的設定都會放在這隻檔案中。

新增 `src/index.js`

```js
import Vue from 'vue'
import App from './components/App.vue'
import Home from './components/Home.vue'
import SecretQuote from './components/SecretQuote.vue'
import Signup from './components/Signup.vue'
import Login from './components/Login.vue'
import VueRouter from 'vue-router'
import VueResource from 'vue-resource'

// 替 Vue 掛上 HTTP Request 的功能
Vue.use(VueResource)

// 替 Vue 掛上路由的功能
Vue.use(VueRouter)

// export 是為了讓其他分離的程式碼也能取得路由的物件實例
export var router = new VueRouter()

// 定義路由
router.map({
  '/home': {
    component: Home
  },
  '/secretquote': {
    component: SecretQuote
  },
  'login': {
    component: Login
  },
  'signup': {
    component: Signup
  }
})

router.redirect({
  '*': '/home'
})

auth.check()
// 啟動路由並將 root component 掛載到 HTML 中 id="app" 的 DOM 上
router.start(App, '#app')
```

現在我們匯入了一些 Vue 元件大致上讓我們理解該怎麼使用元件與 `vue-router`，但注意到我們還未實作任何程式碼。

`vue-router` 和 `vue-resource` 需要透過 `Vue.use()` 將功能附加到 Vue 中。同時我們也定義了一些路由，理解 `vue-router` 中一個路由可以對應一個元件。

接著我們便可以開始完善這些元件。第一個是我們的根元件 `App.vue`

```html
<!-- src/components/App.vue -->

<template>
  <div>
  <!-- 注意：少了最外層的 wrapper 會出現
      Attribute "id" is ignored on component 警告
  -->
  <nav class="navbar navbar-default">
    <div class="container">
      <ul class="nav navbar-nav">
        <li><a v-link="'home'">Home</a></li>
        <li><a v-link="'login'">Login</a></li>
        <li><a v-link="'signup'">Signup</a></li>
        <li><a v-link="'secretquote'">Secret Quote</a></li>
        <li><a v-link="'login'">Logout</a></li>
      </ul>
    </div>
  </nav>
  <div class="container">
    <router-view></router-view>
  </div>
  </div>
</template>
```

在 `App.vue` 元件中我們組織了最外層的結構，我們需要一個 `navbar` 來切換，而更換的元件會顯示在 `<router-view></router-view>` 位置。切換頁面的連結只要使用 `v-link` 屬性即可。最後我們需要一個 `index.html` 來載入 `bundle.js` 以及 `app` 的掛載點。

> 注意到 v-link 中的 `"'home'"`，v-link 有三種設定方式請參考[說明文件](http://router.vuejs.org/zh-cn/basic.html)

新增 `index.html`

```html
<!-- index.html -->

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Authenticate Vue App</title>
  <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.min.css">
</head>
<body>
  <div id="app"></div>
  <script src="bundle.js"></script>
</body>
</html>
```

到這一步如果您很著急想看看到底實作了些什麼，您可以先把 `webpack.config` 中的 `src/auth/index.js` 移除並在 `src/index.js` 把還沒實作的元件註解。

執行 `webpack-dev-server --inline --hot` 應該可以看到如下圖

![](http://i.imgur.com/UHC4zAc.png)

# 驗證機制

為了讓使用者登入我們需要發送一個 HTTP 請求給後端程式驗證，然後儲存回傳的 jwt 到 `localStorage`。我們可以將這段邏輯放到 Login 元件裡，不過為了重複使用，我們會將這些驗證邏輯抽出來放到 `src/auth/index.js` 中。

新增 `src/auth/index.js` 撰寫程式碼如下

```js
// 看到我們之前鋪的梗了吧，exprt router 就是這邊使用。
import {router} from '../index'

const API_URL = 'http://localhost:3001'

export default {
  user: {
    authenticated: false
  },
  login (context, creds, redirect) {
    var url = API_URL + '/sessions/create'

    context.$http.post(url, creds)
      .then((res) => {
        localStorage.setItem('id_token', res.data.id_token)
        this.user.authenticated = true
        console.log('Login successfully', res.data.id_token)
        if (redirect) {
          router.go(redirect)
        }
      })
      .catch((err) => {
        console.log(err)
        context.error = err.data
      })
  },
  signup (context, creds, redirect) {
    var url = API_URL + '/users'

    context.$http.post(url, creds)
      .then((res) => {
        localStorage.setItem('id_token', res.data.id_token)
        this.user.authenticated = true

        if (redirect) {
          router.go(redirect)
        }
      })
      .catch((err) => {
        console.log(err)
        context.error = err.data
      })
  },
  logout () {
    localStorage.removeItem('id_token')
    this.user.authenticated = false
  },
  check () {
    /**
     * 這邊只是單純示範，您不該在驗證邏輯單純只判斷是否有 token
     */
    var jwt = localStorage.getItem('id_token')
    if (jwt) {
      this.user.authenticated = true
    } else {
      this.user.authenticated = false
    }
  },
  getAuthHeader () {
    return {
      'Authorization': 'Bearer ' + localStorage.getItem('id_token')
    }
  }
}
```

還記得我們在 `src/index.js` 中有匯出 `export var router` 所以這邊可以取得 router 的物件實例。
這隻 `src/auth/index.js` 匯出一些 methods 協助我們執行登入，登出，驗證帳密，註冊的行為。
登入只負責取回 jwt 然後儲存，如果帳密驗證失敗整個請求就會出現錯誤，自然就不會登入成功。最後我們透過這些方法和屬性來處理前端關於`登入的相關邏輯`，例如使用 `user.authenticated` 屬性來判斷是否登入。

# 實作 Login 元件

Login 元件中我們需要兩個 `<input>` 來輸入帳密然後使用 `auth` 去驗證是否成功登入。

新增 `src/components/Login.vue`

```html
<!-- src/components/Login.vue -->

<template>
  <div class="col-sm-4 col-sm-offset-4">
    <h2>登入</h2>
    <p>
      登入帳號取得更棒的名言。
    </p>
    <div class="alert alert-danger" v-if="error">
      {{error}}
    </div>
    <div class="form-group">
      <input
        type="text"
        class="form-control"
        placeholder="帳號"
        v-model="credentials.username"
      >
    </div>
    <div class="form-group">
      <input
        type="password"
        class="form-control"
        placeholder="密碼"
        v-model="credentials.password"
      >
    </div>
    <button class="btn btn-primary" @click="submit">登入並存取</button>
  </div>
</template>

<script>
import auth from '../auth'
export default {
  data () {
    return {
      credentials: {
        username: '',
        password: ''
      },
      error: ''
    }
  },
  methods: {
    submit () {
      var credentials = {
        username: this.credentials.username,
        password: this.credentials.password
      }
      auth.login(this, credentials, 'secretquote')
    }
  }
}
</script>
```

HTTP 請求是透過 `vue-reource` 來完成的，由於我們把 `auth` 抽出去，所以在第一個參數部分，我們需要把 `context` 也就是該元件的物件實例(instance)傳過去，如此一來在發生錯誤的時候我們也才能夠把錯誤訊息傳回去。第二個參數是使用者登入所需的驗證資料，第三個參數則是完成登入後要切換的頁面路由。

緊接著 Signup 元件的部份跟 Login 幾乎一樣，只是使用的 `auth` 方法不同。

```html
<!-- src/components/Signup.vue -->

<template>
  <div class="col-sm-4 col-sm-offset-4">
    <h2>註冊</h2>
    <p>
      建立一組新帳號吧！
    </p>
    <div class="alert alert-danger" v-if="error">
      {{error}}
    </div>
    <div class="form-group">
      <input
        type="text"
        class="form-control"
        placeholder="帳號"
        v-model="credentials.username"
      >
    </div>
    <div class="form-group">
      <input
        type="password"
        class="form-control"
        placeholder="密碼"
        v-model="credentials.password"
      >
    </div>
    <button class="btn btn-primary" @click="submit">註冊並存取</button>
  </div>
</template>

<script>
import auth from '../auth'
export default {
  data () {
    return {
      credentials: {
        username: '',
        password: ''
      },
      error: ''
    }
  },
  methods: {
    submit () {
      var credentials = {
        username: this.credentials.username,
        password: this.credentials.password
      }
      auth.signup(this, credentials, 'secretquote')
    }
  }
}
</script>
```

# 實作 SecretQuote 元件 - 取得受保護的資料

使用者需要登入成功之後就才能存取 `secret-quote` 路由，`SecretQuote` 元件看起來跟 Home 很像，不過在呼叫 API 的過程需要使用 jwt token。

```html
<!-- src/components/SecretQuote.vue -->

<template>
  <div class="col-sm-6 col-sm-offset-3">
    <h1>取得需要授權的名言</h1>
    <button class="btn btn-warning" @click="getQuote()">取得名言</button>
    <div class="quote-area" v-if="quote">
      <h2><blockquote>{{quote}}</blockquote></h2>
    </div>
  </div>
</template>

<script>
import auth from '../auth'

export default {
  data () {
    return {
      quote: ''
    }
  },
  methods: {
    getQuote () {
      this.$http.get('http://localhost:3001/api/protected/random-quote', null, {
          headers: auth.getAuthHeader()
        })
        .then( (res) => {
          this.quote = res.data
        })
        .catch((err) => {
          console.log(err)
        })
    }
  },
  route: {
    canActivate () {
      return auth.user.authenticated
    }
  }
}
</script>
```

透過傳入 `options` 參數我們可以附加資料到請求的 `header` 中。而 jwt header 可以透過 `auth.getAuthHeader` 來取得。
最關鍵的部分，因為我們不希望在使用者未登入時能夠存取該路由。所以在這一步我們就會透過 `vue-router` 的 `canActivate` hook 處理。

# 完成剩餘的部分

最後針對不同狀態處理一些是否該顯示的地方完善我們的程式碼 `App.vue`

```html
<!-- src/components/App.vue -->

<template>
  <div>
  <!-- 注意：少了最外層的 wrapper 會出現
      Attribute "id" is ignored on component 警告
  -->
  <nav class="navbar navbar-default">
    <div class="container">
      <ul class="nav navbar-nav">
        <li><a v-link="'home'">Home</a></li>
        <li><a v-link="'login'" v-if="!user.authenticated">Login</a></li>
        <li><a v-link="'signup'" v-if="!user.authenticated">Signup</a></li>
        <li><a v-link="'secretquote'" v-if="user.authenticated">Secret Quote</a></li>
        <li><a v-link="'login'" v-if="user.authenticated" @click="logout">Logout</a></li>
      </ul>
    </div>
  </nav>
  <div class="container">
    <router-view></router-view>
  </div>
  </div>
</template>

<script>
import auth from '../auth'

export default {
  data () {
    return {
      user: auth.user
    }
  },
  methods: {
    logout () {
      auth.logout()
    }
  }
}
</script>
```

# 參考資源

* [Build an app with Vuejs](https://auth0.com/blog/2015/11/13/build-an-app-with-vuejs/)

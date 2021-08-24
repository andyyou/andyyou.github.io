---
title: 使用 Passport 實作 Nodejs 應用程式驗證機制
tags:
  - javascript express passport
date: 2017-04-11 09:30:50
---


為應用程式實作穩固的驗證機制一直以來都是令人害怕的工作，當然在 Nodejs 的世界中也不例外。

在這篇文章，我們將要從零開始開發一個 Nodejs 的應用程式搭配一套熱門的驗證 Middleware - [passport](http://passportjs.org/)。
passport 本身單純只關注驗證相關的部分，在官方文件中它這麼描述：這是一套簡單，可無縫整合於 Node 的 Middlewre。理解它本質是 Middleware 有助於後續的應用。

<!--more-->

透過 Middleware 的機制，passport 將驗證所需的工作抽離以達到關注點分離的特點。同時這讓 passport 可以簡單的設定任何基於 express 的應用程式，就像我們設定其他 express middleware 一樣，例如 [logging](https://github.com/expressjs/morgan)，[body-parser](https://github.com/expressjs/body-parser)，[cookie-parser](https://github.com/expressjs/cookie-parser) 等等。


這裡我們假設您對於 Nodejs 和 express 有些基本的認識，我們將專注在驗證機制的部分。

# 驗證機制/策略 (Authentication Strategies)

passport 提供了 300 多種驗證機制讓我們選擇。我們可以使用本地端或遠端的資料庫來驗證用戶身分又或者使用 OAuth 來完成單一登入 (SSO) 的功能例如 Facebook，Twitter，Google 等等。我們可以從[驗證機制列表](https://github.com/expressjs/cookie-parser)中選擇 passport 有支援的 Node 模組。

小結來說：passport 是一個 Middleware，負責處理身分驗證，其將 `驗證邏輯` 抽離並稱其為 `驗證策略 Stratety` 例如：使用本地端的資料庫驗證是一種策略，Facebook 登入也是一種，我們可以依據需求使用。

先別擔心，我們只需要安裝我們需要的東西即可。每個驗證策略模組都是獨立的，當我們安裝 passport 時，預設`不會`幫我們安裝任何一種。

在這篇文章我們將會使用最常見的方式 - 本地端驗證策略搭配我們存在本地端 MongoDB 資料庫的帳號。為了要使用本地端驗證機制，我們需要安裝 `passport-local`。

OK，到這邊我們理解了， `passport` 需要搭配不同的`驗證策略`，這裡我們使用 `passport-local`。

在開始之前，我們還需要一個 express 應用程式。讓我們打開終端機，建立一個 express 應用程式。這個應用程式會有包含登入，註冊，首頁的功能。注意，我們將要使用 express 4 來實作。

# 設定應用程式

如果您還沒安裝 express 產生器請先跟著[官方文件](https://expressjs.com/en/starter/generator.html)安裝。然後我們執行下面的指令建立專案。

```bash
$ express express-mongodb-passport
```

![](http://imgur.com/a/KQO2K.png)

為了讓事情單純，不被混淆。我們要移除一些預設 generator 產生的但我們不需要的東西 - 刪除 `routes/users.js` 和 `app.js` 中有使用 `routes/users.js` 部分的程式碼。

# 安裝 passport 和相關模組

在終端機上，切換到我們的專案目錄下安裝 `passport`，`passport-local` 和預設的模組。

```bash
$ npm i
$ npm i passport passport-local -S
```

因為我們要把使用者帳號等資訊存在 MongoDB 中。為了方便，這邊我們會使用 `mongoose` 這個 ODM 函式庫來協助我們操作 MongoDB。

一樣讓我們來安裝。

```bash
$ npm i mongoose -S
```

現在，我們已經安裝好相依的函式庫了，讓我們執行 `npm start` 來看看程式是否可以正常運作。

```bash
$ npm start
```

啟動 Server 之後我們用瀏覽器查看 `http://localhost:3000` 應該要能看到 express 預設的首頁。

雖然畫面上除了簡單的文字，什麼都沒有。沒關係！我們會接著完成 `註冊`，`登入`並使用 passport 驗證已註冊的使用者。

# 建立 mongoose 資料模型

因為我們要使用 mongoose 來將使用者帳號資料等存在 MongoDB，要使用 ODM 第一步我們需要先定義資料模型。

在專案目錄下建立 `models/user.js`。

```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;
var UserSchema = new Schema({
  username: String,
  password: String,
  firstname: String,
  lastname: String
});

module.exports = mongoose.model('User', UserSchema);
```

基本上我們建立的這個 mongoose 資料模型是讓我們簡化 CRUD 的操作，參考[文件](http://mongoosejs.com/docs/models.html)中的範例程式碼就能明白為什麼我們需要這麼做。

# 設定 MongoDB

如果您本機還沒安裝 MongoDB 請先遵循[官方文件](https://docs.mongodb.com/manual/installation/)進行安裝。又或者可以使用雲端服務 [MongoLab](https://mongolab.com/welcome/)。

在我們安裝完成 MongoDB 並啟動之後我們就可以使用 `mongodb://localhost:27017` 來連線。如果您是使用雲端服務您應該會取得一個連線字串像是 `mongodb://<dbuser>:<dbpassword>@<service.url.com>:27017/<dbName>`。這個連線字串會決定我們操作的資料庫。

通常在專案中我們會將資料庫設定的部分獨立出來。在這裡我們建立一個 `db.js` 來負責資料庫相關的部分。

```js
module.exports = {
  connection: 'mongodb://<dbuser>:<dbpassword>@<service.url.com>:27017/<dbName>'
}
```

如果您跟我一樣是使用本機的 MongoDB ，請記得啟動您的 MongoDB 並且設定如下

> 資料庫名稱即 `<dbName>` 的部分可以自行修改。

```
module.exports = {
  connection: 'mongodb://localhost:27017/express-mongodb-passport-development'
}
```

現在，我們已經完成前置作業，讓我們在 `app.js` 載人並使用 mongoose 

```js
var mongoose = require('mongoose');
var config = require('./db');
mongoose.connect(config.connection);

// 在需要操作 db 的地方記得載入 Model
```

# 設定 passport

passport 本身只提供驗證機制，其他過程中如果需要功能像是 session 則是交給其他 Middleware。這邊我們則使用 `express-session` 來處理。

```bash
$ npm i express-session -S
```

安裝完之後我們開始在 `app.js` 中設定 passport

```js
var passport = require('passport');
var LocalStrategy = require('passport-local').Strategy
var session = require('express-session');

app.use(session({ 
  secret: 'your secret key',
  resave: false,
  saveUninitialized: false
}));
app.use(passport.initialize())
app.use(passport.session())
```

加上 `session` 的設定是因為我們希望用戶登入後的相關資料可以透過 session 被保留且一致，所以我們才安裝了 `express-session` 來幫我們處理這部分。
`LocalStrategy` 並不強制使用 session，我們也可以透過參數來關閉，但如果沒有使用 session 每一次`登入後資料不會自動保留`需要自己處理。

> 概念：passport 可以 `use` 多個策略，接著只要在對應的路由使用 `passport.authenticate('<strategy-name>')` 即可。

# 序列化與反序列化 User 物件

序列化指的是一個轉換資料格式的過程。為了支援登入的 session，passport 需要把從 session 取得的資料轉換成帳號資料即 User 的物件實例。
因此後續的 request 請求將不會重複包含使用者的認證資訊。為了完成這個功能，passport 提供了兩個方法 `serializeUser` 和 `deserializeUser`。

```js
passport.serializeUser(function (user, done) {
  done(null, user._id);
});

passport.deserializeUser(function (id, done) {
  User.findById(id, function (err, user) {
    done(err, user);
  });
});
```

看完上面的程式碼，您應該理解了；passport 只把 `user._id` 存在 session，取回時在反序列化從資料庫找回 User 物件。

# 使用 passport 驗證機制

### 登入機制

現在，我們終於要為登入和註冊行為定義 passport 的驗證機制。在`這個範例`中每個`驗證策略`或者說驗證邏輯比較通順，它們都是一個 passport 的 `Local Authentication Strategy` 物件，並可以使用 `passport.use()` 來套用。這裡我們要在加上一個函式庫 `connect-flash` 來協助我們處理呈現`動作結果的資訊`。

```js
var User = require('./models/user.js);

passport.use('login', new LocalStrategy({
    passReqToCallback: true
  },
  function (req, username, password, done) {
    User.findOne({ username: username }, function (err, user) {
      if (err) {
        return done(err)
      }

      if (!user) {
        return done(null, false, req.flash('info', 'User not found.'))
      }

      if (!isValidPassword(user, password)) {
        return done(null, false, req.flash('info', 'Invalid password'))
      }

      return done(null, user)
    })
  }
));
```

`passport.use()` 的第一個參數是驗證機制的名稱，稍後這可以用來識別到底是哪一套機制。第二個參數是我們想建立的`策略`類型，這邊我們使用 `LocalStrategy`。值得注意的是預設 LocalStrategy 會在 Middleware 傳遞的過程中查找 `req.body` 和 `req.query` 的 `username` 和 `password` 來取得用戶認證的資訊，但我們也可以使用其他參數名稱。

其中 `passReqToCallback` 參數設定讓我們可以在後續 callback 中存取 `req` 物件。如果沒有的話，那麼預設 callback 參數會如下：

> 注意 `*` 註解

```js
new LocalStrategy({
  // 使用其他參數名稱取得用戶認證的資訊
  usernameField: 'email',
  passwordField: 'passwd',
  // 關閉 session
  session: false
},function(username, password, done) {
  //  * 沒有 passReqToCallback 的話，第一個參數就不是 req
})
```

> 詳細 passport-local 的參數與用法請參考[說明](https://github.com/jaredhanson/passport-local)

取得用戶帳密之後就是驗證的邏輯我們會在 callback 參數中處理這件事。下一步，我們將使用 mongoose 取得帳號資訊來驗證帳號是否正確。
而在 LocalStrategy 的 callback 中最後一個參數 `done` 就是我們用來告訴 passport 模組登入結果的方法，假如登入失敗那麼第一個參數需要傳入錯誤訊息，第二個參數則是 `false`。登入成功的話，第一個參數則傳入 `null`，第二個則是任何為 `真` 的值，這個值會被加到 `req` 物件上以便我們後續使用。

接著，基於存放在 MongoDB 用戶資料的安全性考量，我們應該在儲存之前先將密碼加密。要完成這個需求我們需要使用加解密的函式庫。本文使用 [bcrypt-nodejs](https://www.npmjs.com/package/bcrypt-nodejs)，如果您有習慣的函式庫也可以使用。

```bash
$ npm i bcrypt-nodejs -S
```

```js
var isValidPassword = function (user, password) {
  return bcrypt.compareSync(password, user.password)
}
```
讓我們先來看看目前完成的 `app.js`

```js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var mongoose = require('mongoose');
var passport = require('passport');
var LocalStrategy = require('passport-local').Strategy;
var session = require('express-session');
var flash = require('connect-flash');
var bcrypt = require('bcrypt-nodejs');


// DB
var config = require('./db');
mongoose.connect(config.connection);

passport.serializeUser(function (user, done) {
  done(null, user._id);
});

passport.deserializeUser(function (id, done) {
  User.findById(id, function (err, user) {
    done(err, user);
  });
});

passport.use('login', new LocalStrategy({
    passReqToCallback: true
  },
  function (req, username, password, done) {
    User.findOne({ username: username }, function (err, user) {
      if (err) {
        return done(err)
      }

      if (!user) {
        return done(null, false, req.flash('info', 'User not found.'))
      }

      var isValidPassword = function (user, password) {
        return bcrypt.compareSync(password, user.password)
      }

      if (!isValidPassword(user, password)) {
        return done(null, false, req.flash('info', 'Invalid password'))
      }

      return done(null, user)
    })
  }
));

var index = require('./routes/index');

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use(session({
  secret: 'your secret key',
  resave: false,
  saveUninitialized: false
}));
app.use(passport.initialize());
app.use(passport.session());
app.use(flash());

app.use('/', index);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;
```

# 註冊機制

我們定義好登入部分的機制了，接著我們要定義下一個機制來處理新的使用者註冊帳號的部分

```js

passport.use('signup', new LocalStrategy({
  passReqToCallback: true
}, function (req, username, password, done) {
  var findOrCreateUser = function () {
    User.findOne({ username: username }, function (err, user) {
      if (err) {
        return done(err);
      }

      if (user) {
        return done(null, false, req.flash('info', 'User already exists'));
      } else {
        var newUser = new User();
        newUser.username = username;
        newUser.password = bcrypt.hashSync(password, bcrypt.genSaltSync(10), null);
        newUser.email = req.params.email;
        newUser.firstname = req.params.firstname;
        newUser.lastname = req.params.lastname;

        newUser.save(function (err, user) {
          if (err) {
            throw err;
          }

          return done(null, user);
        });
      }
    });
  };

  process.nextTick(findOrCreateUser)
}));
```

再一次我們使用 mongoose 來查詢看看是否有使用者已經註冊了一樣的 username 。如果沒有那麼我們就建立新的帳號，否則就返回一個錯誤訊息。

# 建立路由

讓我們先來看看這個程式的流程圖

![](https://cms-assets.tutsplus.com/uploads/users/388/posts/21619/image/Bird_Eye_View_of_PassportJS_App.png)

完成登入和註冊的邏輯定義之後，我們要來撰寫路由。預設 express generator 將路由抽離成一個個模組，我們已經刪除了 `routes/users.js`。為了讓 `routes/idnex.js` 裡的路由能夠使用剛剛上面定義在 `app.js` 的 passport 物件，我們將改寫一下 `routes/index.js` 如下

```js
var express = require('express');
var router = express.Router();

module.exports = function (passport) {
  router.get('/', function (req, res, next) {
    res.render('index', { message: req.flash('info') });
  })

  router.post('/signin', passport.authenticate('login', {
    successRedirect: '/home',
    failureRedirect: '/',
    failureFlash: true
  }));

  router.get('/signup', function (req, res, next) {
    res.render('signup', { message: req.flash('info') });
  });

  router.post('/signup', passport.authenticate('signup', {
    successRedirect: '/home',
    failureRedirect: '/signup',
    failureFlash: true
  }));

  return router;
}
```

上面程式碼中最重要的部分就是：當 HTTP 發出 POST 的請求給 `login` 和 `signup` 時分別使用 `passport.authenticate()` 搭配對應的機制(login，signup)來處理。

注意到路由並不需要強制跟驗證機制模組的名稱一樣。

假如您還需要在驗證過程中加入其他處理，則可使用下面這種寫法

```js
router.post('/signin', function (req, res, next) {
  passport.authenticate('login', function (err, user, info) {
    if (err) {
      // handle youself
    }

    if (!user) {
      // handle youself
    }

    req.login(user, function (err) {
      if (err) {
        return next(err)
      }
      req.flash('info', 'Sign in successfully')
      return res.redirect('/')
    })

  })(req, res, next)
})
```

# 樣板

接著我們需要調整 views 的部分，我們總共需要註冊 `signup`，登入 `index`，後續需要登入驗證的頁面 `home`，以及調整我們的 `layout`。

* `layout.jade` 
* `index.jade` 登入頁
* `signup.jade` 註冊頁
* `home.jade` 

### `layout.jade`

```jade
doctype html
html
  head
    title= title
    link(rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css" integrity="sha384-rwoIResjU2yc3z8GV/NPeZWAv56rSmLldC3R/AZzGRnGxQQKnKkoFVhFQhNUwEyJ" crossorigin="anonymous")
    script(src="https://code.jquery.com/jquery-3.1.1.slim.min.js" integrity="sha384-A7FZj7v+d/sdmMqp/nOQwliLvUsJfDHW+k9Omg/a/EheAdgtzNs3hpfag6Ed950n" crossorigin="anonymous")
    script(src="https://cdnjs.cloudflare.com/ajax/libs/tether/1.4.0/js/tether.min.js" integrity="sha384-DztdAPBWPRXSA/3eYEEUWrWCy7G5KFbe8fFjk5JAIxUYHKkDx6Qin1DkWx51bBrb" crossorigin="anonymous")
    script(src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/js/bootstrap.min.js" integrity="sha384-vBWWzlZJ8ea9aCX4pEW3rVHjgjt7zpkNpZk+02D9phzyeVkE+jo0ieGizqPLForn" crossorigin="anonymous")
    link(rel='stylesheet', href='/stylesheets/style.css')
  body
    block content
```

### `index.jade`

```jade
extends layout

block content
  div(class="container")
    div(class="col-md-4 offset-md-4")
      h1(class="text-center login-title") Sign In
      div(class="card")
        div(class="card-blcok")
          form(action="/signin" method="post")
            input(type="text" name="username" class="form-control" placeholder="Email" required="true" autofocus)
            input(type="password" name="password" class="form-control" placeholder="Password" required="true")
            button(class="btn btn-primary btn-block" type="submit") Sign In
            span(class="clearfix")
            if message.length
              div(id="message")
                  div(class="alert alert-danger text-center") #{message}
```

### `signup.jade`

```jade
extends layout

block content
  div(class="container")
    div(class="col-md-4 offset-md-4")
      h1(class="text-center login-title") Sign Up
      div(class="card")
        div(class="card-blcok")
          form(action="/signup" method="post")
            input(type="text" name="username" class="form-control" placeholder="Email" required="true" autofocus)
            input(type="password" name="password" class="form-control" placeholder="Password" required="true")
            button(class="btn btn-primary btn-block" type="submit") Sign In
            span(class="clearfix")
            if message.length
              div(id="message")
                  div(class="alert alert-danger text-center") #{message} 
```

### `home.jade`

```jade
extends layout

block content
  div(class="container")
    div(class="row")
      div(class="col-12")
        p(class="alert alert-primary") This is page will protected by passport.
```

> 基於商標因素 jade 樣板引擎已[改名為 pug](https://github.com/pugjs/pug/issues/2184)，雖然目前仍然支援但官方已警告 `deprecated` 建議更換為 pug。

# 登出

passport 作爲一個 Middleware 它是具備加入屬性和方法到 `req` 和 `res` 物件的能力的。為了方便使用，它加了 `req.logout()` 這個方法讓我們可以將使用者存在 session 的資料作廢。

```js
router.get('/signout', function () {
  req.logout()
  res.redirect('/')
})
```

# 驗證存取權限

passport 也賦予我們控制存取權限的能力，用來保護不能匿名存取的路由。從這個例子來說就是；當使用者沒有登入的情況下是無法存取 `http://localhost:3000/home` 我們會將他們導向登入或其他公開的頁面。

```js
function authenticated (req, res, next) {
  if (req.isAuthenticated()) {
    return next()
  }
  res.redirect('/)
}

router.get('/home', authenticated, function (req, res, next) {
  res.render('home', { user: req.user })
})
```

# 結論

歸納來說，passport 這個 Middleware 會在整個 request 的過程中幫我們處理`驗證流程`，透過使用 `策略` 決定是否登入成功。在本例子中後續 Middleware 的 `req` 物件中我們就可以取得 `req.isAuthenticated()` 和 `req.user`。由於驗證是否登入可能會有很多路由需要使用，故上面我們將這部分抽成一個 Middleware function 方便後續使用。

另外關於 Node.js 世界的驗證套件 ， passport 並不是這個領域唯一的函式庫，還有像是 `everyauth` 等的函式庫。不過在社群支援度，模組化等部分的確相對優秀，這也是為什麼我們選擇 passport。

關於詳細的比較您可以參考[everyauth vs passport](http://stackoverflow.com/questions/11974947/everyauth-vs-passport-js)。

# 參考資源

* [Authenticating Node.js Applications With Passport](https://code.tutsplus.com/tutorials/authenticating-nodejs-applications-with-passport--cms-21619)
* [Node.js 身份認證：Passport 入門](https://nodejust.com/nodejs-passport-auth-tutorial/)
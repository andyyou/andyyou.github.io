---
title: 使用 Sancutm 實作 React SPA 認證
date: 2020-07-10 15:08:29
categories: Program
tags:
  - react
  - laravel
---

[Sanctum](https://laravel.com/docs/7.x/sanctum) 是一套輕量化的 API 認證機制套件。本文會介紹如何使用 Sanctum 支援 React 單頁應用程式認證（會員登入）。我們假設應用程式的前端和後端使用相同 TLD 頂級網域下的子網域，如此才可以使用 Sanctum 基於 Cookie 的驗證流程。利用這種作法可以省去處理 API Token 。為此我們使用 Homestead 設定兩個網域 `api.sanctum.test` 指向 Laravel 專案的 `public` 目錄用於提供後端，`sanctum.test` 指向另一個目錄提供前端的部分。

<!-- more -->

## 後端

讓我們從 API 開始

```bash
$ laravel new api
```

我們假設這個 API 提供書籍查詢資料，因此我們建立一個 `Book` 的資料模型

```bash
$ cd api
$ php artisan make:model Book -mr
```

`-m` 協助產生 Migration 檔案，`-r` 協助我們產生一個 Restful 風格的 Controller 檔案 - 即包含了 CRUD 等 action。雖然這裡只會用到 `index` 但藉此知道這些參數也是非常實用的。接著在 Migration 增加欄位。

```php
Schema::create('books', function (Blueprint $table) {
  $table->id();
  $table->string('title');
  $table->string('author');
  $table->timestamps();
});
```

執行 `migrate` 之前記得檢查 `.env` 確認資料庫建立和相關連線設定是否正確。這裡我使用 pgsql 您可以選擇您熟悉的資料庫

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=api-demo
DB_USERNAME=homestead
DB_PASSWORD=secret
```

注意 `homestead` 和 `secret` 是 Homestead 預設的資料庫使用者和密碼。

執行

```php
$ createdb api-demo
$ php artisan migrate
```

更新 `DatabaseSeeder.php` 加入一些 Book 的資料。（DatabaseSeeder 利用陣列的方式建立資料時， Model 可以不加 `$fillable`，但如果您的 API 涉及到寫入的話記得補上）

```php
use App\User;
use App\Book;
use Faker\Factory;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     *
     * @return void
     */
    public function run()
    {
        // $this->call(UserSeeder::class);
        Book::truncate();
        $faker = Factory::create();
        for ($i = 0; $i < 50; $i++) {
            Book::create([
                'title' => $faker->sentence,
                'author' => $faker->name,
            ]);
        }
        User::truncate();
        User::create([
            'name' => 'Andy',
            'email' => 'andy@example.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

執行 `php artisan db:seed` 建立模擬資料。最後我們還需要加入路由到 `routes/api.php` 

```php
Route::get('/books', 'BookController@index');
```

然後回到 `BookController.php` 的 `index` 加入

```php
return response()->json(Book::all());
```

在正式專案中我們可能會使用 Laravel 的 API Resource 功能來轉換資料，但對於這個範例來說目前的作法已經堪用。

### Homestead 補充

文章的開頭提到會搭配 Homestead 分別設定兩個子網域。這部分的設定您可以參考[官方 Homestead 文件](https://laravel.com/docs/7.x/homestead) 。這裡補充可能遇到的問題處理方式。

#### 外連本機資料庫

如果您使用本機實體機器下的資料庫（非 Homestead 虛擬機中的資料庫）您可能在連線到 `http://api.sanctum.test/api/books` 的時候遭遇如下錯誤訊息。

```
SQLSTATE[08006] [7] fe_sendauth: no password supplied (SQL: select * from "books") 
```

這是因為您連線的 `127.0.0.1` 是虛擬機本身而不是實體機的資料庫。如果是這樣您可以在執行 `migrate` 和 `db:seed` 之後將連線 `DB_HOST` 調整為 `10.0.2.2`。

但是！如果後續有其他 Migration 需要執行都需要先切換回來。為了單純起見建議您還是登入到 Vagrant 使用虛擬機的資料庫

#### Homestead 資料庫

如果您想要使用 Homestead 自帶的資料庫（以 PostgreSQL 為例）我們須進入虛擬機建立資料庫等

```bash
# 備註如果您調整了 Homestead.yaml 記得重載設定
$ vagrant reload --provision

# 建立資料庫
$ vagrant ssh
$ psql -U homestead -h localhost
$ create database [DATABASE_NAME];

# 進入專案目錄
$ php artisan migrate
$ php artisan db:seed

# 補充 - 建立使用者
$ createuser --interactive --pwprompt

$ psql
# 列出使用者
SELECT * FROM "pg_user";
# 列出所有角色 + 權限
$ \du
```

### （補充）前端 Nginx

因後續我們會使用 `react-router-dom` 。每次都從首頁進入才開始操作是沒問題的，但如果直接使用連結遇到 404 錯誤，此時可以直接使用 Homestead 提供的 `spa` 類型在 `Homestead.yaml` 加入 `type`

```yaml
- map: sanctum.test
  to: /home/vagrant/spa-demo/build
  type: "spa"
```

```bash
# 備註如果您調整了 Homestead.yaml 記得重載設定
$ vagrant reload --provision
```

或者自行調整 Nginx 設定

```bash
$ vagrant ssh
$ sudo vi /etc/nginx/sites-available/sanctum.test
$ sudo nginx -t
$ sudo nginx -s reload
```

```
location / {
    try_files $uri /index.html =404;
}
```



## 前端

前端 SPA 的部分我們會使用 `create-react-app` 來建立。在另外目錄下執行下面指令

```bash
$ npx create-react-app sanctum-spa-demo
$ cd sanctum-spa-demo
```

接著我們安裝 `react-router-dom` 和 `axios` 套件

```bash
$ npm i axios react-router-dom
$ npm start
```

現在我們可以建立 `Book` 元件並使用 `axios` 讀取書籍資料

```js
// src/components/Books.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const Books = () => {
  const [books, setBooks] = useState([]);
  useEffect(() => {
    const cb = async () => {
      const response = await axios.get('https://api.sanctum.test/api/books');
      console.log(response);
      if (response.status === 200) {
        setBooks(response.data);
      }
    };
    cb();
  }, []);

  return (
    <ul>
      {books.map(book => (
        <li key={book.id}>{book.title}</li>
      ))}
    </ul>
  );
};

export default Books;
```

接著在 `src/App.js` 中使用 `Books` 元件

```js
// src/App.js
import React from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  NavLink,
} from 'react-router-dom';
import Books from './components/Books';

function App() {
  return (
    <Router>
      <nav>
        <NavLink to="/">Home</NavLink>
        {' '}
        <NavLink to="/books">Books</NavLink>
      </nav>
      <Switch>
        <Route path='/books' component={Books} />
      </Switch>
    </Router>
  );
}

export default App;
```

執行 `npm start` 造訪 `/books` 頁面可以看到書籍列表。

現在我們希望只能特定授權的人可以查閱資料。或者 API 可以根據不同的使用者顯示不同的書籍。是 Sanctum 登場的時候到了。

讓我們回到**後端專案**安裝相關套件。

```bash
$ composer require laravel/sanctum laravel/ui
```

安裝 `laravel/ui` 是因為它提供了一些會員認證功能的檔案（包含 Controller 和 View）。`vendor:publish` 則是為了建立 `sanctum-config` 

```bash
$ php artisan ui:auth
Authentication scaffolding generated successfully.
$ php artisan vendor:publish
Which provider or tag's files would you like to publish?:
  [0 ] Publish files from all providers and tags listed below
  [1 ] Provider: Facade\Ignition\IgnitionServiceProvider
  [2 ] Provider: Fideloper\Proxy\TrustedProxyServiceProvider
  [3 ] Provider: Fruitcake\Cors\CorsServiceProvider
  [4 ] Provider: Illuminate\Foundation\Providers\FoundationServiceProvider
  [5 ] Provider: Illuminate\Mail\MailServiceProvider
  [6 ] Provider: Illuminate\Notifications\NotificationServiceProvider
  [7 ] Provider: Illuminate\Pagination\PaginationServiceProvider
  [8 ] Provider: Laravel\Sanctum\SanctumServiceProvider
  [9 ] Provider: Laravel\Tinker\TinkerServiceProvider
  [10] Tag: cors
  [11] Tag: flare-config
  [12] Tag: ignition-config
  [13] Tag: laravel-errors
  [14] Tag: laravel-mail
  [15] Tag: laravel-notifications
  [16] Tag: laravel-pagination
  [17] Tag: sanctum-config
  [18] Tag: sanctum-migrations
 > 17
 
Copied File [/vendor/laravel/sanctum/config/sanctum.php] To [/config/sanctum.php]
Publishing complete.
```

然後設定路由

```php
Route::middleware('auth:sanctum')->get('/books', 'BookController@index');
```

由於我們想要前端專案對應 `sanctum.test` 網址，然後它會跟後端 `api.sanctum.test` 溝通取得資料。因此在我們完成 前端變更之後我們可以執行 `npm run build`，然後 Homestead 對應到 `build` 目錄。

可以僅使用一台全域的 Homestead 然後設定對應不同的網域。提供我的 `Homestead.yaml` 作為參考

```yaml
ip: "192.168.10.10"
memory: 2048
cpus: 2
provider: virtualbox
ssl: true

authorize: ~/.ssh/id_rsa.pub

keys:
    - ~/.ssh/id_rsa
    
folders:
    - map: ~/workspace/demo/spa
      to: /home/vagrant/spa-demo
    - map: ~/workspace/demo/api
      to: /home/vagrant/api-demo

sites:
    - map: sanctum.test
      to: /home/vagrant/spa-demo/build
      type: "spa"
    - map: api.sanctum.test
      to: /home/vagrant/api-demo/public
```

另外 `/etc/hosts` 記得設定 IP 對應

```
192.168.10.10     sanctum.test
192.168.10.10     api.sanctum.test
```

此時試著瀏覽 `https://sanctum.test/books` 頁面，您應該會在瀏覽器的開發者工具看到 401 Unauthenticated 錯誤。前端需要登入元件。

```js
// src/components/Login.js
import React, {
  useState,
} from 'react';
import axios from 'axios';

const Login = ({

}) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();

    const cb = async () => {
      const response = await axios.post('https://api.sanctum.test/login', {
        email,
        password,
      });
      console.log('submit', response);
    };
    cb();
  };

  return (
    <div>
      <h3>Login</h3>
      <form onSubmit={handleSubmit}>
        <input
          type="email"
          name="email"
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
        <input
          type="password"
          name="password"
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
        />
        <button type="submit">
          Login
        </button>
      </form>
    </div>
  );
};

export default Login;
```

上面只是基本的表單使用 `axios` 發送請求到後端然後輸出回應。接著我們在 `App.js` 使用該元件

```js
// src/App.js
import React from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  NavLink,
} from 'react-router-dom';
import Books from './components/Books';
import Login from './components/Login';

function App() {
  return (
    <Router>
      <nav>
        <NavLink to="/">Home</NavLink>
        {' '}
        <NavLink to="/login">Login</NavLink>
        {' '}
        <NavLink to="/books">Books</NavLink>
      </nav>
      <Switch>
        <Route path='/login' component={Login} />
        <Route path='/books' component={Books} />
      </Switch>
    </Router>
  );
}

export default App;
```

一樣 `npm run build` 之後瀏覽登入頁面輸入我們在 `seed` 加入的的帳密。這次應該遭遇到的則是 Corss-Origin Request Blocked 錯誤。

## （補充）npm-watch

每次要測試的時候都忘記 `npm run build` ？這裡補充一個簡單的方式支援 `watch` Production Mode。記得先到前端專案下

```bash
$ npm i npm-watch
```

```json
// package.json
"scripts": {
  ...
  "watch": "npm-watch build",
  ...
},
"watch": {
  "build": {
    "patterns": [
      "."
    ],
    "ignore": "build",
    "extensions": "*",
    "quiet": false
  }
},
```

後續可使用 `npm run watch` 一旦檔案變更就會執行 `npm run build`

## 關於 CORS

同源政策是瀏覽器的安全機制，可防止一來源的腳本程式碼（來源指的是例如 http, https, ftp 等 scheme 搭配 hostname 和 port 組成）直接存取其他來源的資料。

CORS（Cross-Origin Resource Sharing）是處理在同源政策保護下讀取其他來源資料的解決辦法。請求的表頭會包含 Origin 資料，而伺服器回應的表頭須包含 `Access-Control-Allow-Origin` 如果兩者符合瀏覽器便會允許接收該回應。

OK！知道了問題但如果您查看瀏覽器開發者工具的 Network 您會發現我們甚至連 POST 請求都沒有送出。實際上，只有 OPTIONS 請求。為什麼？原因是我們的請求不符合簡單請求的規則，即包含了 `Content-Type` 是 `application/json`

> [CORS 詳解](https://www.ruanyifeng.com/blog/2016/04/cors.html)

因此會發送一個 OPTIONS 預先請求到伺服器確認，接著伺服器會回應一些 Header 讓瀏覽器檢查是否可以發出請求。由於 Laravel 還沒設定 CORS 因此沒有回應任何 `Access-Control-` Header，因此請求並沒有發生。前端的部分，瀏覽器會自動處理所以只要調整後端的部分。

事實上，Laravel 7 內建 CORS Middleware，可以通過 `config/cors.php` 設定。開啟該檔案會看到預設 `allowed_origins` 為 `*` 意思是任何請求都是被允許的。那為什麼剛剛的請求無法運作。如果我們在往上一點查看設定檔會看到 `paths` 意思是下面的設定只會套用到 `api` 命名空間下的路由。

在 `paths` 加入 `login` 即可解決這個問題。

## CSRF

然後又錯誤，這次是 419  CSRF Token mismatch 錯誤。CSRF（Cross-Site Request Forgery） 是攻擊者在經過認證的環境下執行惡意操作的一種方式。

舉個在 OWASP 文件中提到的例子 - 當你已登入網銀時，攻擊者會透過一些社交手段欺騙您造訪某個連結。
惡意連結可能是隱藏在 Email 中的一張 0x0 的圖片或是吸引人點擊的連結等。
無論是哪種方式，該網址會嘗試存取網銀的 API，並對您的帳戶造成影響。恐怖的是，由於您已經登入了，因此不需要再進行任何身份驗證流程。

該如何解決這個問題？這裡介紹其中一種方式就是伺服器會先傳送隨機的 Token 到客戶端的 Cookie 然後 Token 包含在請求的 Header 中。如果我們執行 `php artisan route:list` 會看到 `login` 路由屬於 `web` Middleware 群組其中包含了`VerifyCsrfToken` Middleware，在其 `handle` 函式我們看到其邏輯如下

```php
public function handle($request, Closure $next)
{
    if (
        $this->isReading($request) ||
        $this->runningUnitTests() ||
        $this->inExceptArray($request) ||
        $this->tokensMatch($request)
    ) {
        return tap($next($request), function ($response) use ($request) {
            if ($this->shouldAddXsrfTokenCookie()) {
                $this->addCookieToResponse($request, $response);
            }
        });
    }

    throw new TokenMismatchException('CSRF token mismatch.');
}
```

如果條件不符合就會產生 `TokenMismatchException` 例外。由於現在這個請求不屬於單純讀取，我們使用的是 POST 請求，不是單元測試，沒有設定例外，只剩 `tokenMatch` 但我們沒有設定 Token 所以比對一定失敗而產生例外。

OK 這是為我們提供的保護機制。但我們要如何取得 CSRF Token 呢？如果我們停留在伺服器產生的頁面，Laravel 會自動協助我們取得 Token 從 Controller（`csrf_token()`） 到 View (`@csrf`) 都很方便。但現在我們不是使用框架提供的 View 我們需要自行取得 CSRF Token。內建 `api:auth` 沒有提供此功能，此時就是 Sanctum 登場的時候了。Sanctum 允許我們詢問 CSRF Token 然後我們將 Token 含在表頭中。

如果您執行下面查詢路由列表的指令

```bash
$ php artisan route:list
```

您應該會看到一道 `/sanctum/csrf-cookie`。這功能是 `SanctumServiceProvider` 在 `boot` 方法中定義的。

知道有這道 API 只會讓我們回到 `Login` 元件調整

```js
const handleSubmit = (e) => {
  e.preventDefault();

  const cb = async () => {
    await axios.get('https://api.sanctum.test/sanctum/csrf-cookie');
    const response = await axios.post('https://api.sanctum.test/login', {
      email,
      password,
    });
    console.log('submit', response);
  };
  cb();
};
```

> Homestead 下的前端專案更新之後請記得 `npm run build` 或使用 `npm-watch`

記得一樣要設定 `cors` 設定檔

```php
'paths' => ['api/*', 'login', 'sanctum/csrf-cookie'],
```

在我們點擊送出按鈕之前請先開啟瀏覽器的開發者工具，到 Firefox 的儲存空間（Storage）或 Chrome 的 Application。您應該可以查閱 Cookie。此時您應該看到 Cookie 為空。點擊送出還是沒有 Cookie...！ 為什麼？

查看瀏覽器開發工具的 Network 我們的確呼叫了 `sanctum/csrf-cookie` 並取得 204 的回應

{% asset_img 1.png %}

的確有 `laravel_session` 和 `XSRF-TOKEN` 但儲存空間 Cookie 那邊卻沒有。

答案是和 [Cookie 存取限制有關](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Scope_of_cookies)，也是文章一開始我們提到為什麼要使用一樣頂級網域的原因。從伺服器來的 `Set-Cookie` 可以設定 `domain` 指定對哪些網域發送請求的時候要包含該 Cookie。我們看一下 Header 中 `XSRF-TOKEN` 的部分

```
XSRF-TOKEN=<token>; expires=Wed, 08-Jul-2020 04:13:20 GMT; Max-Age=7200; path=/; samesite=lax
```

沒有包含 [Domain 屬性](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)。也就是我們 `sanctum.test` 的請求並不會自動包含該 Cookie。因此我們需要在後端的 `.env` 加入

```
SESSION_DOMAIN=sanctum.test
```

再次執行 Cookie 還是沒有送出。這次是因為前端的問題。 `XMLHttpRequest` 從不同網域取得的回應不能直接設定 Cookie - 即就算收到 `Set-Cookie` Header 瀏覽器也不會執行，除非 `withCredentials`設為 `true`。

因為後續 `axios` 都需要帶此參數，將這個部分重構後續會比較方便。我們建立一個 `src/services` 目錄加入 `api.js`

```js
// src/services/api.js
import axios from 'axios';

const client = axios.create({
  baseURL: 'https://api.sanctum.test',
  withCredentials: true,
});

export default client;
```

然後更新 `Books` 和 `Login` 有使用 API 的部分。

```js
import api from '../services/api';
```

把 `axios` 換成 `api` 。

這次儲存空間或 Application 總算出現 Cookie 了，但回到開發工具的主控台（Console）又看到另一個 `Cross-Origin Request Blocked` 錯誤。原因是基於安全性考量瀏覽器只會對那些表明支援 `withCredentials` 的伺服器發送請求。所以我們要回到後端 `cors.php` 設定 `supports_credentials` 為 `true`。

這次我們的登入包含了正確的憑證，您應該可以看到 `/login` 的請求得到 204 回應。但如果我們切換到 `/books` 仍然得到 401 未驗證的錯誤。

修正這個錯誤的方式就是使用 Sanctum 的 Stateful Domain。開啟 `app/Http/Kernel.php` 加入 `EnsureFrontendRequestsAreStateful` Middleware 到 `api` 群組。

```php
'api' => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    'throttle:60,1',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

接著我們來看看[這個 Class](https://github.com/laravel/sanctum/blob/v2.4.2/src/Http/Middleware/EnsureFrontendRequestsAreStateful.php)原始碼做了些什麼。

```php
config([
    'session.http_only' => true,
    'session.same_site' => 'lax',
]);
```

首先，它覆寫了 Session 設定 `http_only` 為 `true` ，這表示客戶端的 Script 不能存取 Token，詳細 [HttpOnly](https://owasp.org/www-community/HttpOnly) 介紹可以參考 OWASP 網站說明。同時也設定了 `same_site` 為 `lax` 。這是用來防止 Cookie 在其他跨站請求的時候被送出，除非請求是從其他站到您的網站。

```php
return (new Pipeline(app()))
    ->send($request)
    ->through(static::fromFrontend($request) ? [
        // Middleware
    ] : [])
    ->then(function ($request) use ($next) {
        return $next($request);
    });
```

Laravel 的 Middleware 是透過 Pipeline 來處理任務的。該 Class 讓我們可以傳入一個任務處理的陣列並依序帶入資料。

如果您查看 `vendor/laravel/framework/src/illuminate/Foundation/Http/Kernel.php` 的 `sendRequestThroughRouter` 方法，您會看到類似的程式碼

```php
return (new Pipeline($this->app))
    ->send($request)
    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
    ->then($this->dispatchToRouter());
```

因此 Sanctum 的 `EnsureFrontendRequestsAreStateful` Middleware 做的其實就是加入了更多處理的 Middleware 。不過只有請求來自前端的時候 - 這也是為什麼會有：

```php
static::fromFrontend($request) ? [ 
    // some middleware 
] : []
```

如果請求來自前端則會加入這些 Sanctum 專屬的 Middleware 否則傳入空陣列。`static::fromFrontend` 會查看 Header 的 `Referer` ，如果包含在 Sanctum 設定的字串，即可判斷為來自前端的請求。要設定 `Referer` 可以在 `.env` 通過 `SANCTUM_STATEFUL_DOMAINS` 變數設定

```
SANCTUM_STATEFUL_DOMAINS=sanctum.test
```

那哪些是 Sanctum 的 Middleware 呢？

```php
[
  config('sanctum.middleware.encrypt_cookies',
  	\Illuminate\Cookie\Middleware\EncryptCookies::class),
  
  \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
  \Illuminate\Session\Middleware\StartSession::class,
  
  config('sanctum.middleware.verify_csrf_token',
  	\Illuminate\Foundation\Http\Middleware\VerifyCsrfToken::class),
]
```

這 4 個 Middleware 內建就有了您可以在 `web` 的 Middleware 看到它們的身影。

* EncryptCookies：Cookie 加密，即使有心人士存取了 Cookie 並修改了內容，到伺服器的時候會檢查並拒絕執行。
* AddQueuedCookiesToResponse：處理 Cookie Facade 加入佇列的 Cookie。
* StartSession：設定與 Laravel 連線階段相關資訊和 Session Cookie，其資訊會被加入回應。
* VerifyCsrfToken：檢查 CSRF Token


## （補充）Postman 取得 CSRF Token

因為 `axios` 會自動幫我們補上 `XSRF-TOKEN` Header ，所以如果想在 Postman 發送請求的話可以 在 Pre-request Script 加入：

```
const Header = require('postman-collection').Header;
const cookieJar = pm.cookies.jar();
const host = pm.request.url.getHost();
const endpoint = host + '/sanctum/csrf-cookie'

pm.sendRequest(endpoint, (err, response) => {
    cookieJar.get(host, 'XSRF-TOKEN', (err, xsrfToken) => {
        pm.request.headers.append(Header.create(xsrfToken, 'X-XSRF-TOKEN'));
    })
});
```


## 登入認證

加入 Middleware 可以處理 Cookie 流程的部分。但現在會執行認證機制是因為路由加了 `auth:sanctum`，意思是使用 [Sanctum Guard class](https://github.com/laravel/sanctum/blob/v2.4.2/src/Guard.php) 來處理驗證。但如果我們查看 Sanctum Guard class 會發現有點奇怪，[官方加入自訂 Guard 文件](https://laravel.com/docs/7.x/authentication#adding-custom-guards)提到自訂的 Guard 需要繼承 `Illuminate\Contracts\Auth\Guard` 介面，但原始碼完全沒有 `implements` 關鍵字而只有 `__invoke` 這個神奇的方法。

查看 [SanctumServiceProvider](https://github.com/laravel/sanctum/blob/v2.4.2/src/SanctumServiceProvider.php) ，發現是使用文件建議的 `$auth->extend` 方法：

```php
$auth->extend('sanctum', function ($app, $name, array $config) use ($auth) {
    return tap($this->createGuard($auth, $config), function ($guard) {
        $this->app->refresh('request', $guard, 'setRequest');
    });
});
```

`tap` 簡單說就是建立第一個參數然後帶入第二個閉包，最後會回傳 `$guard`，接著我們看到 `createGuard` 的部分

```php
return new RequestGuard(
    new Guard($auth, config('sanctum.expiration'), $config['provider']),
    $this->app['request'],
    $auth->createUserProvider()
);
```

首先回傳的是`RequestGuard`物件實例，有實作 `Guard` 滿足 `extend` 方法的參數型別。`RequestGuard` 的第一個參數是閉包，在我們的例子中就是 Sanctum 的 `Guard` Class，其中的差異就是傳入的是一個帶有 `__invoke` 的 Class，您可以把它想成是包含狀態的閉包。接著使用 `RequestGuard` 回傳一個 `user` 。下面是 [Sanctum Gurad](https://github.com/laravel/sanctum/blob/v2.4.2/src/Guard.php#L54-L58) 相關程式碼：

```php
if ($user = $this->auth->guard(config('sanctum.guard', 'web'))->user()) {
    return $this->supportsTokens($user)
                ? $user->withAccessToken(new TransientToken)
                : $user;
}
```

第一行會利用 `web` Guard 取得 `$user` 因為我們使用一般的 `web` 登入的，如果 `$user` 被找到則回傳。

撇開上面深入的探討，如果您加入 `SANCTUM_STATEFUL_DOMAINS` 設定，那麼應該可以登入並使用 `/api/books` API 讀取資料。

## SPA

現在我們已經完成後端的驗證機制了，該輪到前端的部分。後續文章跟 Sanctum 沒有直接關係，如果您對於前端的部分沒興趣可以直接跳過。

>  另外注意的是因為 `web` 路由有使用 `RedirectIfAuthenticated` Middleware，執行登入前請先記得把 Cookie 清除否則就會因為登入過了而跳轉到 `/home` 而在開發者工具看到錯誤訊息。

第一步是 `App` 元件需要一個判斷是否登入的狀態：

```js
const [isLoggedIn, setIsLoggedIn] = useState(false);
```

然後加入一個處理函式

```js
const handleLogin = () => {
  setIsLoggedIn(true);
};
```

接著把函式傳入 `Login` 元件（還有狀態的部分，如果已登入就不用顯示表單了或者您想要實作跳轉都可以）

```jsx
<Route
  path='/login'
  render={(props) => (
    <Login {...props} onLogin={handleLogin} isLoggedIn={isLoggedIn} />
  )}
/>
```

到 `Login` 元件在 `handleSubmit` 使用我們傳入的函式。

```js
import React, {
  useState,
} from 'react';
import api from '../services/api';

const Login = ({
  isLoggedIn,
  onLogin,
}) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();

    const cb = async () => {
      await api.get('sanctum/csrf-cookie');
      const response = await api.post('/login', {
        email,
        password,
      });
      if (response.status === 204) {
        onLogin();
      }
    };
    cb();
  };

  return (
    <div>
      <h3>Login</h3>
      {isLoggedIn ? (
        <div>You are logged in</div>
      ) : (
        <form onSubmit={handleSubmit}>
          <input
            type="email"
            name="email"
            placeholder="Email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
          />
          <input
            type="password"
            name="password"
            placeholder="Password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
          />
          <button type="submit">
            Login
          </button>
        </form>
      )}
    </div>
  );
};

export default Login;
```

現在 `App` 元件已經知道使用者是否登入的狀態，我們可以將狀態也傳入 `Books` 元件這樣就可以依據登入與未登入的狀態來執行對應的操作

```jsx
<Route
  path='/books'
  render={(props) => (
    <Books {...props} isLoggedIn={isLoggedIn} />
  )}
/>
```

如果 `isLoggedIn` 為 `false` 的話就不用嘗試讀取 API 載入資料，而是提示使用者應該先登入。

調整 `Books` 元件

```jsx
import React, { useState, useEffect } from 'react';
import api from '../services/api';

const Books = ({
  isLoggedIn,
}) => {
  const [books, setBooks] = useState([]);
  useEffect(() => {
    const cb = async () => {
      const response = await api.get('/api/books');
      if (response.status === 200) {
        setBooks(response.data);
      }
    };

    if (isLoggedIn) {
      cb();
    }
  }, []);

  return (
    <>
      {isLoggedIn ? (
        <ul>
          {books.map(book => (
            <li key={book.id}>{book.title}</li>
          ))}
        </ul>
      ) : (
        <div>
          Please login to read books.
        </div>
      )}
    </>
  );
};

export default Books;
```

那登出呢？回到 `App` 元件調整，如果已經登入的話顯示登出連結，未登入的話則顯示登入連結。如下完整 `App` 程式碼主要注意 `handleLogout` 和 `authLink` 的部分

```jsx
import React, { useState } from 'react';
import {
  BrowserRouter as Router,
  Switch,
  Route,
  NavLink,
} from 'react-router-dom';
import Books from './components/Books';
import Login from './components/Login';
import api from './services/api';

function App() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const handleLogin = () => {
    setIsLoggedIn(true);
  };
  
  const handleLogout = (e) => {
    e.preventDefault();
    const cb = async () => {
      const response = await api.post('/logout');
      if (response.status === 204) {
        setIsLoggedIn(false);
      }
    };
    cb();
  };

  const authLink = isLoggedIn ? (
    <a href="" onClick={handleLogout}>Logout</a> 
  ) : (
    <NavLink to="/login">Login</NavLink>
  );
  return (
    <Router>
      <nav>
        <NavLink to="/">Home</NavLink>
        {' '}
        {authLink}
        {' '}
        <NavLink to="/books">Books</NavLink>
      </nav>
      <Switch>
        <Route
          path='/login'
          render={(props) => (
            <Login {...props} onLogin={handleLogin} />
          )}
        />
        <Route
          path='/books'
          render={(props) => (
            <Books {...props} isLoggedIn={isLoggedIn} />
          )}
        />
      </Switch>
    </Router>
  );
}

export default App;
```

這個時候直接執行一樣會遇到 CORS 的問題記得在 `cors.php` 把 `logout` 加入 `paths`。

```
'paths' => ['api/*', 'login', 'sanctum/csrf-cookie', 'logout'],
```

再一次登出，您應該可以看到選單發生變化。

最後我們需要將 `isLoggedIn` 的狀態存在瀏覽器上，如果不這麼做當使用者重新整理頁面 SPA 的狀態就會消失。我們可以利用 `sessionStorage` API 來處理這個問題

```js
const [isLoggedIn, setIsLoggedIn] = useState(
  sessionStorage.getItem('isLoggedIn') === 'true' || false
);
const handleLogin = () => {
  setIsLoggedIn(true);
  sessionStorage.setItem('isLoggedIn', true);
};
const handleLogout = (e) => {
  e.preventDefault();
  const cb = async () => {
    const response = await api.post('/logout');
    if (response.status === 204) {
      setIsLoggedIn(false);
      sessionStorage.setItem('isLoggedIn', false);
    }
  };
  cb();
};
```

## 資源參考

* [原文](https://laravel-news.com/using-sanctum-to-authenticate-a-react-spa)
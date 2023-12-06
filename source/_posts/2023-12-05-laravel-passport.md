---
title: 'Laravel Passport 和 OAuth2'
date: 2023-12-05 15:21:57
tags:
  - laravel
categories: Program
---

Laravel Passport 為您的 Laravel 應用程式提供了完整的 OAuth2 伺服器實作。Passport 是基於 League OAuth2 Server 套件建置。

> 本文假設您已經熟悉 OAuth2。如果您還不知道何謂 OAuth2，在繼續之前，請熟悉 OAuth2 的一般術語和功能。
>
> [參考資源](https://oauth2.thephpleague.com/terminology/)

<!-- more -->

## 何謂 OAuth2

讓我們先從基本的驗證功能開始：

1. 用戶於客戶端輸入帳號密碼
2. 客戶端送給伺服器驗證身份
3. 驗證成功後通過 Session/Cookie 機制在 **客戶端** 儲存登入的狀態

基本的驗證機制儘管適用於很多情境，但因為

- 缺乏擴展彈性（多台伺服器的 Session / Cookie 機制須另外處理）
- 安全性的問題（密碼被盜沒有太多應對辦法）
- 維護成本（同組織的各項服務可能需要反覆實作一樣的功能）
- 用戶體驗（社交媒體的興起，簡化註冊流程，多設備訪問）
- 授權第三方應用

種種因素尤其是授權第三方應用，因此 OAuth2 廣泛的受到採用。

OAuth2 是一個業界標準協議，處理有關授權的相關流程。

例如某網站服務可以使用 Facebook 登入；Facebook 授權這個某網站有限範圍內取得用戶在 Facebook 的相關資訊或可操作的行為。

用戶驗證是在 Facebook 完成，登入且用戶同意授權，第三方應用程式被授權取得存取憑證（Access Token），可以使用該存取憑證從 Facebook 取得需要的資源。

總結來說就是你授權第三方可以去 Facebook 取得你的資料。

1. 在第三方應用點擊 "使用 Facebook 登入"
2. 跳轉至 Facebook 輸入帳密登入
3. 確認授權第三方範圍（Scope）即第三方要求的權限
4. 同意的話，瀏覽器導回第三方，拒絕的話停止在此步驟
5. 第三方取得授權可以向 Facebook 存取資料或執行你授權的操作

### 認識 OAuth2 的術語

- `Access Token` （存取權杖）: 這是一種特殊的權杖，用於讓應用程式能夠安全存取用戶在平台上保護的資料，如社群平台的個人資料。
- `Authorization Code` （授權碼）: 這是一種中介權杖。當用戶同意授權後，系統會先提供這個權杖給第三方應用程式，目的是提高交換存取權杖的安全性。這是因為直接回傳存取權杖可能存在安全風險，這個步驟增加了一層保護。
- `Authorization Server` （授權伺服器）: 這是管理用戶身份驗證和授權的伺服器。它負責處理授權請求，確認身份，發行存取權杖和授權碼給客戶端（Client）。
- `Client` （客戶端）: 指的是想要取得你資料的第三方應用程式，例如某個需要取得用戶 Facebook 資料的應用程式。
- `Grant` （授權）: 這是用戶授予第三方應用程式存取其資料的過程，如 Facebook 登入之後點擊同意按鈕的操作。
- `Resource Server` （資源伺服器）: 這是儲存用戶資料的伺服器，如社群媒體帖文和個人資料。它通過驗證存取權杖來檢查是否提供資料存取。
- `Resource Owner` （資源擁有者）: 指的是用戶本人，他們可以授權第三方應用程式存取他們在目標平台的帳戶。這種存取是有限制的，僅限於用戶授予的權限範圍內。
- `Scope` （授權範圍）: 這定義了第三方應用程式可以存取的資料類型和程度，例如僅讀取個人資料權限。
- `JWT` （JSON Web Token）: 這是一種格式為 JSON 的安全權杖，常用於用戶和伺服器之間的身份驗證和資訊傳輸。它依靠伺服器端的密鑰和簽名過程來保障安全，而客戶端則負責安全地儲存和使用這個權杖。
- `Redirect URI（Callback URL）`：授權伺服器驗證完身份，授權同意之後導向的網址或路徑。

## OAuth2 支援 4 種授權模式

### 授權碼模式 Authorization Code

1. 用戶發起登入：用戶在應用點擊**使用 Google 登入**
2. 瀏覽器導向授權伺服器：過程中會先提供 Google 需要的資料如 Client Id, Redirect URI, Scope 等
3. 用戶身份驗證：在授權伺服器上進行用戶身份驗證
4. 授權確認：用戶被提示是否同意授予應用程式所需權限
5. 授權碼返回：用戶同意後，瀏覽器導向回應用程式的 Redirect URL 網址，同時包含授權碼（Authorization Code）
6. 交換存取權杖：應用程式使用授權碼向授權伺服器請求存取權杖（Access Token）
7. 存取資源：應用程式使用存取權杖向資源伺服器請求資料

```
Client ID: 客戶端 ID
Client Secret: 客戶端密鑰
Redirect URI: 導向網址
Scope: 授權範圍(可選)
State: 防止 CSRF 攻擊隨機產生的值(推薦)
```

### Authorization Code + PKCE（Proof Key for Code Exchange）

1. 產生程式碼驗證器和程式碼挑戰：

- 應用程式產生一個隨機的程式碼驗證器（Code Verifier）即一個隨機字串。
- 應用程式使用程式碼驗證器產生一個程式碼挑戰（Code Challenge），通常是對驗證器進行 SHA256 雜湊 Hash 並進行 Base64 編碼。

2. 用戶發起登入
3. 發送授權請求：應用程式將用戶導向授權伺服器，並在請求中包含程式碼挑戰和其他必要資訊，如客戶端 ID、導向 URI 等。
4. 用戶身份驗證和授權
5. 返回授權碼
6. 使用授權碼和程式碼驗證器取得存取權杖
7. 存取資源

主要在步驟 1 中加入產生程式碼驗證器和程式碼挑戰的過程，並在步驟 6 中使用這些值來獲取存取權杖。PKCE 通過確保只有發送授權請求的同一客戶端能夠使用授權碼來提高了安全性。

```
Client ID: 客戶端 ID
Redirect URI: 導向網址
Scope: 授權範圍(可選)
Code Verifier: 隨機字串，用於產生 Code Challenge
Code Challenge: 用 Code Verifier 產生，發送給授權伺服器
State: : 防止 CSRF 攻擊隨機產生的值(推薦)
```

### 隱式模式 Implicit

適用於客戶端應用程式（例如 JavaScript SPA）。通常整個程式都在前端運行。因為需向 API 取得資料，也因為沒有後端無法通過授權碼交換存取權杖。因而直接讓授權伺服器核發存取權杖（Access Token）。

1. 用戶發起登入：用戶在應用程式中點擊登入。
2. 導向到授權伺服器：瀏覽器導向授權伺服器。
3. 用戶身份驗證：用戶在授權伺服器上進行身份驗證。
4. 授權確認：用戶被提示是否同意授予權限。
5. 直接獲取存取權杖：用戶同意後，瀏覽器重定向回應用程式同時包含回傳存取權杖。
6. 存取資源：應用程式使用存取憑證向資源伺服器請求資料。

```
Client ID: 客戶端 ID
Redirect URI: 導向網址
Scope: 授權範圍(可選)
State: : 防止 CSRF 攻擊隨機產生的值(推薦)
```

### 資源擁有者密碼憑證模式 Resource Owner Password Credentials

用戶將帳號密碼交給第三方應用程式，由應用程式直接向授權伺服器交換存取權杖。

1. 收集用戶憑證：用戶將自己的登入資訊（例如，用戶名和密碼）提供給應用程式
2. 應用程式請求存取權杖：應用程式直接使用這些資訊向授權伺服器請求存取權杖
3. 存取資源：獲得存取權杖後，應用程式即可使用此權杖向資源伺服器請求用戶資料。

```
Client ID: 客戶端 ID（可選，取決於伺服器端的要求）
Client Secret: 客戶端密鑰（可選，取決於伺服器端的要求）
Username: 用戶帳號。
Password: 用戶密碼。
```

### 客戶端憑證模式 Client Credentials

通常是應用程式向授權伺服器取得存取權杖，讀取自己的資料而不是用戶的資料。

1. 應用程式認證：應用程式使用其自身的憑證（不涉及用戶）向授權伺服器請求存取權杖
2. 存取資源：應用程式使用獲得的存取權杖向資源伺服器請求資料，通常是應用程式自己的資料而非用戶個人資料

```
Client ID: 客戶端 ID
Client Secret: 客戶端密鑰
```

## Passport 或 Sanctum?

在開始之前您可能需要確認您的應用程式需要使用 Passport 還是 Sanctum。如果您的應用程式需要支援 OAuth2 那麼你該使用 Passport 。如果你只是需要支援存取第三方平台的資料如 Facebook 登入或者簡單提供 API Token 讓應用程式可以使用 API，那麼你應該使用 Sanctum。

Scanctum 不支援 OAuth2 但是提供簡單的 API 驗證開發體驗。

## 安裝 Passport

通過 Composer 安裝

```sh
$ compoer require laravel/passport
```

Passport 的 Service Provider 會提供相關的 migration 檔案，在安裝之後需要執行 `php artisan migrate`，主要會建立 OAuth2 需要的相關資料表。

```sh
$ php artisan migrate
```

接著，您需要執行 `php artisan passport:install` 指令**建立加密金鑰(產生 Access Token 時使用)**。此外，這個指令還會建立**個人存取**和**密碼存取**的 Client，因應不同的資料結構與流程產生 Access Token。

⚠️ 注意：在執行之前，如果您的 `Client` 資料模型希望使用 UUID 作為主鍵，執行 `passport:install` 安裝指令時請使用 `uuids` 參數。

```sh
$ php artisan passport:install

$ php artisan passport:install --uuids
```

執行 `passport:install` 之後，在 `App\Models\User` 加入 `Laravel\Passport\HasApiTokens` trait。這個 trait 會提供資料模型一些輔助函式，讓你可以檢查已登入用戶的 Token 和授權範圍 Scope。如果你的資料模型已經使用了 `Laravel\Sanctum\HasApiTokens` trait，請移除。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

最後，在 `config/auth.php` 設定檔，設定 `api` 驗證機制和 `driver` 設定成 `passport`。這個步驟會讓應用程式使用 Passport 的 `TokenGuard` 來驗證。

```php
'guards' => [
  'web' => [
    'driver' => 'session',
    'provider' => 'users'
  ],

  'api' => [
    'driver' => 'passport',
    'provider' => 'users'
  ]
]
```

您可能已經執行 `passport:install --uuids` 。這個參數會讓 Passport 使用 UUID 作為主鍵而不是自動遞增的數字。選擇使用 UUID 作為主鍵時，需要進行一些額外的設定步驟，包括修改或禁用預設的數據庫遷移設定，需要重新執行 Migration。

> 指令會提供提示訊息，另外如果 User 等也要使用 UUID 須調整 Migrations。[參考教學修改 Migrations](https://laraveldaily.com/post/laravel-users-table-change-primary-key-id-to-uuid)
>
> ⚠️ `--uuids` 參數會解開 Migration 檔案，即使你沒有 `php artisan vendor:publish --tag=passport-migrations`

### 部署 Passport

第一次部署 Passport 的應用程式到伺服器時，你可能需要執行 `php artisan passport:keys` 指令來產生加密所需的金鑰，這把金鑰用於產生 Access Token，檔案預設在 `storage/` 目錄下。

```sh
$ php artisan passport:keys
```

你可以設定 Passport 載入金鑰的路徑。你可以使用 `Passport::loadKeysFrom` 方法。一般來說這個方法在 `App\Providers\AuthServiceProvider` 類別的 `boot` 方法中調用。

```php
public function boot(): void
{
  Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
}
```

又或者，你可以使用 `php artisan vendor:publish` 指令解開 Passport 的設定檔

```sh
$ php artisan vendor:publish --tag=passport-config
```

匯出設定檔到 `config/passport.php` 使用環境變數來設定加密金鑰。

```yaml
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

### 自訂 Migration

如果你不想使用 Passport 預設的 Migration，你可以在 `App\Providers\AppServiceProvider` 的 `register` 方法中使用 `Passport::ignoreMigrations` 方法

```sh
$ php artisan vendor:publish --tag=passport-migrations
```

## 相關設定

### Client Secret 雜湊

如果你希望儲存在資料庫的 Client Secret 也進行加密，如同密碼進行不可逆的 hash ，你可以在 `App\Providers\AuthServiceProvider` 的 `boot` 加入 `Passport::hashClientSecrets` 方法。

```php
use Laravel\Passport\Passport;

public function boot()
{
  Passport::hashClientSecrets();
}
```

一旦啟用，所有的 Client Secret 將只會在用戶建立時顯示一次，因為原始資料沒有儲存在資料庫且加密不可逆，因此一旦遺失該 Secret 無法恢復和查詢。

### 憑證有效時間

預設 Passport 發行的存取憑證 Access Token 屬於長效期，一年後才會到期。如果你希望調整更長或短一點的效期，你可以使用 `tokensExpireIn`，`refreshTokenExpireIn` ，`personalAccessTokensExpireIn` 方法。

上面這些方法在 `App\Providers\AuthServiceProvider` 的 `boot` 方法中呼叫。

> Access Token：用戶授權後用於取得資源的金鑰
>
> Refresh Token：Access Token 過期後 app 不用重新登入取得新的 Access Token 使用
>
> Person Access Token：開發階段或特定情境直接取得的 Access Token 無需標準 OAuth 授權流程的金鑰。

```php
public function boot(): void
{
  Passport::tokenExpireIn(now()->addDays(15));
  Passport::refreshTokensExpireIn(now()->addDays(30));
  Passport::personalAccessTokensExpireIn(now()->addMonths(6));
}
```

**Passport 資料表的 `expires_at` 是唯讀的只用於顯示使用。發行憑證的時候 Passport 會把過期資訊儲存在簽署和加密的憑證中，也就是不會依賴資料表的資料，如果你需要讓憑證失效需要註銷。**

### 覆寫預設資料模型

你可以繼承 Passport 內部的資料模型進而自訂模型。

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
  // ...
}
```

在定義自己的模型之後，可以通過 `Laravel\Passport\Passport` 類別設定 Passport 使用自訂的模型。

一般來說你會在 `App\Providers\AuthServiceProvider` 的 `boot` 設定

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\PersonalAccessClient;
use App\Models\Passport\RefreshToken;
use App\Models\Passport\Token;

public function boot(): void
{
  Passport::useTokenModel(Token::class);
  Passport::useRefreshTokenModel(RefreshToken::class);
  Passport::useAuthCodeModel(AuthCode::class);
  Passport::useClientModel(Client::class);
  Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
}
```

### 覆寫路由

自訂路由的部分。為了達成這個目的，首先需要在 `AppServiceProvider` 使用 `Passport::ignoreRoutes` 略過 Passport 預設註冊的路由

```php
use Laravel\Passport\Passport;

public function register(): void
{
  Passport::ignoreRoutes();
}
```

然後，可以複製 [Passport 預設的路由](https://github.com/laravel/passport/blob/11.x/routes/web.php) 到 `routes/web.php` 並修改調整。

```php
Route::group([
    'as' => 'passport.',
    'prefix' => config('passport.path', 'oauth'),
    'namespace' => '\Laravel\Passport\Http\Controllers',
], function () {
    // Passport routes...
});
```

## 發行存取憑證 Access Token

通過授權碼 Authorization Code 的方式使用 OAuth2 是多數開發者熟悉的流程。授權碼模式下，第三方應用先將用戶導向我們的授權伺服器讓用戶進行授權或拒絕請求發行存取憑證。

### 管理 Client

首先，開發者建立的第三方應用程式在和我們 Passport 的 API 溝通之前需要註冊一個 "Client" 。一般來說就是提供應用程式的名稱，和同意授權後導向回去的 URL（Redirect URL）。

#### `passport:client` 指令

最簡單建立 Client 的方式就是使用 `php artisan passport:client` 指令。這個指令可以協助建立測試 OAuth2 的 Client。當我們執行這個指令時 Passport 會提示詢問我們更多資訊然後建立 Client 並提供 Client ID 和 Secret。

```sh
$ php artisan passport:client
```

### 導向 URLs

若你需要允許多個導向網址可以使用 `,` 逗號分隔。這個設定主要是起到檢查的作用。第三方應用發送授權請求的時候會提供一個 `redirect_uri` 然後伺服器會檢查是否和該 Client 設定的多個路由的其中一個是否匹配（`redirect` 參數）。檢查符合設定的 URLs 才會進行導向。

### JSON API

由於第三方應用程式的用戶無法直接使用 `client` 指令，因此 Passport 提供了 JSON API 用於建立 Client。省去我們自行建立 Controller 開發新增，更新，刪除 Client 的時間。

不過我們還是需要自己實作前端通過這些 JSON API 提供用戶管理他們的 Client 的介面。下面我們將過一遍這些 API。為了方便後續會使用 Axios 來展示發送請求。

JSON API 預設使用 `web` 和 `auth` middleware 驗證；因此我們只能在我們的應用程式上呼叫，外部是不能直接 call 這些 API 的。

#### GET /oauth/clients

這個路由會回傳通過驗證的用戶全部 Client 列表。讓用戶可以編輯或刪除 Client。

```js
axios.get('/oauth/clients').then((response) => {
  console.log(response.data);
});
```

#### POST /oauth/clients

此路由用來新增 Client 。它需要兩個參數 `name` 和 `redirect` 。當 Client 建立後會發行 Client ID 和 Secret 。這兩個資料在第三方應用程式用於後續請求 Access Token

```js
const data = {
  name: 'Client Name',
  redirect: 'https://example.com/auth/provider/callback', // 遵循 Socialite 慣例
};

axios
  .post('/oauth/clients', data)
  .then((response) => {
    console.log(response.data);
  })
  .catch((response) => {
    // List errors
  });
```

#### PUT /oauth/clients/{client-id}

此路由用於更新 Client。需要提供兩個參數 `name` 和 `redirect` 。

```js
const data = {
  name: 'New Client Name',
  redirect: 'http://example.com/auth/provider/callback',
};

axios
  .put(`/oauth/clients/${clientId}`, data)
  .then((response) => {
    console.log(response.data);
  })
  .catch((response) => {
    // List errors
  });
```

#### DELETE /oauth/clients/{client-id}

刪除 Client：

```js
axios.delete(`/oauth/clients/${clientId}`).then((response) => {
  // ...
});
```

## 請求存取憑證 Access Token

### 導向驗證

一旦 Client 建立完成，開發者就可以使用 Client ID 和 Secret 跟我們的服務請求 Authorization Code 和 Access Token。首先，這個第三方應用要導向我們的 `/oauth/authorize` 範例如下：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
  $request->session()->put('state', $state = Str::random(40));

  $query = http_build_query([
    'client_id' => 'client-id',
    'redirect_uri' => 'https://example.com/auth/provider/callback',
    'response_type' => 'code',
    'scope' => '',
    'state' => $state,
    // 'prompt' => '', // "none", "consent", "login"
  ]);

  return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

`prompt` 參數可以用來設定 Passport 驗證的行為。

- `none`：當 `prompt` 為 `none` 時如果用戶沒有登入使用 Passport 的應用程式則拋出錯誤
- `consent`：即使之前已經被授權了，Passport 還是會宣示授權確認頁面
- `login`：即使用戶已經登入了 Passport 還是會提示用戶重新登入
- 沒設定：如果用戶之前還沒授權則提示用戶進行授權

### 同意請求授權

當收到驗證請求，Passport 會自動根據 `prompt` 的設定回應並顯示同意或拒絕的頁面。如果同意請求，那麼就會導向一開始請求參數中的 `redirect_uri` 。這個 `redirect_uri` 必須符合 Client 設定的 `redirect`。

如果你希望自訂驗證同意的畫面，你可以使用 `php artisan vendor:publish` 解開 Passport 的視圖，檔案會在 `resources/views/vendor/passport`目錄

```sh
$ php artisan vendor:publish --tag=passport-views
```

有時候你希望跳過驗證提示，例如你提供的另一個 app 而非其他人的應用程式。你可能不想要用戶每次都看到確認授權的頁面，直接發放 Access Token。此時你可以擴展 `Client` 資料模型使用 `skipsAuthorization`。假如 `skipsAuthorization` 回傳 `true` 那麼 Client 會直接同意授權並導向 `redirect_uri` ，除非提出請求的應用設定 `prompt`。

```php
<?php

namespace App\Models\Passport;

use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
  public function skipsAuthorization(): bool
  {
    return true;
  }
}
```

### 使用 Authorization Code 交換 Access Token

若用戶同意授權請求，將會導向提出請求的應用程式。該 app 首先須驗證 `state` 參數和發出請求前設定的是否一致然後發送一個 `POST` 請求到我們的應用程式來取得 Access Token。這個請求須包含 Authorization Code

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
  $state = $request->session()->pull('state');

  throw_unless(
  	strlen($state) > 0 && $state === $request->state,
    InvalidArgumentException::class,
    'Invalid state value.'
  );

  $response = Http::asForm()->post('http://passport-app.text/oauth/token', [
    'grant_type' => 'authorization_code',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'redirect_uri' => 'http://third-party-app.com/callback',
    'code' => $request->code,
  ]);

  return $response->json();
});
```

`/oauth/token` 路由會回傳一個 JSON 物件包含 `access_token`，`refresh_token` 和 `expires_in`。`expires_in` 屬性表示 Access Token 的過期秒數。

### JSON API

Passport 也提供 JSON API 管理存取憑證。我們一樣需要在前端使用對應的 API 提供用戶管理存取憑證。一樣為了方便下面使用 Axios 展示發送請求，這些 API 一樣也是使用 `web` 和 `auth` Middleware 驗證，因此只能在自己的應用呼叫，不支援外部請求。

#### GET /oauth/tokens

這個路由回傳全部驗證的 Access Token。主要的用途就是列出 Token 讓用戶可以撤銷。

```js
axios.get('/oauth/tokens').then((response) => {
  console.log(response.data);
});
```

#### DELETE /oauth/tokens/{token-id}

此路由可用來撤銷通過驗證的 Access Token 和其相關的 Refresh Token

```js
axios.delete(`/oauth/tokens/${tokenId}`);
```

### Refresh Token

若你發行的 Access Token 有效時間很短，用戶端會需要使用 Refresh Token 更新 Access Token

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
  'grant_type' => 'refresh_token',
  'refresh_token' => 'the-refresh-token',
  'client_id' => 'client-id',
  'client_secret' => 'client-secret',
  'scope' => '',
]);

return $response->json();
```

`/oauth/token` 路由會回傳 JSON 物件包含 `access_token` `refresh_token` 和 `expires_in`

### 撤銷金鑰

你可以使用`Laravel\Passport\TokenRepository` 的 `revokeAccessToken` 方法撤銷 Token。`revokeRefreshTokensByAccessTokenId` 可以撤銷 Refresh Token。

```php
use Laravel\Passport\TokenRepository;
use Laravel\Passport\RefreshTokenRepository;

$tokenRepository = app(TokenRepository::class);
$refreshTokenRepository = app(RefreshTokenRepository::class);

$tokenRepository->revokeAccessToken($tokenId);
$refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);
```

### 清除 Tokens

當 Token 被撤銷或過期，你可能希望從資料庫清除它們。Passport 提供了 `php artisan passport:purge` 指令

```sh
$ php artisan passport:purge

# 僅清除過期超過6小時的 token
$ php artisan passport:purge --hours=6

# 僅清除撤銷的 Token 和 Authorization code
$ php artisan passport:purge --revoked

# 僅清除過期的 Token 和 Authorization code
$ php artisan passport:purge --expired
```

你也可以在 `App\Console\Kernel` 設定排程自動清除。

```php
protected function schedule(Schedule $schedule): void
{
  $schedule->command('passport:purge')->hourly();
}
```

## PKCE 與 Authorization Code 授權

> PKCE （Proof Key for Code Exchange）是一種 OAuth 授權流程額外增強安全性的技術。
>
> 1. 應用生成一個隨機字串 - Code Verifier
> 2. 應用計算該字串的 SHA256 + Base64 encode 作為 Code Challenge 發給授權伺服器
> 3. 用戶授權，伺服器將授權碼傳回應用
> 4. 應用使用授權碼和 Code Verifier 請求 Access Token
> 5. 伺服器檢查 Code Verifier 轉換後和 Code Challenge 是否匹配，然後發放 Access Token

Authorization Code 授權搭配 PKCE 流程，PKCE 主要對 SPA 或行動 app 強化驗證機制和存取 API 安全性。這種授權應被用於當您無法保證客戶端密鑰能被保密儲存，或為了減輕授權碼被攻擊者攔截的風險時，應該使用這種授權方式。在使用 Authorization Code 交換 Access Token 時，Code Verifier 和 Code Challenge 的組合將取代 Client Secret 的角色。

### 建立 Client

在你的應用程式可以通過 Authorization Code 授權搭配 PKCE 發行 Token 之前，你需要建立啟用 PKCE 的 Client

```sh
$ php artisan passport:client --public
```

> 主要的差異就是這個 Client 預設不會建立 Secret 因為你將使用 PKCE Code Verifier 和 Code Challenge 的組合將取代 Client Secret

#### Code Verifier & Code Challenge

由於這種授權方式不會提供 Client Secret，開發者需要自己產生 Code Verifier 和 Code Challenge 來取得 Access Token。

Code Verifier 應該是隨機產生長度 43 - 128 包含字母，數字和 `-`, `.` `_` `~` 字元的字串符合 RFC 7636 規範。

Code Challenge 應該為 Base64 encode 的字串並使用合法網址和檔名的字元。結尾時 `=` 字元且應該移除換行等其他字元。

```php
$encoded = base64_encode(hash('sha256', $codeVerifier, true));
$codeChallenge = strtr(ttrim($encoded, '='), '+/', '-_');
```

#### 導向驗證

一旦 Client 建立，就可以使用 Client ID 和 Code Verifier 以及 Code Challenge 去請求 Authorization Code 和 Access Token 。首先第三方應用需要導向 `/oauth/authorize`

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
  $request->session()->put('state', $state = Str::random(40));

  $request->session()->put('codeVerifier', $codeVerifier = Str::random(128));

  $codeChallenge = strtr(rtrim(
  	base64_encode(hash('sha256', $codeVerifier, true))
    , '='
  ), '+/', '-_');

  $query = http_build_query([
    'client_id' => 'client-id',
    'redirect_uri' => 'https://third-party-app.com/callback',
    'response_type' => 'code',
    'scope' => '',
    'code_challenge' => $codeChallenge,
    'code_challenge_method' => 'sha256',
    // 'prompt' => '', // "none", "consent", or "login"
  ]);

  return redirect('http://passport-app.test/oauth/authorize?'.$query);
})
```

#### Authorization Code 交換 Access Token

如果用戶同意授權，將會導回第三方應用。該應用驗證 `state` 是否與導向前匹配。

如果匹配，第三方應用會在 `POST` 請求 Access Token。該請求包含 Authorization Code 以及原來產生的 Code Verifier。

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    $codeVerifier = $request->session()->pull('code_verifier');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'client-id',
        'redirect_uri' => 'http://third-party-app.com/callback',
        'code_verifier' => $codeVerifier,
        'code' => $request->code,
    ]);

    return $response->json();
});
```

## 密碼授權

OAuth2 密碼授權允許你的其他應用程式例如同樣是自己開發的行動 app 使用帳號和密碼取得 Access Token。這個模式讓你的 app 可以快速取得 Access Token 而不需要用戶執行完整的 OAuth2 流程

### 建立密碼授權 Client

在我們的應用程式通過密碼授權發行 Token 之前，我們需要建立一個密碼授權的 Client。使用 `php artisan passport:client --password` 可以建立。預設 `passport:install` 會兩種類型各自建立一組 Client。

```sh
$ php artisan passport:client --password
```

### 請求 Token

一旦建立密碼授權 Client，就可以使用帳號和密碼 `POST /oauth/token` 請求 Access Token。注意這個路由 Passport 已經註冊了不需要重新定義。如果請求成功會自己收到 `access_token` 和 `refresh_token`

```php
use Illuminate\Support\Facades\Http;

$reponse = Http::asForm()->post('http://passport-app.test/oauth/token', [
  'grant_type' => 'password',
  'client_id' => 'client-id',
  'client_secret' => 'client-secret',
  'username' => 'taylor@laravel.com',
  'password' => 'my-password',
  'scope' => '',
]);

return $response->json();
```

### 授權範圍

當使用密碼授權或客戶端憑證授權，你可能希望驗證的 Token 授權全部範圍的資源。你可以使用 `'*'` 星號。如果要求的是 `'*'` 授權範圍（Scope）接著 Token 物件實例的 `can` 方法會永遠回傳 `true`。這個範圍只能使用在 `password` 或 `client_credentials` 授權。

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
  'grant_type' => 'password',
  'client_id' => 'client-id',
  'client_secret' => 'client-secret',
  'username' => 'taylor@laravel.com',
  'password' => 'my-password',
  'scope' => '*',
]);
```

### 自訂 User Provider

如果我們的應用程式使用多個驗證機制（User Provider），你可以在建立 Client 的時候通過 `--provider` 指定密碼授權的 Provider。Provider 的名稱應匹配有效且定義在 `config/auth.php` 設定。

換句話說 User Provider 就是對應登入後那個代表用戶的資料模型，具體實作如下：

1. 在 `config/auth.php` 設定假設名為 `employees`

   ```php
   'providers' => [
     'users' => [
       'driver' => 'eloquent',
       'model' => App\Models\User::class,
     ],

     'employees' => [
       'driver' => 'eloquent',
       'model' => App\Models\Employee::class,
     ],
   ]
   ```

2. 使用 `php artisan passport:client --password --provider=employees`

3. 最後搭配路由 `Route::middleware('auth:employees')`

### 自訂用戶帳號欄位

當使用密碼授權，Passport 會使用 `email` 欄位來作為驗證的帳號。但你可能希望自訂，此時可以使用 `findForPassport`

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
  use HasApiTokens, Notifiable;

  public function findForPassport(string $username): User
  {
    return $this->where('username', $username)->first();
  }
}
```

### 自訂驗證密碼

當使用密碼驗證，Passport 會使用 `password` 欄位驗證。如果你的資料模型沒有 `password` 欄位，或你希望自訂驗證邏輯，可以使用 `validateForPassportPasswordGrant` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
  use HasApiTokens, Notifiable;

  public function validateForPassportPasswordGrant(string $password): bool
  {
    return Hash::check($password, $this->password);
  }
}
```

## 隱式授權

隱式授權類似 Authorization Code 授權；但是 Token 會直接傳回給用戶端而不是 Authorization Code。這種授權通常用在 JavaScript 前端站或行動應用。要啟用授權，在 `App\Providers\AuthServiceProvider` 的 `boot` 調用 `enableImplicitGrant`

```php
public function boot(): void
{
  Passport::enableImplicitGrant();
}
```

一旦授權啟用，開發者可以使用 Client ID 請求 Access Token。第三方應用應該導向我們的 `/oauth/authorize` 路由。

```php
use Illuminate\Http\Request;

Route::get('/redirect', function (Request $request) {
  $request->session()->put('state', $state = Str::random(40));

  $query = http_build_query([
    'client_id' => 'client-id',
    'redirect_uri' => 'http://third-party-app.com/callback',
    'response_type' => 'token',
    'scope' => '',
    'state' => $state,
    // 'prompt' => '', // "none", "consent", "login"
  ]);

  return redirect('http://passport-app.test/oauth/authorize?'.$query);
});
```

### Client Credentials 授權

Client Credentials 授權適合用於機器對機器驗證。例如你可能授權一個排程 Job 利用 API 執行一些維護的任務。

在我們的應用可以通過 Client Credentials 授權發行金鑰之前，你需要建立 Client Credentials 授權的 Client。使用下面指令

```sh
$ php artisan passport:client --client
```

接著，為了使用這種授權類型，加入 `CheckClientCredentials` Middleware 到 `app/Http/Kernel.php` 的 `$middlewareAliases` 屬性

```php
use Laravel\Passport\Http\Middleware\CheckClientCredentials;

protected $middlewareAliases = [
  'client' => CheckClientCredentials::class,
];
```

然後在路由使用

```php
Route::get('/orders', function (Request $request) {
  // ...
})->middleware('client');
```

為了限制可存取的範圍 Scope 我們可以使用 `,` 加入 Middleware 的後面

```php
Route::get('/orders', function (Request $request) {
  ...
})->middleware('client:check-status,your-scope');
```

### 讀取 Token

為了取得這種授權方式的 Token，需要發送請求到 `/oauth/token` 路由

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
  'grant_type' => 'client_credentials',
  'client_id' => 'client-id',
  'client_secret' => 'client-secret',
  'scope' => 'your-scope',
]);

return $response->json()['access_token'];
```

## 個人 Access Token

有時候，我們的用戶希望自己發行自己的 Access Token 而不經過完整的驗證流程。允許用戶通過介面取得自己的 Token 進而讓用戶試驗您的 API，或可以更簡單的取得 Access Token。

> 如果你的應用程式主要使用 Passport 來產生個人 Access Token，也可以考慮使用 Laravel Sanctum，這是輕量化用於產生 API Access Token 的套件

### 建立個人存取的 Client

在我們應用可以發行個人 Access Token 之前，你需要建立個人存取 Client。使用 `php artisan passport:client` 搭配 `--personal` 建立。如果你使用 `passport:install` 你可以不用執行這個指令

```sh
$ php artisan passport:client --personal
```

建立個人存取 Client 之後，將 Client 的 ID 和 Secret 放到 `.env`

```yaml
PASSPORT_PERSONAL_ACCESS_CLIENT_ID="client-id-value"
PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET="unhashed-client-secret-value"
```

### 管理個人 Access Token

一旦你建立了個人存取 Client，就可以使用 `App\Models\User` 的 `createToken` 建立 Token 。`createToken` 方法可以傳入名稱和範圍參數等

```php
use App\Models\User;

$user = User::find(1);

$token = $user->createToken('Token Name')->accessToken;

$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

### JSON API

Passport 也包含一個 JSON API 用於管理個人 Access Token。你可以搭配你的前端提供用戶管理個人 Access Token 的管理介面。下面將概覽一下全部的 API 為了方便將使用 Axios 示範發送 HTTP 請求。

JSON API 有 `web` 和 `auth` Middleware 保護；因此只能在我們的應用呼叫，無法從外部呼叫。

#### GET /oauth/scopes

此路由回傳所有的範圍 `scopes` ，可以顯示範圍列表給用戶並授權給個人 Access Token

```js
axios.get('/oauth/scopes').then((response) => {
  console.log(response.data);
});
```

#### GET /oauth/personal-access-tokens

此路由回傳全部的個人 Access Token ，登入的用戶可以管理。用戶 Access Token 列表在提供用戶編輯和撤銷也非常實用。

```js
axios.get('/oauth/personal-access-tokens').then((response) => {
  console.log(response.data);
});
```

#### PSOT /oauth/personal-access-tokens

此路由可以建立新的用戶 Access Token。需要 2 個參數 `name` 和 `scopes`

```js
const data = {
  name: 'Token Name',
  scopes: [],
};

axios
  .post('/oauth/personal-access-tokens', data)
  .then((response) => {
    console.log(response.data.accessToken);
  })
  .catch((response) => {
    // ...
  });
```

#### DELETE /oauth/personal-access-tokens/{token-id}

此路由可以用於撤銷個人 Access Token

```js
axios.delete('/oauth/personal-access-tokens/' + tokenId);
```

## 保護路由

### 使用 Middleware

Passport 包含一個驗證的 Guard 用於驗證請求的 Access Token 。一旦設定 `api ` Guard 使用 `passport` Driver，我們只需設定 `auth:api` Middleware 就可以驗證 Access Token

```php
Route::get('/user', function() {
  //...
})->middleware('auth:api');
```

如果使用的是 Client 憑證授權則 Middleware 需要使用 `client`。

#### 複數驗證 Guard

如果我們的應用包含驗證多種不同的用戶類型（使用不同的 Model），會需要為每一個 Provider 類型定義 guard 設定。這讓你可以保護特定用戶的請求。例如在 `config/auth.php` 提供

```php
'api' => [
  'driver' => 'passport',
  'provider' => 'users'
],
'api-customers' => [
  'driver' => 'passport',
  'provider' => 'customers'
]
```

下面路由則使用 `api-customers` Guard 搭配 `customers` Provider 來驗證請求

```php
Route::get('/customer', function () {
  // ...
})->middleware('auth:api-customers');
```

#### 傳送 Access Token

當呼叫的路由受到 Passport 保護的時候，API 請求應設定 `Authorization: Bearer TOKEN` Header 。

```php
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
  'Accept' => 'application/json',
  'Authorization' => 'Bearer ' . $accessToekn,
])->get('https://passport-app.test/api/user');

return $response->json();
```

## Token Scopes

範圍 Scope 允許 API 用戶請求驗證存取用戶帳號時指定權限。例如，如果你建立一個購物網站，並非所有調用 API 都需要建立訂單的功能。你可能希望用戶只能請求訂單的出貨狀態。換句話說，scopes 讓用戶可以限制第三方應用可以執行的權限。

### 定義 Scopes

你可以在 `App\Providers\AuthServiceProvider` 的 `boot` 用 `Passport::tokenCan` 方法定義 API scope。`tokenCan` 方法需要一個陣列參數

```php
public function boot(): void
{
  Passport::tokenCan([
    'place-orders' => 'Place orders',
    'check-status' => 'Check order status',
  ])
}
```

### 預設 Scopes

如果用戶端沒有請求特定 Scopes， 你可以用 `setDefaultScope` 設定預設的 Scopes 。一樣是在 `App\Providers\AuthServiceProvider` 類別定義：

```php
use Laravel\Passport\Passport;

Passport::tokenCan([
  'place-orders' => 'Place orders',
  'check-status' => 'Check order status',
]);

Passport::setDefaultScope([
   'check-status',
   'place-orders',
]);
```

預設 Scopes 不會套用在個人 Access Token。

### 設定 Access Token 的範圍

#### 請求 Authorization Code

當使用 Authorization Code 授權模式請求 Access Token 時，用戶端應在 Query String 設定他們希望的 `scope`

`scope` 使用空白字元分隔範圍清單

```php
Route::get('/redirect', function () {
  $query = http_build_query([
    'client_id' => 'client-id',
    'redirect_uri' => 'http://example.com/callback',
    'response_type' => 'code',
    'scope' => 'place-orders check-status',
  ]);

  return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

#### 個人 Access Token

如果你使用 `App\Models\User` 的 `createToken` 產生個人 Access Token ，你需要在第二個參數傳入希望的範圍：

```php
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

### 檢查 Scopes

Passport 包含兩個 Middleware 可以用來檢查請求是否包含授權範圍的 Token。要使用 Middleware 須在 `app/Http/Kernel.php` 的 `$middlewareAliases` 加入

```php
'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
'scope' => \Laravel\passport\Http\Middleware\CheckForAnyScope::class,
```

#### 檢查全部 Scopes

`scopes` Middleware 可以設定到路由用於驗證請求的 Access Token 是否具備全部列出的 scopes

```php
Route::get('/orders', function () {
  // ...
})->middleware(['auth:api', 'scopes:check-status,place-orders']);
```

#### 檢查任意 Scopes

`scope` Middleware 可以設定到路由用於檢查請求的 Access Token 至少符合列表其中一項 Scope

```php
Route::('/orders', function () {
  // Access Token 至少有 check-status 或 place-orders scope
})->middleware(['auth:api', 'scope:check-status,place-orders']);
```

#### 檢查 Token 物件實例的 Scope

一旦 Access Token 通過驗證到應用程式，還可以用 `App\Models\User` 的 `tokenCan` 檢查 Access Token 是否有指定的範圍。

```php
use Illuminate\Http\Request;

Route::get('/orders/', function (Request $request) {
  	if ($request->user()->tokenCan('place-orders')) {

    }
});
```

#### 其他 scope 方法

`scopeIds` 方法會回傳全部定義 scope 的 ID / 名稱陣列

```php
use Laravel\Passport\Passport;

Passport::scopeIds();
```

`scopes` 方法會回傳全部定義的 `Laravel\Passport\Scope` 物件實例陣列。

```php
Passport::scopes();
```

`scopesFor` 方法會回傳匹配指定 IDs / 名稱的 `Laravel\Passport\Scope` 物件實例陣列。

```php
Passport::scopesFor(['place-orders', 'check-status']);
```

`hasScope` 檢查是否有定義該範圍

```php
Passport::hasScope('place-orders');
```

## JavaScript 使用 API

在開發 API 時，如果您的前端應用（比如用 JavaScript 寫的 SPA）需要訪問這個 API，通常的做法是在每次發送請求時附加一個 Access Token。但是，Laravel Passport 提供了一種更簡便的方法。
這種方式讓我們的應用可以使用將對外開放一樣的 API。同樣的 API 可以被我們的網頁應用，行動 app ，第三方應用，任何發佈的 SDK 使用。

一般來說，如果你希望從 JavaScript 應用使用 API，會需要手動發送 Access Token ，不過 Passport 支援一個 Middleware 可以處理這種情況。你只要加入 `CreateFreshApiToken` Middleware 到 `app/Http/Kernel.php` 檔案的 `web` Middleware 群組

```php
'web' => [
  // ...
  \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
]
```

需要確保這個 `CreateFreshApiToken` Middleware 在最後。

這個 Middleware 會將一個名為 `laravel_token` 的 Cookie 附加到 Response。 Cookie 包含一個加密的 JWT ，Passport 會使用這個 JWT 驗證來自你 JavaScript 應用的請求。也就是當讀取頁面時，由於瀏覽器會自動隨所有請求發送這個 Cookie，您的前端應用可以不必在每次請求時明確地傳遞訪問權杖。JWT 的有效時間等於 `session.lifetime` 設定。

```js
axios.get('/api/user').then((response) => {
  console.log(response.data);
});
```

#### 自訂 Cookie 名稱

如果你需要自訂 `laravel_token` 的名稱可以使用 `Passport::cookie` 方法。一般來說，這個方法應該在 `App\Providers\AuthServiceProvider` 的 `boot` 使用。

```php
public function boot(): void
{
  Passport::cookie('custom_name');
}
```

#### CSRF

當使用這種驗證方式時，你需要確保 CSRF 有效。預設 Laravel 會在 Axsio 的請求自動將 `XSRF-TOKEN` Cookie 的值加入 `X-XSRF-TOKEN` Header 裡。

## 事件

當發行 Access Token 或 Refresh Token 時 ，Passport 會引發事件，你可以利用這些事件清除或撤銷其他失效的 Access Token。如果你希望這麼做可以在 `App\Providers\EventServiceProvider` 註冊監聽。

```php
protected $listen = [
  'Laravel\Passport\Events\AccessTokenCreated' => [
    'App\Listeners\RevokeOldTokens',
  ],

  'Laravel\Passport\Events\RefreshTokenCreated' => [
  	'App\Listeners\PruneOldTokens',
  ],
]
```

## 測試

Passport 的 `actingAs` 方法可以用來指定目前通過驗證的用戶和其 Scopes。`actingAs` 第一個參數時用戶的物件實例，第二個參數這是 scopes 的陣列

```php
use App\Models\User;
use Laravel\Passport\Passport;

public function test_servers_can_be_created(): void
{
  Passport::actingAs(
  	User::factory()->create(),
    ['create-servers']
  );

  $response = $this->post('/api/create-server');

  $response->assertStatus(201);
}
```

Passport 的 `actingAsClient` 方法可以用來指定當前通過驗證的 Client 和 Scopes

```php
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function test_orders_can_be_retrieved(): void
{
  Passport::actingAsClient(
  	Client::factory()->create(),
    ['check-status']
  );

  $response = $this->get('/api/orders');

  $response->assertStatus(200);
}
```

## 實戰練習

### 取得 Client Credentials Token & JWK 驗證

**1. 取得 Client Credentials Token**

```sh
$ php artisan passport:client --client
```

**2. 取得 Access Token**

```sh
$ curl -X POST http://localhost:8000/oauth/token \
    -H "Content-Type: application/json" \
    -d '{
        "grant_type": "client_credentials",
        "client_id": "your-client-id",
        "client_secret": "your-client-secret",
        "scope": ""
    }'
```

**3. 取得 JWK**

```sh
$ curl -X GET http://localhost:8000/oauth/jwk
```

**4. 驗證 Access Token**

```js
import jwt from 'jsonwebtoken';
import jwkToPem from 'jwk-to-pem';

const jwtToken = 'jwt-token';

const jwks = {
  keys: [
    {
      kty: 'RSA',
      alg: 'RS256',
      use: 'sig',
      kid: 'oauth-public-key',
      n: '---',
      e: 'AQAB',
    },
  ],
};

const pem = jwkToPem(jwks.keys[0]);
jwt.verify(jwtToken, pem, { algorithms: ['RS256'] }, (err, decoded) => {
  if (err) {
    console.log(err);
  } else {
    console.log(decoded);
  }
});

// decoded
{
  aud: '9ac5c11a-xxx', // oauth_clients.id
  jti: 'c6a8xxx', // oauth_access_tokens.id
  iat: 1701741512.363331,
  nbf: 1701741512.363331,
  exp: 1706925512.356904,
  sub: '',
  scopes: []
}
```

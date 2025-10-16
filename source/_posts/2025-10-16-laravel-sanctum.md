---
title: "Laravel Sanctum 筆記"
date: 2025-10-16 10:07:15
tags:
  - laravel
categories: Program
---

Laravel Sanctum 主要是針對 SPA 、行動應用程式和基於憑證的 API 提供的一個輕量級身份驗證系統。Sanctum 讓每一個應用程式的使用者可以產生多個 API 憑證。這些憑證可以授權功能、範圍執行特定操作。

> 讓我們的 API 提供支援 SPA、行動應用、第三方 API 一個 Token 認證保護機制。

<!-- more -->

## 目標

Laravel Sanctum 的存在是為解決 2 個不同的問題。

- API Token 認證
- SPA 認證

### API Tokens

首先，Sanctum 是一個簡單的套件，我們用於發行 API 憑證給使用者，比起 OAuth 更加簡單。這個功能靈感來自 Github 「個人存取憑證」。想像你的應用程式中的「帳號設定」頁面，使用者可以建立自己帳號的 API 憑證，然後就可以使用這個憑證（Token）呼叫 API 不需要 OAuth 複雜的流程。通過 Sanctum 可以建立和管理憑證。這些憑證具有較長的使用期限，也可以隨時手動撤銷。

> Sanctum 、Passport、Fortify、Starter Kit
>
> Sanctum 簡單說就是 API Token 的管理套件，一般是給「自己的」程式用的 API Token 管理。而 Passport 支援的 OAuth 一般是給第三方服務使用，讓別人的 App 經過用戶同意後可以存取我們的 API。
>
> Fortify 只有「後端邏輯」的認證系統（無 UI）提供：註冊、登入、密碼重置、2FA 等。Starter Kit 為 Laravel 12 之後完整的「前後端」認證實作（有 UI），基本認證不用 Fortify，但 2FA 使用 Fortify。

Laravel Sancum 儲存使用者 API 憑證到資料表，後續比對驗證 HTTP 請求 `Authorization` 標頭中的憑證是否正確來實現這個功能。

其次，Sanctum 有提供一個簡單的方式讓 SPA 可以認證並使用 Laravel 提供的 API。應用程式可以是在同一個 Laravel 專案的前端或者獨立的 Next 或 Nuxt 專案。這個功能，Sanctum 不使用任何類型的憑證，Sanctum 使用 Laravel 預設基於 Cookie 的 Session 驗證機制。基本上 Sanctum 利用 Laravel 的 `web` 驗證保護機制（Guard），支援 CSRF，Session 驗證，防止 XSS 洩漏憑證。

只有在請求來自我們自己的 SPA 時（也就是有包含 CSRF 或符合設定的網域），Sanctum 才會嘗試使用 Cookie 進行身份驗證。否則就是使用 `Authorization` 標頭和 API 憑證驗證。

## 安裝

我們可以使用 `install:api`  Artisan 指令進行安裝：

```sh
$ php artisan install:api
```

## 設定

### 覆寫預設資料模型

雖然一般來說這不是必要的，但是我們可以自由擴展 Sanctum 內部使用的 `PersonalAccessToken` 

```php
use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;
 
class PersonalAccessToken extends SanctumPersonalAccessToken
{
    // ...
}
```

然後我們可以到 `AppServiceProvider` 的 `boot()` 設定，通過 `usePersonalAccessTokenModel` 讓 Sanctum 使用我們自訂的資料模型。

```php
use App\Models\Sanctum\PersonalAccessToken;
use Laravel\Sanctum\Sanctum;

public function boot(): void
{
  	Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
}
```

> 對於自行開發的 SPA 建議不應使用 API 憑證的方式。而是使用內建 [SPA 驗證功能](https://laravel.com/docs/12.x/sanctum#spa-authentication)

## API 憑證

### 發行 API 憑證

Sanctum 讓我們可以發行 API Token / 個人存取 Token 用於驗證 API 請求。當請求使用 API Token 時 Token 需要加入到標頭 `Authorization: Bearer [TOKEN]`。

要幫使用者發行憑證，我們的 `User` 資料模型須先加入 `Laravel\Sanctum\HasApiTokens` trait：

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

接著，可以使用 `createToken` 方法建立憑證。`createToken` 方法回傳 `Laravel\Sanctum\NewAccessToken` 物件實例。API 憑證在儲存之前會使用 SHA-256 加密，但是我們可以使用 `NewAccessToken` 物件的 `plainTextToken` 屬性存取憑證值。我們在建立憑證之後應該向用戶顯示這個值。

```php
use Illuminate\Http\Request;

Route::post('/tokens/create', function (Request $request) {
  	$token = $request->user()->createToken($request->token_name);
  	return [
      'token' => $token->plainTextToken;
    ]
});
```

後續我們可以使用 `User` 物件的 `tokens` 關聯來存取憑證，這個關聯方法是通過 `HasApiTokens` trait 加入的，本質上是 Eloquent ORM 的關聯。

```php
foreach ($user->tokens as $token) {
  	// ...
}
```

### 憑證授權的能力

Sanctum 讓我們可以為憑證（Token）分配能力（Abilities），類似於 OAuth 的範圍（Scopes）。我們可以在 `createToken` 方法使用第二個參數傳入一組字串形式的權限陣列。

```php
return $user->createToken('token-name', ['server:update'])->plainTextToken;
```

當要處理由 Sanctum 驗證傳入的請求時，我們可以使用 `tokenCan` 或者 `tokenCant` 方法檢查憑證是否具備指定的能力。

```php
if ($user->tokenCan('server:update')) {
    // ...
}
 
if ($user->tokenCant('server:update')) {
    // ...
}
```

> 也就是當請求進來的時候，Sanctum 檢查了 `Authorization: Bearer <token>` 取得憑證。然後在資料庫 `personal_access_tokens` 找到對應的憑證，並關聯到 `User`。更進一步的說 `HasApiTokens` trait 會將憑證資訊留在當前的 `User` 提供後續 `tokenCan()` 等方法使用。
>
> 使用 Sanctum 你不會直接拿到一個 `$token` 變數來操作，而是透過 `$user` 來間接使用 Token 的資訊。它是透過 Auth 中介層（auth:sanctum）將 Token 資訊綁定到當前請求。
>
> 這也是為什麼這個方式是 `$user->tokenCan` 而不是使用 PersonalAccessToken 物件。

### 憑證能力中介層 Token Ability Middleware

Sanctum 同時包含 2 個中介層可以用來驗證包含 Token 經過驗證的請求是否有能力的授權。要使用中介層，我們可在 `bootstrap/app.php` 如下先註冊別名：

```php
use Laravel\Sanctum\Http\Middleware\CheckAbilities;
use Laravel\Sanctum\Http\Middleware\CheckForAnyAbility;

return Application::configure(basePath: dirname(__DIR__))
		// ...
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->alias([
          	'abilities' => CheckAbilities::class,
          	'ability' => CheckAnyAbility::class,
        ])
    })
  	// ...
```

`abilities` 可以設定到路由上驗證請求的 Token 憑證是否符合全部列表的能力權限：

```php
Route::get('/orders', function () {
  	//
})->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);
```

而 `ability` 則是至少有一個權限即可：

```php
Route::get('/orders', function () {
  
})->middleware('auth:sanctum', ['ability:check-status,place-orders']);
```

#### 我方應用程式 UI 發出的請求

為了方便起見，如果傳入已經驗證的請求來自我們的 SPA 應用程式，並且使用 Sanctum 內建的 SPA 驗證機制，那麼 `tokenCan` 永遠返回 `true`。

然而這不表示應用程式允許使用者執行任何操作。一般來說，應用程式使用 `php artisan make:policy` 建立的授權政策會決定該 Token 是否被賦予執行這些能力的權限，同時也會檢查使用者本身是否具備權限。

例如：我們的應用程式負責管理伺服器，我們會需要檢查該 Token 是否被授權更新伺服器，以及該用戶是否具備權限：

```php
return $request->user()->id == $server->user_id && $request->user()->tokenCan('server:update');
```

允許我們 UI 發起的請求在`tokenCan` 方法被呼叫時永遠回傳 `true` 看起來有點怪 ，但是這樣的設定假設 API 憑證始終可以使用 `tokenCan` 來檢查會比較方便。通過這個設計，我們在開發時都是使用 `tokenCan` 而不用去思考這是 UI 觸發的還是第三方呼叫 API。

進一步說假設我們有一個筆記應用：

- 如果使用者在我們自己的 SPA 編輯
- Sanctum 會確保 `tokenCan('note:edit')` 都是 `true`，因為這是我們 UI 觸發的。
- 但是 Laravel Policy 會繼續檢查該筆記是否屬於使用者，只有都符合才能編輯。
- 也就是第一方 UI 請求是基於 Session-based 驗證沒有 Token，但是我們一樣可以使用 `tokenCan` 來檢查權限。
- 更進一步的說也就是 `Policy` 和 `Gate` 還是要實作，作為權限檢查的最後守門。

### 防護路由

要限制路由的請求必須要驗證，我們要在 `routes/web.php` 和 `routes/api.php` 掛載 `sanctum` 驗證守衛 `Guard` 。守衛會確保傳入的請求有狀態、Cookie 驗證的請求、或者包含有效的 API Token。

您可能想知道為什麼我們建議在 `routes/web.php` 也使用 `sanctum` Guard。注意， Sanctum  會先嘗試使用 Laravel 預設一般的 Session 驗證來驗證請求的身份。如果 Cookie 不存在，則 Sanctum 會繼續嘗試使用請求的標頭來驗證。此外，對全部請求使用 Sanctum 進行身份驗證確保我們可以在當前的使用者物件實例呼叫 `tokenCan`：

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
  	return $request->user();
})->middleware('auth:sanctum');
```



> 關於 Laravel 內建的身份驗證與授權：
>
> - Guard：負責認證用戶身份（如 session 或 API token）。
> - Gate：檢查用戶是否允許執行特定功能（通用授權，閉包式）。
> - Policy：針對特定模型（如 Post）的操作（如 update、delete）進行授權檢查。

### 撤銷憑證

我們可以通過 `Laravel\Sanctum\HasApiTokens` 提供的 `tokens` 關聯刪除資料庫撤銷憑證

```php
$user->tokens()->delete();

$request->user()->currentAccessToken()->delete();

$user->tokens()->where('id', $tokenId)->delete();
```

### 憑證有效期限

預設，Sanctum 的憑證不會過期並且只有在撤銷的時候才會失效。然而，如果我們希望設定一個有效期限，我們可以通過 `expiration` 設定。這個設定為在發行之後的多少分鐘後過期：

```php
# config/sanctum.php

'expiration' => 525600,
```

若希望為每一個憑證設定不同的有效時間，可以在 `createToken` 的時候加入參數設定：

```php
return $user->createToken(
	'token-name', ['*'], now()->addWeek()
)->plainTextToken;
```

如果我們有設定憑證過期時間，可能也會希望設定一個任務來清理過期的 Token。Sanctum 提供了 `sanctum:prune-expired` Artisan 指令來完成這件事。例如我們可以設定一個排程任務刪除過期超過 24 小時的 Token：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('sanctum:prue-expired --hours=24)->daily();
```

## SPA 驗證

Sanctum 同時也提供一個簡單的方式讓需要和 Laravel API 溝通的 SPA 進行驗證。這些 SPA 可以是同一個 Laravel 應用程式也可以是獨立的專案。

針對這個功能，Sanctum 不使用任何類型的憑證。取而代之的是使用 Laravel 內建基於 Cookie Session 的驗證機制。這種身份驗證支援 CSRF，Session 驗證，並且可以防止 XSS 洩漏憑證的問題。

為了驗證我們的 SPA 和 API 必須使用相同的根網域，也就是可以使用不同的子網域。此外我們應該要確保請求包含 `Accept: application/json` 標頭以及 `Referer` 或 `Origin` 。
### 設定

#### 設定網域

首先，我們應該要設定 SPA 的網址，也就是從哪裡發送請求。我們可以在 `sanctum` 設定檔案中設定網址使用`stateful` 。這個設定會判斷那些網域向我們 Laravel API 發送請求的時候保持使用 Cookie Session 維持狀態機制的驗證。

為了協助設定我們的網域，Sanctum 提供 2 個輔助函式可以加入設定。第一個是 `Sanctum::currentApplicationUrlWithPort()` 會從 `APP_URL` 環境變數回傳當前應用程式的 URL，而 `Sanctum::currentRequestHost()` 則會在狀態機制的網址列表中加入一個佔位符，該佔位符會在執行時被當前請求的主機替換，這樣全部相同網域的請求都會被視為要使用狀態機制（動態獲取當前請求的主機名，在執行時才確定具體的域名）。

如果存取的應用程式包含埠號，則要確保在網域名稱中包含連接埠。

#### Sanctum 中介層

接著，我們需要告訴 Laravel 來自 SPA 的請求使用 Cookie Session 進行身份認證，同時允許第三方或行動應用程式可以使用 API 憑證的方式進行身份認證。這一步可以在 `bootstrap/app.php` 檔案呼叫 `statefulApi` 中介層來實現。

```php
->withMiddleware(function (Middleware $middleware): void {
  	$middleware->statefulApi();
})
```

#### CORS 和 Cookie

如果在其他子網域進行身份驗證遇到問題，大概率是 CORS 跨來源資源共享或 Cookie Session 設定有問題。

`config/cors.php` 設定預設並沒有發佈到專案中。如果需要設定 Laravel 的 CORS 設定，則需要：

```sh
$ php artisan config:publish cors
```

下一步，我們確認應用程式的 CORS 回傳 `Access-Control-Allow-Credentials` 標頭值為 `True`。通過在 `config/cors.php` 設定 `supports_credentials` 為 `true`。

此外，我們要在全域 `axios` 物件實例啟用 `withCredentials` 和 `withXSRFToken` 。通常在 `resources/js/bootstrap.js` 中設定。如果前端不是使用 `axios` 發送請求，則需要找到相同效果的設定。

```js
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
```

最後，我們要確保應用程式的 Session Cookie 網址設定支援任何子網域。在 `config/session.php` 設定檔中為網域前面加一個 `.` 完成設定。

### 認證

#### CSRF 保護

要驗證 SPA，我們 SPA 的登入頁面應先發送請求到 `/sanctum/csrf-cookie` 初始化 CSRF 保護：

```js
axios.get('/sanctum/csrf-cookie').then(response => {
  // 登入操作
});
```

在這個請求的過程中，Laravel 會設定 `XSRF-TOKEN` cookie 包含當前的 CSRF 金鑰。然後需要對這個金鑰進行 URL 解密，並在後續的請求中搭配 `X-XSRF-TOKEN` 標頭一起送出。一些 HTTP 客戶端函式庫例如 Axios 和 Angular HttpClient 會自動處理這個操作。

> 這是源自 Angular 的慣例，Cookie 使用 XSRF-TOKEN ，標頭使用 X-XSRF-TOKEN 名稱，當我們呼叫 `/sanctum/csrf-token` 的時候，Laravel 會回應 Set-Cookie: XSRF-TOKEN=... 。瀏覽器會自動儲存 Cookie，當 Axios 發送下一個請求的時候會自動讀取 Cookie，進行 URL Decode 然後加入標頭。

如果您的 JavaScript HTTP 函式庫不會自動處理的話，需要手動設定 `X-XSRF-TOKEN` 標頭。

#### 登入

一旦 CSRF 保護初始化，接著應發送 `POST` 請求到我們 Laravel 的 `/login` 路由。這個部分可以[手動實作](https://laravel.com/docs/12.x/authentication#authenticating-users)或者使用 Laravel Fortify。

如果登入成功，身份將會被認證並且後續的請求會自動使用 Session Cookie 的機制通過驗證。此外由於我們已經向 `/sanctum/csrf-cookie` 發送請求，只要我們的 JavaScript 客戶端請求有在 `X-XSRF-TOKEN` 標頭包含 `XSRF-TOKEN` Cookie 的值，後續的請求應該會自動接收 CSRF。

當然，如果使用者的 Session 因為閒置而逾期，後續的請求可能會收到 401 或 419 HTTP 錯誤回應。在這種情況下，您應該導向使用者到登入頁面。

當然，我們可以自行開發 `/login` ，但要確保使用標準的 Session 驗證機制來執行身份認證。通常就是指使用 `web` 驗證守衛 Guard。

#### 保護路由

為了保護路由因此全部進來的請求必須要驗證，我們應該在 `routes/api.php` 掛載 `sanctum` 驗證 guard 到 API 路由。這個守衛將會確保進來的請求通過驗證為我們 SPA 來的狀態化機制的驗證或包含有效 API 憑證。

```php
use Illuminate\Request;

Route::get('/user', function (Request $request) {
  	return $request->user();
})->middleware('auth:sanctum');
```

### 授權私有廣播頻道

Private Broadcast Channels 也就是 Laravel 的即時通訊功能。私有頻道也就是特定身份才能監聽。如果 SPA 需要授權私有廣播頻道，我們需要在 `bootstrap/app.php` 檔案的 `withRouting` 方法中移除 `channels`。然後使用 `withBroadcasting` 方法。

```php
return Application::configure(basePath: dirname(__DIR__))
  ->withRouting(
			web: __DIR__.'/../routes/web.php',
      // ...這附近移除 channels
	)->withBroadcasting(
			__DIR__.'/../routes/channels.php',
      ['prefix' => 'api', 'middleware' => ['api', 'auth:sanctum']],
	);
```

接下來，為了讓 Pusher 的授權請求成功，我們需要在初始化 Laravel Echo 時提供自訂的 Pusher `authorizer` ，讓 Pusher 使用正確設定跨域請求的 `axios` ：

```js
window.Echo = new Echo({
    broadcaster: "pusher",
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    encrypted: true,
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    authorizer: (channel, options) => {
        return {
            authorize: (socketId, callback) => {
                axios.post('/api/broadcasting/auth', {
                    socket_id: socketId,
                    channel_name: channel.name
                })
                .then(response => {
                    callback(false, response.data);
                })
                .catch(error => {
                    callback(true, error);
                });
            }
        };
    },
})
```

### 行動應用程式驗證

我們也可以使用 Sanctum 的憑證來驗證行動應用程式對於 API 的請求。這個流程類似於驗證第三方 API 請求；但是在發行 API 憑證有些許不同。

#### 發行 API 憑證

一開始建立一個路由接收使用者的電子郵件、使用者名稱、密碼和裝置名稱，然後用這些身份識別資訊交換 Sanctum 憑證。裝置名稱僅提供資訊參考，可以是任何希望的值。通常，裝置名稱應該是使用者可以識別的資訊例如 "Nuno's iPhone 12"。

一般來說，我們會在行動應用程式的登入頁面發送請求取得憑證。而 API 會回傳純文字 API 憑證資訊，儲存在行動裝置後續發送請求使用。

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

Route::post('/sanctum/token', function (Request $request) {
  	$request->validate([
      	'email' => 'required|email',
      	'password' => 'required',
      	'device_name' => 'required',
    ]);
  
  	$user = User::where('email', $request->email)->first();
  
  	if (! $user || ! Hash::check($request->password, $user->password)) {
      	throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }
  	
  	return $user->createToken($request->device_name)->plainTextToken;
});
```

當行動應用程式使用這個憑證發送請求，應該要包含 `Authorization:` 標頭和憑證。

另外，建立憑證的時候也可以指定能力（Ability）。

#### 保護路由

如同之前文件提到，我們可以保護路由，那麼進來的請求必須通過掛載 `sanctum` 的身份驗證。

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

#### 撤銷憑證

為了允許使用者撤銷發行給行動裝置的 API 憑證，我們可以在介面提供名稱列表和撤銷按鈕。當使用者點擊按鈕時，我們可以刪除資料庫的憑證。注意，我們可以使用 `Laravel\Sanctum\HasApiTokens` trait 建立的關聯來存取憑證進行刪除。

```php
$user->tokens()->delete();

$user->tokens()->where('id', $tokenId)->delete();
```



### 測試

測試的時候，可以使用 `Sanctum::actingAs` 方法來驗證使用者並指定憑證和授權的能力。

```php
use App\Models\User;
use Laravel\Sanctum\Sanctum;

test('task list can be retrieved', function () {
  	Sanctum::actingAs(
    		User::factory()->create(),
      	['view-tasks']
    );
  
  	$response = $this->get('/api/task');
  	
  	$response->assertOk();
  
})
```

如果你希望授權全部的能力可以使用 `*` 。

```php
Sanctum::actingAs(
    User::factory()->create(),
    ['*']
);
```



## 筆記重點

```sh
# ===== 安裝 =====
php artisan install:api

# ===== 基本設定 =====
// User Model 加入 trait
use Laravel\Sanctum\HasApiTokens;
class User extends Authenticatable {
    use HasApiTokens;
}

# ===== API Token 管理 =====
// 建立 Token（回傳給用戶，只顯示一次）
$token = $user->createToken('token-name', ['server:update'])
              ->plainTextToken;

// 檢查 Token 能力
$user->tokenCan('server:update');   // 有權限
$user->tokenCant('server:update');  // 無權限

// 撤銷 Token
$user->tokens()->delete();                    // 全部
$user->currentAccessToken()->delete();        // 當前
$user->tokens()->where('id', $id)->delete();  // 特定

// 設定過期時間（config/sanctum.php）
'expiration' => 525600,  // 分鐘
// 或建立時指定
$user->createToken('name', ['*'], now()->addWeek());

# ===== Ability 中介層 =====
// bootstrap/app.php 註冊
$middleware->alias([
    'abilities' => CheckAbilities::class,      // 全部符合
    'ability' => CheckForAnyAbility::class,    // 至少一個
]);

// 路由使用
->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);  // AND
->middleware(['auth:sanctum', 'ability:check-status,place-orders']);    // OR

# ===== 權限檢查模式（重要概念）=====
// 統一寫法：SPA 和 API 都用同一套
if ($user->tokenCan('note:edit')) {           // 第一關：Token 能力
    $this->authorize('update', $note);        // 第二關：Policy 檢查
    $note->update($request->all());
}

// SPA 請求：tokenCan 永遠 true，只檢查 Policy
// API 請求：tokenCan 檢查 abilities，再檢查 Policy

# ===== 路由保護 =====
Route::get('/user', fn(Request $req) => $req->user())
    ->middleware('auth:sanctum');

# ===== SPA 設定 =====
// 1. config/sanctum.php - 設定允許的網域
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost'));

// 2. bootstrap/app.php - 啟用 statefulApi
->withMiddleware(function (Middleware $middleware) {
    $middleware->statefulApi();
})

// 3. 前端初始化 CSRF（登入前）
axios.get('/sanctum/csrf-cookie').then(() => {
    // 執行登入
});

// 4. axios 設定
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;

# ===== 行動應用 Token 發行 =====
Route::post('/sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);
    
    $user = User::where('email', $request->email)->first();
    
    if (!$user || !Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['帳密錯誤'],
        ]);
    }
    
    return $user->createToken($request->device_name)->plainTextToken;
});

# ===== 測試 =====
Sanctum::actingAs(User::factory()->create(), ['view-tasks']);
// 或授權全部能力
Sanctum::actingAs($user, ['*']);

# ===== 關鍵概念提醒 =====
// ✅ Token 存在 personal_access_tokens 表
// ✅ 請求帶 Authorization: Bearer {token}
// ✅ SPA 用 Session Cookie，API 用 Bearer Token
// ✅ tokenCan 不能取代 Policy，兩者要搭配使用
// ✅ Token 只顯示一次（plainTextToken），儲存前已 SHA-256 加密
```





 

---
title: Laravel 8 - 快速整合 Jetstream + Socialite
categories: Program
tags:
  - laravel
  - jetstream
---


本文為 [Laravel 7 — Socialite in Action ( Social Media Login Integration with Facebook, Twitter, LinkedIn, Google)](https://medium.com/@andyyou/laravel-7-socialite-in-action-social-media-login-integration-with-facebook-twitter-linkedin-9661a089bc41) 之更新簡化版本。省略大部分說明只提供步驟紀錄。詳細說明請參考原文。

<!-- more -->

## Create Project

```sh
$ laravel new demo
```
建立專案之後，請建立資料庫和更新 `.env`。下面以 Postgre SQL 為例

```sh
$ createdb demo
```

`.evn` 的部分如下：

```
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=demo
DB_USERNAME=root
DB_PASSWORD=
```

## 安裝 Jetstream

```sh
$ composer require laravel/jetstream
$ php artisan jetstream:install inertia --teams
$ npm install && npm run dev
# (opt) For customize template you should publish these views
$ php artisan vendor:publish --tag=jetstream-views
```

## 安裝 Socialite

```sh
$ composer require laravel/socialite
# (opt) If you meet memory limit error, place `COMPOSER_MEMORY_LIMIT=-1` before command
# $ COMPOSER_MEMORY_LIMIT=-1 composer require laravel/socialite
```

## 修改 database schema

詳細原因請參考 [Laravel 7 — Socialite in Action ( Social Media Login Integration with Facebook, Twitter, LinkedIn, Google)](https://medium.com/@andyyou/laravel-7-socialite-in-action-social-media-login-integration-with-facebook-twitter-linkedin-9661a089bc41)


```js
$ composer require doctrine/dbal
$ php artisan make:migration edit_columns_in_users_table
```

在新增的 migration 檔案中調整如下:

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropUnique(['email']);
        $table->string('password')->nullable()->change();
        $table->json('social')->nullable();
        $table->softDeletes();
        $table->unique(['email', 'deleted_at']);
    });
}

public function down()
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropUnique(['email', 'deleted_at']);
        $table->dropSoftDeletes();
        $table->dropColumn(['social']);
        $table->string('password')->change();
        $table->string('email')->unique()->change();
    });
}
```

然後執行

```sh
$ php artisan migrate
```

## Model

```php
use Illuminate\Database\Eloquent\SoftDeletes;
// ...
// If you want to support verify you can add implements
class User extends Authenticatable
{
  use Notifiable;
  use SoftDeletes;
  
  // ...
  
  /**
   * The attributes that should be cast to native types.
   *
   * @var array
   */
  protected $casts = [
      'email_verified_at' => 'datetime',
      'social' => 'array',
  ];
  
  // (opt) Most of case you should keep email in lowercase
  public function setEmailAttribute($value)
  {
      $this->attributes['email'] = strtolower($value);
  }
}
```

## Fortify

有兩個 Fortify 相關的檔案須修正，原因是我們現在支援 `SoftDeletes`，注意 `email` 的規則 `unique` 須置換為 `unique:users,email,NULL,id,deleted_at,NULL`

```php
// app/Actions/Fortify/CreateNewUser.php
<?php
// ...
class CreateNewUser implements CreatesNewUsers
{
    use PasswordValidationRules;
  	// ...
    public function create(array $input)
    {
        Validator::make($input, [
            // ...
            'email' => ['required', 'string', 'email', 'max:255', 'unique:users,email,NULL,id,deleted_at,NULL'],
            // ...
        ])->validate();

        return DB::transaction(function () use ($input) {
            return tap(User::create([
                'name' => $input['name'],
                'email' => $input['email'],
                'password' => Hash::make($input['password']),
            ]), function (User $user) {
                $this->createTeam($user);
            });
        });
    }
		// ...
}

```

```php
// app/Actions/Fortify/UpdateUserProfileInformaiton.php
<?php

// ...

class UpdateUserProfileInformation implements UpdatesUserProfileInformation
{
    // ...
    public function update($user, array $input)
    {
        Validator::make($input, [
            // ...
            'email' => ['required', 'email', 'max:255', 'unique:users,email,NULL,id,deleted_at,NULL'],
            // ...
        ])->validateWithBag('updateProfileInformation');

        // ...
    }

    // ...
}

```

## 取得平台憑證 Client ID 和 Secret

將您需要的資訊補在 `.env`，下面只是局部平台的範例

```
FACEBOOK_CLIENT_ID=
FACEBOOK_CLIENT_SECRET=
FACEBOOK_CALLBACK_URL=/login/facebook/callback
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_CALLBACK_URL=/login/google/callback
LINKEDIN_CLIENT_ID=
LINKEDIN_CLIENT_SECRET=
LINKEDIN_CALLBACK_URL=/login/linkedin/callback
TWITTER_CLIENT_ID=
TWITTER_CLIENT_SECRET=
TWITTER_CALLBACK_URL=/login/twitter/callback
```

## Services 設定

在 `config/services.php` 加入設定，您可能注意到 `scopes` 的部分，但在官方文件沒有關於這段。的確這是額外的設定，我覺得將它們放在一起比較合適您也可以放到 `.env`。

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
],
'facebook' => [
    'client_id' => env('FACEBOOK_CLIENT_ID'),
    'client_secret' => env('FACEBOOK_CLIENT_SECRET'),
    'redirect' => env('FACEBOOK_CALLBACK_URL'),
    'scopes' => ['email', 'public_profile'],
],
'google' => [
    'client_id' => env('GOOGLE_CLIENT_ID'),
    'client_secret' => env('GOOGLE_CLIENT_SECRET'),
    'redirect' => env('GOOGLE_CALLBACK_URL'),
    'scopes' => [
        'https://www.googleapis.com/auth/userinfo.email',
        'https://www.googleapis.com/auth/userinfo.profile',
        'openid',
    ],
],
'linkedin' => [
    'client_id' => env('LINKEDIN_CLIENT_ID'),
    'client_secret' => env('LINKEDIN_CLIENT_SECRET'),
    'redirect' => env('LINKEDIN_CALLBACK_URL'),
    'scopes' => ['r_emailaddress', 'r_liteprofile'],
],
'twitter' => [
    'client_id' => env('TWITTER_CLIENT_ID'),
    'client_secret' => env('TWITTER_CLIENT_SECRET'),
    'redirect' => env('TWITTER_CALLBACK_URL'),
    'scopes' => [],
],
```

## Auth Controller

這是本文最重要的段落，也可能是您一直在尋找的部分。我們將建立一個 Controller 來處理 OAuth 回呼的部分

```sh
$ php artisan make:controller Auth/LoginController
```

檔案建立之後，下面是完整的 `app/Http/Controllers/Auth/LoginController.php` 程式碼，雖然有點長，但方便您直接複製貼上並完整理解

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Providers\RouteServiceProvider;
use Illuminate\Support\Facades\Auth;
use Illuminate\Http\Request;
use Socialite;

use App\Models\User;
use App\Models\Team;


class LoginController extends Controller
{
    protected $redirectTo = RouteServiceProvider::HOME;

    /**
     * Redirect to authentication page based on $provider.
     * 
     * @param string $provider
     * @return \Illuminate\Http\Response
     */
    public function redirectToProvider(string $provider)
    {
        try {
            $scopes = config("services.$provider.scopes") ?? [];
            if (count($scopes) === 0) {
                return Socialite::driver($provider)->redirect();
            } else {
                return Socialite::driver($provider)->scopes($scopes)->redirect();
            }
        } catch (\Exception $e) {
            abort(404);
        }
    }

    /**
     * Obtain the user information from $provider
     * 
     * @param string $provider
     * @return \Illuminate\Http\Response
     */
    public function handleProviderCallback(string $provider)
    {
        try {
            $data = Socialite::driver($provider)->user();
            
            return $this->handleSocialUser($provider, $data);
        } catch (\Exception $e) {
            return redirect('login')->withErrors(['authentication_deny' => 'Login with '.ucfirst($provider).' failed. Please try again.']);
        }
    }

    /**
     * Handles the user's information and creates/updates
     * the record accordingly.
     *
     * @param string $provider
     * @param object $data
     * @return \Illuminate\Http\Response
     */
    public function handleSocialUser(string $provider, object $data)
    {
        $user = User::where([
            "social->{$provider}->id" => $data->id,
        ])->first();

        if (!$user) {
            $user = User::where([
                'email' => $data->email,
            ])->first();
        }

        if (!$user) {
            return $this->createUserWithSocialData($provider, $data);
        }

        $social = $user->social;
        $social[$provider] = [
            'id' => $data->id,
            'token' => $data->token
        ];
        $user->social = $social;
        $user->save();

        return $this->socialLogin($user);
    }

    /**
     * Create user
     *
     * @param string $provider
     * @param object $data
     * @return \Illuminate\Http\Response
     */
    public function createUserWithSocialData(string $provider, object $data)
    {
        try {
            $user = new User;
            $user->email = $data->email;
            $user->name = $data->name;
            $user->social = [
                $provider => [
                    'id' => $data->id,
                    'token' => $data->token,
                ],
            ];
            // markEmailAsVerified() contains save() behavior
            $user->markEmailAsVerified();
            $team = Team::forceCreate([
                'user_id' => $user->id,
                'name' => $user->name."'s Team",
                'personal_team' => true,
            ]);
            $user->current_team_id = $team->id;
            $user->save();

            return $this->socialLogin($user);
        } catch (Exception $e) {
            return redirect('login')->withErrors(['authentication_deny' => 'Login with '.ucfirst($provider).' failed. Please try again.']);
        }
    }

    /**
     * Log the user in
     *
     * @param User $user
     * @return \Illuminate\Http\Response
     */
    public function socialLogin(User $user)
    {
        auth()->loginUsingId($user->id);

        return redirect($this->redirectTo);
    }
}
```

## Routes

在 `routes/web.php`，注意 Laravel 8 的路由設定語法有些改變。

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Auth\LoginController;

// ...

Route::middleware(['auth:sanctum', 'verified'])->get('/dashboard', function () {
    return Inertia\Inertia::render('Dashboard');
})->name('dashboard');

Route::get('/login/{provider}', [LoginController::class, 'redirectToProvider'])
    ->name('social.login');
Route::get('/login/{provider}/callback', [LoginController::class, 'handleProviderCallback'])
    ->name('social.callback');

```


## Views

最後在  `resources/views/auth/` 補上登入的按鈕則完成。使用 `php artisan serve` 測試看看吧。

```php
@php
  $providers = [
    'google' => [
      'bgColor' => '#ec462f',
      'icon' => 'fab fa-google',
    ],
    'facebook' => [
      'bgColor' => '#1877f2',
      'icon' => 'fab fa-facebook-f',
    ],
    'linkedin' => [
      'bgColor' => '#2969b1',
      'icon' => 'fab fa-linkedin-in',
    ],
    'twitter' => [
      'bgColor' => '#41aaf1',
      'icon' => 'fab fa-twitter',
    // ],
  ];
@endphp

@foreach($providers as $provider => $params)
  <a 
    class="block py-3 px-4 mb-5/2 rounded-sm text-white text-center font-bold hover:no-underline hover:opacity-75" 
    href="{{ route('social.login', ['provider' => $provider]) }}"
    style="background-color: {{ $params['bgColor'] }}; min-height: 48px;"
  >
    <i class="tw-float-left tw-inline-block tw-h-5 {{ $params['icon'] }}"></i>
    Login with {{ ucwords($provider) }}
  </a>
@endforeach
```




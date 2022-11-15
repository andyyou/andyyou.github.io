---
title: Laravel 如何在 Jetstream 或 Fortify 覆寫登入後導向
date: 2022-07-15 10:58:04
tags:
  - laravel
categories: Program
---

由於專案已經陸續採用 Laravel Jetstream 並且遇到一個需求; 根據使用的類型在登入時導向不同路由。想像一個情境，您有一般使用者和管理員，而管理員擁有後台功能。通常可能會需要將管理員導向後台介面。

使用 Laravel Jetstream 或 Fortify，要實作這個功能並不直覺，特別是使用兩階段驗證。

<!-- more -->

## Jetstream 和 Fortify 驗證機制

Jetstream 底層使用 Laravel Fortify，因此流程是一樣的。

Fortify 使用 Action Pipeline 來將請求派送到一系列的 class 類別，這些類別各自負責一個任務，例如嘗試驗證使用者或將他們導向兩階段驗證。

## 如何覆寫導向

Fortify 支援完全自訂這些 Pipeline 但有比較簡單的方式覆寫導向的部分。

在驗證流程的最後一步 Pipeline 會從服務容器中擷取 `Laravel\Fortify\Contracts\LoginResponse` 類別並回傳。意思是我們可以覆寫 `LoginResponse` 達到自訂導向的需求。

預設 `LoginResponse` 看起來如下:

```php
<?php

namespace Laravel\Fortify\Http\Responses;

use Laravel\Fortify\Contracts\LoginResponse as LoginResponseContract;

class LoginResponse implements LoginResponseContract
{
  	/**
     * Create an HTTP response that represents the object.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function toResponse($request)
    {
        return $request->wantsJson()
                    ? response()->json(['two_factor' => false])
                    : redirect()->intended(config('fortify.home'));
    }
}
```

由於 Fortify 使用多個 Action 來執行個別人任務，`LoginResponse` 這一步執行的任務非常簡單乾淨。

要自訂導向，首先將在 `app/Http/Responses` 加入自訂的 `LoginResponse` ，然後自訂 `toResponse` 方法讓使用者基於我們的條件導向不同的路由

```php
<?php

namespace App\Http\Responses;

use Laravel\Fortify\Contracts\LoginResponse as LoginResponseContract;

class LoginResponse implements LoginResponseContract
{
    /**
     * @param  $request
     * @return mixed
     */
    public function toResponse($request)
    {
        $home = auth()->user()->is_admin ? '/admin' : '/dashboard';

        return redirect()->intended($home);
    }
}
```

然後在 `FortifyServiceProvider` ，我們可以繫結自訂的 `LoginResponse` 覆寫 Fortify 預設提供的。

```php
<?php
namespace App\Providers;
// ...
use App\Http\Responses\LoginResponse;
use Laravel\Fortify\Contracts\LoginResponse as LoginResponseContract;
class FortifyServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        // ...
        $this->app->singleton(LoginResponseContract::class, LoginResponse::class);
    }
}
```

## 兩階段驗證

這個解決方案幾乎完美的解決了我們的需求。但有一個缺失的地方，就是兩階段驗證。如果使用者有兩階段驗證，那麼 Fortify 會回傳不一樣的 `Response` 類別。幸好，兩階段驗證也是使用服務容器繫結。這次我們需要覆寫容器中的 `Laravel\Fortify\Http\Responses\TwoFactorLoginResponse` 類別，然後一樣在 `FortifyServiceProvider` 繫結。

```php
<?php
namespace App\Providers;
// ...
use App\Http\Responses\LoginResponse;
use Laravel\Fortify\Contracts\LoginResponse as LoginResponseContract;
use Laravel\Fortify\Contracts\TwoFactorLoginResponse as TwoFactorLoginResponseContract;
class FortifyServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        // ...
        $this->app->singleton(LoginResponseContract::class, LoginResponse::class);
        $this->app->singleton(TwoFactorLoginResponseContract::class, LoginResponse::class);
    }
}
```

> 注意: 我們直接使用同樣的 `LoginResponse` 覆寫 `TwoFactorLoginResponse`，因為兩者的功能應該是一樣的。

## 覆寫其他 Jetstream 和 Fortify 功能

其他在 Jetstream 和 Fortify 的功能也可以用類似的方式自訂。如果您深入套件 `FortifyServiceProvider` 的程式碼在 `registerResponseBindings` 方法中應該可以看到

```php
<?php

namespace Laravel\Fortify;

// ...

class FortifyServiceProvider extends ServiceProvider
{
    // ...

    /**
     * Register the response bindings.
     *
     * @return void
     */
    protected function registerResponseBindings()
    {
        $this->app->singleton(FailedPasswordConfirmationResponseContract::class, FailedPasswordConfirmationResponse::class);
        $this->app->singleton(FailedPasswordResetLinkRequestResponseContract::class, FailedPasswordResetLinkRequestResponse::class);
        $this->app->singleton(FailedPasswordResetResponseContract::class, FailedPasswordResetResponse::class);
        $this->app->singleton(FailedTwoFactorLoginResponseContract::class, FailedTwoFactorLoginResponse::class);
        $this->app->singleton(LockoutResponseContract::class, LockoutResponse::class);
        $this->app->singleton(LoginResponseContract::class, LoginResponse::class);
        $this->app->singleton(TwoFactorLoginResponseContract::class, TwoFactorLoginResponse::class);
        $this->app->singleton(LogoutResponseContract::class, LogoutResponse::class);
        $this->app->singleton(PasswordConfirmedResponseContract::class, PasswordConfirmedResponse::class);
        $this->app->singleton(PasswordResetResponseContract::class, PasswordResetResponse::class);
        $this->app->singleton(RegisterResponseContract::class, RegisterResponse::class);
        $this->app->singleton(SuccessfulPasswordResetLinkRequestResponseContract::class, SuccessfulPasswordResetLinkRequestResponse::class);
    }

    // ...
}
```

這些 Response 類別都是可以自訂的。

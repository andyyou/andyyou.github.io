---
title: "[譯]Laravel 介面最佳實作"
date: 2024-07-08 16:07:15
tags:
  - laravel
categories: Program
---

首先，介面（Interface）是物件導向的基礎，在 Laravel 中，介面被用來定義規範 `Contracts` ，即規定類別 class 必須實作的方法。讓我們可以建立彈性且模組化的程式碼且容易維護和測試。

本文我們將探索如何在 Laravel 中使用介面實作，如何為 `Service` 定義介面，使用相依注入 DI（Dependency Injection） ，繫結 `Service` 到應用程式 `Container`。

另外補充；``Factory` 工廠模式是一種建立物件的設計模式，主要用途除了生成測試資料還可以協助「相依注入」建立需要的物件。上面幾個概念將有助於我們實現更具可維護性，可讀性的程式碼。

<!-- more -->


## 為 Service 定義 Interface

在 Laravel，Service 被用來封裝複雜的商業邏輯。通過定義 Service 的 Interface，我們可以輕鬆的確保切換或擴展功能不會影響既有的程式碼。

讓我們使用一個金流服務的例子來說明，首先讓我們先定義 Interface

```php
// A payment gateway is a technology platform that acts as an intermediary in electronic financial transactions.
interface PaymentGetewayInterface {
  public function checkout(int $amount);
  public function refund(int $transactionId);
}
```

在範例中我們建立了支付中間商的介面 `PaymentGatewayInterface` 其規定了任何實作此介面的類別必須實作的 2 個方法 `checkout` 和 `refund`



## 使用 DI (Dependency Injection)

Laravel 強大的相依注入 DI 系統使其可以在我們的程式碼中輕鬆的使用介面。通過將介面注入到您的類別 class 中，可以確保這些類別只依賴於介面中定義的方法，而不是依賴於特定的實現。

接續上面的範例，我們可以在負責執行付款的地方注入介面：

```php
class PaymentProcesser {
  protected $gateway;
  
  public function __construct(PaymentGatewayInterface $gateway) {
    $this->gateway = $gateway;
  }
  
  public function checkout(int $amount) {
    $this->geteway->checkout($amount);
  }
  
  public function refound($transactionId) {
    $this->gateway->refund($transactionId);
  }
}
```

> 快速補充 DI 和 IoC
>
> 相依注入 DI 和控制反轉 IoC 都是設計模式，簡單的說就是儘量不要直接建立物件，而是通過從外部傳入「注入」取得。而 IoC 者是把建立和管理物件的責任交給容器而不是自行管理。
>
> ```php
> // 沒有使用 DI
> class MailService {
>   public function send($to, $subject, $body) {
>     $mailer = new SmtpMailer();
>     $mailer->send($to, $subject, $body);
>   }
> }
> 
> // DI 版本
> class MailService {
>   private $mailer;
> 
>   public function __construct(MailerInterface $mailer) {  // 從外部注入
>       $this->mailer = $mailer;
>   }
>   
>   public function send($to, $subject, $body) {
>     $this->mailer->send($to, $subject, $body);
>   }
> }
> ```
>
> 而在 Laravel 中，所謂 IoC 容器就是 Service Container
>
> ```php
> // 定義如何創建 EmailService
> $container->bind('EmailService', function($container) {
>     return new EmailService($container->make('SmtpMailer'));
> });
> ```



## 繫結 Service 到 Application Container

**為了在我們應用程式任何地方可以方便的使用 Service，我們可以用 Service Provider 將它們綁定註冊到 Laravel 應用程式容器。**Laravel 的 Application Container 應用程式容器也叫做 Service Container 是整個框架的核心，這個容器主要負責建立和管理程式中各種物件，自動解析依賴關係，並將物件注入到類別中。Laravel 內部的數種功能例如郵件發送、Queue、Cache、等都是利用這種方式在需要的時候加載。而開發者定義的服務則在 `bootstrap/providers.php` 檔案註冊。

下面繼續我們的範例：

```php
class AppServiceProvider extends ServiceProvider
{
  public function register()
  {
    app()->bind(PaymentGatewayInterface::class, function ($app) {
      	return collect([
          'paypal' => app(PaypalGateway::class),
          'stripe' => app(StripeGateway::class),
        ]);
    });
  }
}
```

上面例子，我們使用 `bind` 繫結了 Payment Gateway 的集合到容器，這裡的重點是「鍵」為 `PaymentGatewayInterface::class` ，和「值」是閉包其回傳了支付供應商處理器。這個集合包含了兩個處理器 `paypal` 和 `stripe`。



## 在 Controller 使用 Service

要在我們的 Controller 使用 Payment Gateway 服務，我們可以 Controller 中簡單的實例化一個 `PaymentProcesser` 物件，Laravel 容器會協助我們解析 `PaymentGatewayInterface` 

```php
class PaymentController extends Controller
{
  public function checkout()
  {
    $gateway = app(PaymentGatewayInterface::class)->get('paypal');
    $processor = new PaymentProcessor($gateway);
    $processor->checkout(100);
  }
  
  public function refund()
  {
    $gateway = app(PaymentGatewayInterface::class)->get('paypal');
    $processor = new PaymentProcessor($gateway);
    $processor->refund(100);
  }
}
```

後續我們就可以輕鬆的切換供應商。



## 基於 Interface 測試

下面我們嘗試使用 `PaymentGatewayInterface` 來測試 `PaymentProcessor` 類別：

```php
class PaymentProcessorTest extends TestCase
{
  public function testProcessMethodCallsCheckoutOnGateway()
  {
    $gateway = $this->createMock(PaymentGatewayInterface::class);
    $gateway->expects($this->once())
      ->method('checkout')
      ->with($this->equalTo(100));
    $processor = new PaymentProcessor($gateway);
    $processor->checkout(100);
  }
}
```

## 參考

- [Interfaces in Laravel: Best Practices for Maintainable and Testable Code](https://dev.to/tuandp/interfaces-in-laravel-best-practices-for-maintainable-and-testable-code-24o0)
- [Laravel 最佳實踐](https://github.com/alexeymezenin/laravel-best-practices/blob/master/traditional-chinese.md#mass-assignement---%E5%A4%A7%E9%87%8F%E8%B3%A6%E5%80%BC)




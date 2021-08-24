---
title: 設定 Heroku 使用 Cloudflare HTTPS
date: 2017-02-14 10:21:33
categories: Cloud
tags:
  - heroku
  - cloudflare
---

# 設定 Heroku 使用 Cloudflare HTTPS

<!--more-->

Heroku 是一個 SaaS 雲端平台，支援多種程式語言應用程式可快速部署。使用者不需要擔心底層的機器與設定，只需要專注在應用程式。

這篇文章將說明：如何設定 Cloudflare 使用自有的網域搭配 Heroku 讓我們的傳輸使用 HTTPS。針對這個需求我們假設您已經在 Cloudflare 有一組設定好的網域，同時有一個正在運行的 Heroku app。

## 加入自訂網域到 Heroku app

1. 登入 Heroku 選擇該 App 切換到`設定`頁面。  
![](https://support.cloudflare.com/hc/en-us/article_attachments/202162557/image01.png)
2. 捲至下方 `Domains` 輸入您的網域，儲存。  
![](https://support.cloudflare.com/hc/en-us/article_attachments/202162567/image06.jpg)

如果您是透過 Heroku CLI 管理那麼則使用

```bash
$ heroku domain:add [domain]
# 例如
$ heroku domain:add andyyou.io
$ heroku domain:add www.andyyou.io
```

## 設定 Cloudflare DNS

Heroku 的應用程式伺服器使用了多組的 IP，所以使用 A record 對應單一 IP 可能影響您的應用程式。因此我們將使用 CANME Flattening 的功能來動態解析對根網址的請求。

> 簡單說 CNAME Flattening 就是把 CNAME 設定的值 動態解析成 IP。在下面情況，這個技術非常有用；舉例來說我們使用了 Heroku 這類的雲端服務，通常這些服務會給我們一組 subdomain，在不用 SSL 的情況下到也沒有差，不過一但要使用 SSL 憑證和 CNAME 值所設定的域名就會對不上。另外一個常見的案例就是想把 root domain 直接對應到服務，不過一般來說 root domain 只能設 A Record。

## root domain 使用 CNAME Flattening

1. 登入 Cloudflare 然後切換到 DNS 介面。
2. 加入新的 CNAME 讓 root domain 根網域對應到 Heroku APP URL（例如：cf-app-test.herokuapp.com）。到了這一步 CNAME Flattening 會自動生效，將根網域對應到 Heroku 的伺服器。

## 子網域 使用 CNAME Flattening

如果您的網站有其他子網域也要為其加入 CNAME。注意，CNAME Flattening 預設只提供根網域，當您需要支援整個 Zone 時`請連絡支援團隊`，在 Support 介面下開票。

## 確認網域路由是使用 Cloudflare

```bash
$ curl -I martijn.cf
HTTP/1.1 200 OK
Date: Wed, 24 Jun 2015 02:46:16 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Set-Cookie: __cfduid=d981b13a67668bc3dba1f4cf901450cf21435113976; expires=Thu, 23-Jun-16 02:46:16 GMT; path=/; domain=.martijn.cf; HttpOnly
X-Frame-Options: SAMEORIGIN
Via: 1.1 vegur
Server: cloudflare-nginx
CF-RAY: 1fb519efe503119b-SJC
```

您可以透過 `__cfuid` cookie 或 `CF-Ray` 來判斷是否已經由 Cloudflare 來代理。假設兩個值都存在則表是已經由 Cloudflare 代理了。

## 設定網域的 SSL

Cloudflare 為付費方案提供 SANs 通用型憑證，免費方案則是 SNI 通用型憑證。
利用 Cloudflare 設定 SSL 非常簡單。只要到 Cloudflare 的 Crypto 介面設定即可。同時 Cloudflare 提供不同的選項設定。

![](https://support.cloudflare.com/hc/en-us/article_attachments/202234318/Crypto__martijn_cf___CloudFlare_-_Web_Performance___Security.jpg)

## SSL
Cloudflare 提供了三種不同的 SSL 選項如下圖

![](https://support.cloudflare.com/hc/en-us/article_attachments/203519687/ssl.png)

預設 Heroku 提供通用 SSL 憑證，只適用 `*.herokuapp.com` 範圍下。 這表示在不要求 SAN 為完整網域名稱的情況下可以使用 `Full SSL` 。

如果要使用嚴格模式 Full(Strict) 我們就需要在 Heroku 上掛上自己的憑證，詳細的[安裝說明](https://devcenter.heroku.com/articles/ssl-endpoint)可參考官方文件。

## 強迫全部傳輸使用 HTTPS

要強迫全部傳輸使用 HTTPS 我們可以透過 Page Rules 的功能來完成。

![](https://support.cloudflare.com/hc/en-us/article_attachments/202163627/Page_Rules__martijn_cf___CloudFlare_-_Web_Performance___Security.jpg)

進入 Page rules 頁面加入新的規則：

![](https://support.cloudflare.com/hc/en-us/article_attachments/202234358/Page_Rules__martijn_cf___CloudFlare_-_Web_Performance___Security_2.jpg)

再此透過 cURL 來檢查

```
$ curl -I -L martijn.cf

HTTP/1.1 301 Moved Permanently
Date: Wed, 24 Jun 2015 04:06:00 GMT
Connection: keep-alive
Set-Cookie: __cfduid=d37122464f26484c74e2359938310ac611435118760; expires=Thu, 23-Jun-16 04:06:00 GMT; path=/; domain=.martijn.cf; HttpOnly
Location: https://martijn.cf/
Server: cloudflare-nginx
CF-RAY: 1fb58ebf219111b9-SJC

HTTP/1.1 200 OK
Server: cloudflare-nginx
Date: Wed, 24 Jun 2015 04:06:01 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Set-Cookie: __cfduid=dacb13cbfbc6415248b68d3c51e03a8001435118761; expires=Thu, 23-Jun-16 04:06:01 GMT; path=/; domain=.martijn.cf; HttpOnly
X-Frame-Options: SAMEORIGIN
Via: 1.1 vegur
CF-RAY: 1fb58ec130851213-SJC
```

如果 SSL 沒運作那麼我們會看到 525 或 526 的錯誤。

# 資源

* [Configure Cloudflare and Heroku over HTTPS](https://support.cloudflare.com/hc/en-us/articles/205893698-Configure-CloudFlare-and-Heroku-over-HTTPS)
* [Rage4、CloudXNS、Route53 和 CloudFlare 解析](https://ze3kr.com/2016/05/rage4-best-dns/)

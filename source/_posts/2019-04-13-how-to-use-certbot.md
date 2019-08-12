---
title: 解析 Certbot（Let's encrypt） 使用方式
tags:
  - letsencrypt
  - ssl
categories: System
date: 2019-04-13 09:39:20
---

如果您曾好奇為什麼在網路上搜尋到關於 Let's encrypt 的設定有各式各樣的作法，或者想要好好的理解一下 certbot 的使用方式那麼本篇筆記就是您所需要的。

<!-- more -->

## Certbot

Certbot 是可在託管伺服器上執行的指令。如果您不具有 shell 的存取權限並且不熟悉相關指令，您應該確認託管供應商是否提供內建 Let's Encrypt 功能，[支援供應商列表](https://community.letsencrypt.org/t/web-hosting-who-support-lets-encrypt/6920)。



### 安裝

> 為了能夠快速的實作練習，筆者推薦使用任一雲端服務建立 VM 來測試。{% post_link google-cloud-platform-getting-start 使用 Google Cloud Platform 為例 %}
> 本文指令以 Ubuntu 18.04 環境為例。

Certbot 團隊自行維護 PPA(Personal Package Archive)。您可以將其加入檔案庫並依據下列指令安裝

```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository universe
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot python-certbot-nginx
```

##### DNS 套件（可選）

如果您希望從 Let's Encrypt 的 *ACMEv2 伺服器*自動取得憑證或在其他主機上取得憑證，您可以使用 DNS 套件，參考[套件列表](https://certbot.eff.org/docs/using.html#dns-plugins)取得更多使用資訊。
要安裝其他套件可以將上面 `sudo apt-get install certbot python-certbot-nginx` 指令換成 `sudo apt-get install python3-certbot-dns-PLUGIN`，大寫 PLUGIN 則是套件名稱。

> 如果您在其他教學看到使用 `certbot-auto` 指令，本質上和 `certbot` 是一樣的東西，不過`certbot-auto` 會自動安裝相依套件。但**請注意**：`certbot-auto` 目前（2019-04-12）不支援 DNS 套件，您還是得自行安裝套件後搭配 `certonly` 取得憑證。




### 基本使用

在網路上有許多關於 Certbot 的教學，您可能感到困惑許多教學的指令或設定流程並不相同。其原因是 Certbot 根據不同的情境提供不同的指令和作法，本質上分成單純取得憑證和自動化取得憑證搭配是否自動設定伺服器的部分就產生多種作法。

如果要支援自動化就必須通過 acme-challenge（http-01 或 dns-01 ）驗證，設定流程複雜的部分多來自於驗證階段的設定。

後續我們將依據下面列表逐一介紹

* 全自動 (自備 HTTP 伺服器)  `certbot`
* 半自動（自備 HTTP 伺服器，不調整 HTTP 伺服器設定）`certbot certonly`
* webroot （自備 HTTP 伺服器，自行設定 acme-challenge 部分）`certbot certonly --webroot`
* 手動（自備 HTTP 伺服器 、其他主機）`certbot certonly --manual`
* DNS 套件 `certbot certonly --dns-PLUGIN`
* Standalone（Certbot 提供獨立 HTTP 伺服器部分）`certbot certonly --standalone`

上面筆者只是依照常用的作法概略列出好讓您理解其中的差異，但這並沒有羅列所有作法，分類上也不嚴謹您應該參考[官方文件的說明](https://certbot.eff.org/docs/using.html#getting-certificates-and-choosing-plugins)。



##### 全自動設定

執行下面指令會自動取得憑證且 Certbox 會編輯 Nginx 的設定協助完成流程，一道指令完成設定。但前提是您的伺服器和 DNS 須先設定好一般 *http* 連線的部分。

```bash
$ sudo certbot --nginx

# 步驟說明（以下步驟皆於伺服器主機上執行）：
# 1. 建置一台 Linux 主機或 VM（Ubuntu 18.04）
# 2. 安裝 Nginx
# 3. 安裝 Certbox
# 4. 確認與設定 DNS 設定 A 紀錄指向主機，如果 DNS 沒設定，acme-challenge 會驗證失敗，失敗仍然可以取得憑證但後續無法自動更新。無論是 http-01  或 dns-01 都是驗證您是否擁有網域的控制權以得到後續自動操作的權限，只是驗證的方式有些微不同。
# 5. 執行 certbot 全自動指令
# 6. 完成


# 成功提升訊息
# Congratulations! You have successfully enabled <您的網址>
# You should test your configuration at:
# https://www.ssllabs.com/ssltest/analyze.html?d=<您的網址>
```




##### 半自動

如果您是保守派希望自己設定 Nginx ，則可以使用 `certonly`

```bash
$ sudo certbot --nginx certonly

# certonly 具體執行的步驟
# 1. 暫時的變更設定，加入新的 *Server 區塊設定* 以通過 ACME Challenge。
# 2. 重載 Nginx 設定。(sudo nginx -s reload)
# 3. 還原變更的設定。
# 4. 重載 Nginx 設定。

# 接著您要自行設定 Nginx 安裝憑證。
```

> [ACME Challenge](https://tools.ietf.org/html/draft-ietf-acme-acme-02#section-7.3)
>
> ACME Challenge 是為了要證明您具有網域控制權的一種驗證方式，主要是通過您設定一組 A/AAAA （IPv4/IPv6）紀錄指向伺服器，並在伺服器上驗證 challenge 憑證。這也是為什麼自動安裝憑證的流程需要您的網站以及網址是處於線上的狀態（DNS 指向伺服器，伺服器提供對應證明 hash）。

**全自動和半自動的差別是在最後是否自動在 Nginx 設定的 `server {}` 的部分為您自動加入 *ssl* 相關設定**，如果您不想讓 Certbot 接觸 Nginx 設定或著有線上的服務不能中斷的需求，您可以使用 `webroot` 套件。

> 注意：全自動和半自動只是筆者依據行為命名好讓您比較容易立即，實際上僅是不同指令的行為。


**另外，如果您希望直接從 Let's Encrypt 的新 ACMEv2 伺服器取得多子網域通用憑證，您還是需要使用 [Certbot 的 DNS 套件](https://certbot.eff.org/docs/using.html#dns-plugins)**。請確認您已經針對您 DNS 供應商安裝對應套件然後執行下面的指令：

```bash
$ sudo certbot -a dns-plugin -i nginx -d "*.example.com" -d example.com --server https://acme-v02.api.letsencrypt.org/directory
```

`dns-plugin` 的部分請替換成您想要使用的套件。例如：Cloudflare 要使用 `dns-cloudflare`。您可能需要提供額外的資訊例如 API 憑證的路徑，更多關於套件的使用請參考[相關文件](https://certbot.eff.org/docs/)



##### webroot

需要在伺服器不中斷下取得憑證。須自行設定 acme-challenge ，等於是不讓 Certbot 碰到 HTTP 伺服器設定的作法。

```bash
# 0. 安裝 Nginx 與 Certbot
# 1. 設定 acme-challenge，編輯 /etc/nginx/sites-available/default
$ sudo vi /etc/nginx/sites-available/default

# 2. 加入設定
location ^~ /.well-known/acme-challenge/ {
  default_type    "text/plain";
  root /var/www/html/;
}

# 完整設定參考
server {
  listen 80 default_server;
  listen [::]:80 default_server;

  # ... 省略 ...

  # ↓↓↓↓↓↓↓↓↓↓↓ 重點
  location ^~ /.well-known/acme-challenge/ {
    default_type    "text/plain";
    root /var/www/html/;
  }
	# ↑↑↑↑↑↑↑↑↑↑↑
}

# 3. 執行 Certbot 取得憑證，這裡的 /var/www/html/ 是因為我們直接使用 Nginx 預設站點
$ sudo certbot certonly --webroot -w /var/www/html/ -d <您的網址>

# 4. 成功之後會取得下面提升訊息

# Saving debug log to /var/log/letsencrypt/letsencrypt.log
# Plugins selected: Authenticator webroot, Installer None
# Obtaining a new certificate
# Performing the following challenges:
# http-01 challenge for <您的網址>
# Using the webroot path /var/www/html for all unmatched domains.
# Waiting for verification...
# Cleaning up challenges

# IMPORTANT NOTES:
#  - Congratulations! Your certificate and chain have been saved at:
#   /etc/letsencrypt/live/<您的網址>/fullchain.pem
#   Your key file has been saved at:
#   /etc/letsencrypt/live/<您的網址>/privkey.pem
#   Your cert will expire on 2019-12-25. To obtain a new or tweaked
#   version of this certificate in the future, simply run certbot
#   again. To non-interactively renew *all* of your certificates, run
#   "certbot renew"
# - If you like Certbot, please consider supporting our work by:

#   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
#   Donating to EFF:                    https://eff.org/donate-le

# 5. 安裝憑證
$ sudo vi /etc/nginx/sites-available/<您的網址>

# 6. Nginx 設定
server {
  listen 443 ssl;
  listen [::]:443 ssl;

  root /var/www/html;

  index index.html index.nginx-debian.html;

  server_name <您的網址>;

  location / {
          try_files $uri $uri/ =404;
  }

  ssl_certificate /etc/letsencrypt/live/<您的網址>/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/<您的網址>/privkey.pem;
}

# 7. 加入設定與重載
$ sudo ln -s /etc/nginx/sites-available/<您的網址> /etc/nginx/sites-enabled/<您的網址>
$ sudo nginx -t
$ sudo nginx -s reload
```

> 注意：沒有通過 ACME-Challenge（Automatic Certificate Management Environment） 還是可以產生憑證，只是後續無法自動產生/更新憑證。



##### 手動

手動的部分支援各種不同的參數，我們只會點出一些指令並紀錄提升訊息，希望能協助您有些基本的理解，查閱官方資料時能夠更容易掌握。

```bash
# 手動驗證可用 --preferred-challenges 參數指定 http 驗證或 dns 驗證，預設為 http。

# 1. 單純取得憑證
$ certbot certonly --manual

# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

# 2. 在其他機器上取得憑證 - macOS
$ brew install certbot
$ certbot certonly --manual --config-dir ~/your/path/letsencrypt --work-dir ~/your/path/letsencrypt --logs-dir ~/your/path/letsencrypt --preferred-challenges dns
```

執行指令後您可能會遭遇的執行流程紀錄如下：

```bash
# http
Saving debug log to /your/path/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Please enter in your domain name(s) (comma and/or space separated)  (Enter 'c'
to cancel):
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Create a file containing just this data:

Ad8wBqsP8mZoiUeN-XL_0v60vHFFjoaZTsGZ9gxGDpw.MUtTMf4SSLHIm0CLmqTzjYVPlBO7pDtO1tvDq90CHW8

And make it available on your web server at this URL:

http://<您的網址>/.well-known/acme-challenge/Ad8wBqsP8mZoiUeN-XL_0v60vHFFjoaZTsGZ9gxGDpw

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue

# dns
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.<您的網址> with the following value:

KzQ9pmKDC_iP9q4AWhGdGx9TezqoKy-eYo8PFAIYTG9

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```



### 自動更新

Certbot 套件內建自動排程可以在到期之前自動更新憑證。因為 Let's Encrypt 憑證只有 90 天，所以推薦使用這個功能。您可以利用下面的指令**測試**自動更新：

```bash
$ sudo certbot renew --dry-run

# 檢查自動更新
$ sudo systemctl status certbot.timer
```



### 手動更新

```bash
$ sudo certbot renew -v
```



### 移除

```bash
# 移除憑證
$ sudo certbot delete
# 刪除 /etc/nginx/<your site> SSL 設定
# 檢查 /etc/letsencrypt/archive, /etc/letsencrypt/live, /etc/letsencrypt/renewal

# 直接根據域名刪除
$ sudo certbot delete --cert-name <domain_name.com>

# 移除 certbot
$ sudo apt-get remove certbot python-certbot-nginx
```



### 查詢憑證日誌

```
https://crt.sh/?q=<您的網址>
```



## 總結

現在您可以自行判斷該使用哪種方式，而不是單純照文章執行指令而不知為何。一般來說如果只是測試主機您大可放心的使用全自動的方式，如果是正式主機最好還是使用 *webroot* 的方式自己設定 HTTP 伺服器。



### 補充

SSL 憑證有三個屬性：**驗證方式**、**加密強度**、**對應主旨名稱數量**。

##### 驗證方式

- Extended Validation 最高等級認證

- Organization validated 公司組織認證，例如：TWCA 在憑證的Organization(O)，含有組織資訊

  ```
  CN = github.com
  O = GitHub, Inc.
  L = San Francisco
  S = California
  C = US
  ```

- Domain Validation 最低等級認證，僅做網域認證。例如：**Let's encrypt**

##### 對應主旨名稱數量

> 完全限定域名（Fully qualified domain name），縮寫為**FQDN**，又譯為完全資格域名、完整領域名稱，絕對領域名稱（Absolute domain name）、 絕對域名，網域名稱的一種，能指定其在域名系統 (DNS) 樹狀圖下的一個確實位置。

一張憑證的主旨名稱（Subject Name）即對應的 FQDN：

- 一對一
- 不同網域一對多(SAN/UC)
- 同網域多對多（Wildcard/萬用憑證/多子網域通用憑證）



## 參考資源

* [官方安裝流程](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx)
* [官方取得憑證文件](https://certbot.eff.org/docs/using.html#getting-certificates-and-choosing-plugins)
* [HTTPS 简介及使用官方工具 Certbot 配置 Let’s Encrypt SSL 安全证书详细教程](https://linuxstory.org/deploy-lets-encrypt-ssl-certificate-with-certbot/)
* [安裝 Nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)
* [自動排程 renew](https://tmn.io/lets-encrypt-with-nginx-auto-renewal/)

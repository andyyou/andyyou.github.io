---
layout: post
title: 'SSL 相關筆記'
date: 2016-04-12 12:00:00
categories: System
tags: ssl
---

# 如何自己簽一張 SSL 憑證用於測試

### 概覽

下面簡易介紹了 SSL 是如何實作以及在整個流程中各個憑證所扮演的角色。

一般網頁傳輸是透過非加密的方式，意思是每一個人只要透過工具都可以存取，並且窺視所有的傳輸內容。
可以見得的，這可能造成一些問題。尤其是在安全和隱私方面，例如信用卡與銀行交易的資訊。
安全套接層協議(Secure Socket Layer)也就是 SSL 是用來加密這些伺服器與客戶端傳輸的資料。
簡單說就是保證兩個應用程式之間通訊的機密和完整性，也可驗證對方的身份。

<!--more-->

SSL 使用我們所知道的非對稱式加密，也就是我們常說的公鑰加密的方式(PKI，Public Key Cryptography)
公鑰加密，這種方式總共會有兩把鑰匙(兩個加密檔)，一把是 Public 一把 Private。任何加密的資料，只能透過對應鑰匙解密。
也就是說當一筆資料透過伺服器上的私鑰加密，那麼就只能透過對應的公鑰解開，反之亦然。
更白話的說就是鑰匙產方通常握有私鑰，將公鑰交給信任之對象，之後要產生的加解密都靠私鑰，而信用對象的加解密都用公鑰。

如果 SSL 已經使用了公鑰對傳輸資料加密，那為什麼我們還需要一張憑證？技術面的回答是這張憑證本來就不是必須的。
因為資料已經被加密，沒那麼簡單被第三方破解。然而憑證在溝通過程中扮演了一個關鍵的角色。
一張憑證需要一個被信任的憑證發行機構(CA)簽署(Signed)來確認憑證的擁有者聲稱的資訊是正確的。
少了被信任的簽章，資料仍然會被加密，但您所溝通的對象卻不一定跟你想的一樣。如果沒有憑證那麼偽裝式的攻擊會更容易發生。

具體來說 SSL 實際運作在第四層傳輸層 Transport Layer (TCP/UDP) 與第七層應用層之間。
主要的協定或者我們說`任務`分成 `SSL Handshake Protocol`, `SSL Change Cipher Spec Protocol`, `SSL Alert Protocol`
和 `SSL Record Protocol`

### SSL Handshake

在開始溝通之前要先互相確認身份，遵循的規則，加密演算法或密鑰交換演算法。

### SSL Change Cipher

用來變更雙方傳輸加解密的演算法與驗證的規格。

### SSL Alert

當發生錯誤時用來傳遞錯誤訊息。

### SSL Record Protocol

確認資料的完整性以及沒有被篡改。

### 流程

1 瀏覽器發送請求給 Server
  * 包含瀏覽器支援的加密演算法資料與 TLS/SSL 版本
2. Server 使用自己的私鑰加密 CA 憑證給瀏覽器，同時附加一把公鑰
  * 瀏覽器收到加密資料後嘗試使用網站的公鑰解密，這一步是為了確保憑證在傳輸過程中沒有被竄改
  * 驗證 CA 的訊息內容之後確認網站的真實性
  * Server 也丟出目前支援的加密演算法和 TLS/SSL 版本，決定兩者都支援的方式
3. 產生一組 Session Key (加密金鑰) 並用 Server 的公鑰加密
4. Server 用私鑰解開得知 Session Key
5. 使用雙方協議好的 Session Key 對接下來的資料進行加解密

![](http://www.qa-knowhow.com/wp-content/uploads/2014/11/SSL-handshake.png)

### 實作 - 步驟 1 產生一把私鑰

我們會使用 `openssl` 來產生`RSA加密演算法的私鑰`和 CSR(Certificate Signing Request)憑證請求檔
第一步我們需要先產生私鑰

~~~bash
$ openssl genrsa -out server.key 2048
~~~

### 實作 - 步驟 2 產生 CSR

CSR 憑證請求檔是要發給認證機構，簡單說其功用就是把你的個人資料填好附上私鑰交給認證機構來做簽證的動作

~~~bash
$ openssl req -new -key server.key -out server.csr
~~~

### 實作 - 步驟 3(選用) 移除 Key 的密碼

~~~bash
$ cp server.key server.key.org
$ openssl rsa -in server.key.org -out server.key
~~~

### 實作 - 步驟 4 產生自簽憑證

~~~bash
> openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
~~~

# Let’s Encrypt


# 參考資源

* [說明](http://www.qa-knowhow.com/?p=713)
* [Apache 使用](http://www.akadia.com/services/ssh_test_certificate.html)
* [创建并部署自签名的 SSL 证书到 Nginx](https://hinine.com/create-and-deploy-a-self-signed-ssl-certificate-to-nginx/)
* [SSL憑證設定教學](http://blog.faq-book.com/?p=92)
* [Let’s Encrypt 教學 - 中文](https://blog.dg-space.com/2015/12/setup-lets-encrypt-free-ssl-on-virtual-web-hosting.html)
* [Let’s Encrypt 教學](https://letsencrypt.org/getting-started/)
* [Let's Encrypt on OSX](https://community.letsencrypt.org/t/installing-and-configuring-letsencrypt-on-a-mac-os-x-client-server/8407)

---
title: "Debian/Ubuntu 安裝 RabbitMQ"
categories: System
tags: [ubuntu, tools, queue, rabbitmq]
---

# 檔案下載

* [官方教學](https://www.rabbitmq.com/install-debian.html)

<!--more-->

# 使用標準 Ubuntu 或 Debian 檔案庫

在官方的 Debian 和 Ubuntu 檔案庫中就有包含 `rabbitmq-server`。不過一般版本都比較舊。
大部分的時候我們可能要從 rabbitmq.com 這邊安裝比較新的版本。您可以從 [Debian package](http://packages.qa.debian.org/r/rabbitmq-server.html) 或 [Ubuntu package](https://launchpad.net/ubuntu/+source/rabbitmq-server) 來檢視版本和相關資料。

您也可以從上面的官方教學中下載 `dpkg` 或者使用下面的 APT 檔案庫。

# 對應支援的版本

下面列表是支援 RabbitMQ 3.6.3 的對應版本：

* Ubuntu 14.04 - 17.04
* Debian Jessie
* Debian Wheezy(須參考 [Wheezy backports repository](https://backports.debian.org/Instructions/) )

如果相依的套件都符合，該套件可能支援其他底層使用 Debian 的 Linux 系統，但我們不保證完成所有細節的測試。

# 安裝 Erlang/OTP

RabbitMQ 需要 Erlang/OTP 來執行。Erlang/OTP 套件在官方的 Debian 和 Ubuntu 檔案庫同樣支援，但一樣版本可能比較舊。
建議使用教新的版本，例如 19.3

|Erlang 版本|檔案庫及相關注意事項|
|20.x|[Erlang Solutions](https://packages.erlang-solutions.com/erlang/#tabs-debian) 從 RabbitMQ 3.6.11 開始支援，舊版不支援|
|19.x|[Erlang Solutions](https://packages.erlang-solutions.com/erlang/#tabs-debian), [Debian Stretch](https://packages.debian.org/search?suite=stretch&searchon=names&keywords=erlang), [Debian Jessie backports](https://packages.debian.org/search?suite=jessie-backports&searchon=names&keywords=erlang), Ubuntu Zesty(17.04)|
|18.x|[Erlang Solutions](https://packages.erlang-solutions.com/erlang/#tabs-debian), Ubuntu Yakkety(16.10), Ubuntu Xenial(16.04)|
|17.x|[Erlang Solutions](https://packages.erlang-solutions.com/erlang/#tabs-debian), [Debian Jessie](https://packages.debian.org/search?suite=jessie&searchon=names&keywords=erlang), [Debian Wheezy backports](https://packages.debian.org/search?suite=wheezy-backports&searchon=names&keywords=erlang)|


```bash
# 查詢 Ubuntu 版本名稱
$ lsb_release -c

# $ deb https://packages.erlang-solutions.com/ubuntu <版本名稱> contrib
$ deb https://packages.erlang-solutions.com/ubuntu xenial contrib

# 為 apt-secure 加入憑證
$ wget https://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc
$ sudo apt-key add erlang_solutions.asc

# 查詢 apt-get 安裝資訊（含線上可安裝版本)
$ apt-cache policy <package name>

# 安裝 Erlang
$ sudo apt-get update
$ sudo apt-get install erlang

# OR 完整安裝 Erlang （擇一）
$ sudo apt-get update
$ sudo apt-get install esl-erlang=1:19.3.6

# 查詢 Erlang 版本
$ erl -eval 'erlang:display(erlang:system_info(otp_release)), halt().'  -noshell
```

# 固定 Erlang 版本

使用 [apt package pinning](https://wiki.debian.org/AptPreferences) 可以避免不需要的更新，例如直接升到最新可能不支援當前的 RabbitMQ 版本。
下面的範例會固定 `esl-erlang` 套件到 19.3.6 然後 `erlang-*`是 19.3。

```
# /etc/apt/preferences.d/erlang
Package: erlang*
Pin: version 1:19.3-1
Pin-Priority: 1000

Package: esl-erlang
Pin: version 1:19.3.6
Pin-Priority: 1000
```

上面這隻檔案被放在 `/etc/apt/preferences.d/` 目錄下即 `/etc/apt/preferences.d/erlang`。

是否設定正確可以使用下面的指令驗證

```bash
$ sudo apt-cache policy
```

# 套件相依性

當我們使用 apt 來安裝時，所有的相依套件應該都要被自動安裝。如果您不是使用這種方式則可以從[backports 檔案庫](https://backports.debian.org/Instructions/)自行安裝。

# APT 安裝

```bash
$ echo 'deb http://www.rabbitmq.com/debian/ testing main' |
     sudo tee /etc/apt/sources.list.d/rabbitmq.list

$ wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc |
     sudo apt-key add -

$ sudo apt-get update
$ sudo apt-get install rabbitmq-server
```

# 新增設定檔

```bash
$ sudo su -
$ vi /etc/rabbitmq/rabbitmq.config
```

```
[
  {rabbit, [{loopback_users, []}]}
].
```

最後

```bash
# 啟動
$ service rabbitmq-server start
# OR
$ systemctl restart rabbitmq-server


# 檢查狀態
$ systemctl is-active rabbitmq-server
```

# 參考資源

* [Installing on Debian / Ubuntu](https://www.rabbitmq.com/install-debian.html)
* [erlang-solutions](https://www.erlang-solutions.com/resources/download.html)

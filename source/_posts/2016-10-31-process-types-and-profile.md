---
title: Heroku 運行類別、 Procfile、常用指令筆記
tags:
  - RoR
  - heroku
categories: Program
date: 2016-10-31 18:23:18
---


`Procfile` 是一種定義指令是否可以在 Heroku dynos（一種輕量化的容器，可以執行特定用戶設定的指令）上執行的機制。
它遵循著 Unix 的程序模型（process model）。這裡為了簡化概念，我們可以說 dyno 就是執行指令的一個個體。舉例來說一個 web dyno 就意味著執行一個 web server 的程式。希望這樣的說明可以理解這些特殊的名詞概念。

<!--more-->

您可以使用 Procfile 來宣告多種程序的類型（執行的程式種類），更簡單的來說一個執行中的程式就稱為一個程序（process）。例如一個標準的 Rails 程式至少需要兩種程序類型一個是 rack 相容的網站服務程序（Webrick, Unicorn）和 worker 程序（Delayed Job, Resque）。套用上面白話的說法就是一個執行中的伺服器服務程式，和一個排程的執行程式。

# Profile 命名與配置

一個 Procfile 檔案名稱就是 `Procfile`，不應該是其他名稱也不需要副檔名。舉例來說 `Procfile.txt` 就是錯誤的，如果是 txt 檔案就應該只是一個文字檔。

再者這個檔案應該被存放在專案的根目錄底下，如果放到其他目錄則不產生任何作用。

# 程序類型（process types）

一個 Procfile 可包含多個程序（process）的宣告，且每一行可以定義一個 process type 和指令。我們稱為程序類型（process type）其實就是一指定某一種 dyno 個體，隨後當這個 dyno 啟動後就會執行後面的指令。

好難懂！沒關係，舉例來說假設我們宣告了一個 web process type，意思就是靠設定後面的指令啟動該 dyno。這表示我們透過指令啟動一個 web server。直接看一下程式碼您比較清楚：

一個 Rails 的 process type 在 Procfile 中會如下設定

```rb
web: bundle exec rails server -p $PORT
```

而一個 Clojure 程式的 web process type 大概會長得像下面這個：

```rb
web: lein run -m demo.web $PORT
```

使用 Maven 產生的批次檔執行 Tomcat Java 的應用程式伺服器：

```rb
web: sh target/bin/webapp
```

對於許多應用程式來說，這些預設的設定就足夠了。不過某些複雜並且帶有的特定執行環境需求的程式，您也許會需要自己設定相關的 process type。舉例來說，Rails 應用程式也提供額外的 process type

```rb
worker: bundle exec rake jobs:work
```

# 定義程序類型（process type）

程序類型是透過 Procfile 這個檔案來定義的，並且此檔需要放在專案的跟目錄下，而其中每一行定義一個 process type。其格式如下：

```
<process type>: <command>
```

* process type - 英數字串，定義該程序類型例如：web, worker, urgentworker, clock 等。
* command - 用來啟動該程序的指令，例如：`rake jobs:work`

> 其中 `web` process type 較為特殊，只有這種類型可以接收 Heroku 路由送來的 HTTP 傳輸資訊，其他類型可以隨意命名。

> 另外最一開始提到 dyno 與 process type 其實這兩者之間的關係： process type 是原型類似於 class 的概念，而 dyno 則是實際初始化後的執行個體。


# 本地端開發（Heroku Local）

當我們在本地端開發和除錯的時候讓開發環境盡可能的與遠端環境相同是很重要的，這可以協助我們在部署之前找到問題。這時我們可以使用 Heroku Local 指令，在本地端模擬支援 Procfile 的機制。

這個時候只要先配置 Procfile 和 `.env` 環境並數的檔案並執行 `heroku local` 並可在開啟本地端模式。

Heroku Local 是一系列指令工具協助我們執行支援 Procfile 的應用程式。預設就會跟著 Heroku CLI 一起被安裝。同時它會從 `.env` 檔案讀取環境變數，該指令集使用 node-foreman 來完成這個任務。

### 啟動

```bash
$ heroku local
$ heroku local:start # 上面是縮寫
$ heroku local web # 啟動指定特定 process type，Ctrl + C 關閉
$ heroku local -f Procfile.test # 指定其他 Procfile
$ heroku local -e .env.test # 指定其他環境變數檔案
$ heroku local -p 7000 # 指定 port，預設使用 5000 port
$ heroku local -r # 當程序死掉時自動重啟
```

.env 的設定格式

```
S3_KEY=mykey
S3_SECRET=mysecret
```

> 注意 .env 應該要加到 .gitignore

### 複製 Heroku config 環境變數到 .env

```bash
$ heroku config:get CONFIG-VAR-NAME -s >> .env
```

# 部署到 Heroku

Procfile 並不是部署應用程式必須的東西，Heroku 會自動偵測使用的程式語言然後替它們建立對應的 web 程序類型。當然這邊推薦替您的程式建立一個明確的 Procfile。

另外，使用 `heroku ps` 可以提供我們明確正在執行的 dyno 數量與其狀態資訊。

```bash
heroku ps
=== web (Free): bin/rails server -p $PORT -e $RAILS_ENV (1)
web.1: idle 2016/10/31 12:32:49 +0800 (~ 4h ago)
```

`heroku logs` 可以彙整顯示關於所有 dyno 的 log。

# 操作（擴展） process type

Heroku 正常情況下會自動執行*一個* `web` dyno ，並且其他的 process type 預設並不會自動執行。
例如我們要啟動一個 worker 我們就需要自己執行（注意：Procfile 沒有註明的情況下）

```bash
$ heroku ps:scale worker=1
```

當然在隨著服務成長我們可能會需要增加 dyno 的規格

```bash
$ heroku ps:resize worker=standard-2x

# web=2 意味著水平擴展增加為 2 個叢集節點，standard-2x 為垂直擴展增加單節點的效能
$ heroku ps:scale web=2:standard-2x

# 單純水平擴展
$ heroku ps:scale web=4 worker=2
```

{% asset_img 2016-10-31-17-27-18.png %}

使用 `ps` 可以確認看看 process type 是否運行。

```bash
heroku ps
=== web: `bundle exec rails server -p $PORT`
web.1: up for 2m

=== worker: `env QUEUE=* bundle exec rake resque:work`
worker.1: up for 5s
```

由於預設 `heroku logs` 會把所有 process type 的日誌訊息都混在一起，有時候我們僅需要針對某一類型查看資訊

```bash
$ heroku logs --ps worker
```

# Heroku 指令彙整

```bash
$ heroku login # 登入
$ heroku version # 查看版本
$ heroku keys:add # 上傳金鑰
$ heroku create --app [app_name] # 建立 app
$ heroku open # 開啟網址
$ heroku rename # 重新命名 app 與對應網址
$ heroku ps:scale worker=2 # 啟動 worker dyno
$ heroku ps:stop worker # 停止所有 worker dynos
$ heroku ps:stop worker.2 # 停止指定的 worker dyno
$ heroku ps:restart # 重啟所有 dynos
$ heroku dyno:type web=3 # 設定指定 dyno 規格
$ heroku dyno:type standard-1x # 全部服務都設定為 standard-1x
$ heroku ps # 查看運行狀態
$ heroku logs --ps worker # 針對特定 dyno 列出 log
$ heroku pg:reset DATABASE # 清空資料庫

# Rails v5.x
$ heroku run rails console # 運行在遠端主機的 rails console
$ heroku run rails db:migrate
$ heroku run rails assets:precompile
```

# 資源

* [官方 dynos 管理教學文件](https://devcenter.heroku.com/articles/dynos)

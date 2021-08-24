---
title: Heroku 優化 Node 應用程式 - 併發
tags:
  - javascript
  - heroku
  - node
categories: Program
date: 2017-05-25 15:12:34
---


Node 對於擴展到不同規格的主機環境上其能力有限。首先是它是單執行緒，所以預設它不會自動使用額外的 CPU 核心。再者由於它基於 v8 引擎所以有記憶體上的限制概略是 1.5GB，所以也無法自動使用額外的記憶體。

因此 Nodejs 的應用程式需要多個進程來最大化可用的資源。這個過程我們稱為叢集，我們可以透過使用 Nodejs Cluster API 來完成這個功能。
我們可以直接在程式中使用 API 又或者使用基於這些 API 抽象化的函式庫。這裡我們將使用 [throng](https://github.com/hunterloftis/throng)。

使用叢集的方式，應用程式可以善加利用各種 dyno 的資源使效能得到提升。Heroku 的 Nodejs buildpack 也提供一些環境變數來協助我們。

<!--more-->

# 讓應用程式支援併發

我們建議所有的應用程式支援叢集。即使您不認為到您的程式需要執行超過一個進程，讓您的程式提供更大的彈性和控制權以利未來作調整。您可以參考下面的[範例](https://github.com/heroku-examples/node-concurrency/blob/master/server.js)。

首先，我們需要知道我們有多少我們的 dyno 有多少 process 可以使用

```js
var WORKERS = process.env.WEB_CONCURRENCY || 1
```

> 如果您想理解關於 process 和 thred 的關係可以參考[进程与线程的一个简单解释](http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)。

```bash
# 補充：查詢 Heroku dyno 的 CPU 資訊
$ heroku run grep -c processor /proc/cpuinfo
```

接著，我們定義一個 `start` function 這將是我們每個 process 開始執行時的進入點。

```js
function start () {

}
```

最後我們使用 `throng` 來使我們的程式支援叢集。我們設定 `throng` 的 lifetime 參數為 `Infinity`。假如一個 worker 死掉了它會自己再爬起來。
這意味著我們永遠有 `WORKERS` 在執行。

```js
throng({
  workers: WORKERS,
  lifetime: Infinty
}, start)
```

# 補充 - 使用 pm2 支援 Cluster

如果您已經完成了您的程式，或者說您想讓支援叢集的方式再彈性一點，我們可以使用 [pm2](http://pm2.keymetrics.io/)。

## 調整 package.json

以 Express 為例：

```json
"scripts": {
  "preinstall": "npm i -g pm2 && pm2 install pm2-logrotate",
  "start": "pm2 start --attach -i 0 ./bin/www && pm2 logs all"
}
```

## 驗證啟用叢集 

```
$ heroku ps:exec
Initializing feature... done
Adding the Heroku Exec buildpack to xeats

Run the following commands to redeploy your app, then Heroku Exec will be ready to use:
  git commit -m "Heroku Exec initialization" --allow-empty
  git push heroku master


$ git commit -m "Heroku Exec initialization" --allow-empty
$ git push heroku master
$ pm2 ls

# 如果要驗證平常是只啟用一個 Ap 可以把 `-i 0` 參數移除 
```

* [Heroku exec](https://devcenter.heroku.com/articles/heroku-exec)

# 範例 - 完整示範 Express 支援叢集

```bash
$ express demo
$ cd demo
$ npm i

# 編輯 package.json

$ git init
$ git add .
$ git ci -m 'Initial commit'
$ heroku login
$ heroku create
$ git push heroku master
$ heroku ps:exec
$ pm2 ls
$ grep -c processor /proc/cpuinfo # cpu 資訊

# 壓力測試(osx)
$ brew install siege
$ siege -c100 -t1M <your_url>
```

# 參考資源

* [Heroku 官方優化文件](https://devcenter.heroku.com/articles/node-concurrency#enabling-concurrency-in-your-app)
* [pm2](http://pm2.keymetrics.io/)
* [Heroku exec](https://devcenter.heroku.com/articles/heroku-exec)
* [Introducing Exec (beta) - Connect to a dyno via SSH](https://devcenter.heroku.com/changelog-items/1112)
* [更多 pm2 設定](http://pm2.keymetrics.io/docs/usage/use-pm2-with-cloud-providers/)
* [pm2 設定檔方式](http://pm2.keymetrics.io/docs/usage/cluster-mode/)
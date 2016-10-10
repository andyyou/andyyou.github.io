---
layout: post
title: "npm 最佳實作的小技巧"
date: 2016-10-04 12:00:00
categories: Program
tags: [javascript, nodejs, package manager]
---

# npm 最佳實作

`npm install` 是我們最常在 npm 中用到的指令，但其實它還提供了更多的功能。在這篇文章中我們將會學到 npm 如何在開發過程中協助我們 - 從建立專案到部署應用程式。

<!--more-->

# 0 了解您的 npm

在進入更深的主題之前，讓我們先來看看一些可以協助我查閱 npm 版本的指令

### npm versions

要取得 npm 指令的版本我們可以執行下面的指令

```bash
$ npm --version
```

要取得更詳細的訊息

```bash
$ npm version

{
  npm: '3.10.3',
  ares: '1.10.1-DEV',
  http_parser: '2.7.0',
  icu: '57.1',
  modules: '48',
  node: '6.6.0',
  openssl: '1.0.2h',
  uv: '1.9.1',
  v8: '5.1.281.83',
  zlib: '1.2.8'
}

$ npm show [library name] version # 顯示某 npm 線上最新版本
$ nvm ls # 顯示本地端套件資訊
$ nvm ls-remote # 顯示 node 可用版本
$ npm ls -g # Locally global
```

### npm help

跟大多數指令一樣，npm 也具備 `help` 功能。它會顯示相關指令的使用方式資訊，類似於 Linux 中的 man pages

```
$ npm help test
NAME
  npm-test - Test a package

SYNOPSIS
  npm test [-- <args>]
  aliases: t, tst

DESCRIPTION
  This runs a package's "test" script, if one was provided.
  To run tests as a condition of installation, set the npat config to true.
```

# 1 使用 npm init 建立新專案

當我們要開始一個新專案時，`npm init` 可以透過問答式的方式協助我們建立 `package.json` 當然如果我們想要全部使用預設值時可以使用

```bash
$ npm init --yes
$ npm init -y
```

有些預設值我們可以事先設定，像是專案作者這類資訊：

```bash
$ npm config set init.author.name [YOUR NAME]
$ npm config set init.author.email [YOUR EMAIL]
```

# 2 查詢 npm 套件

找到正確的套件有時候也是一個挑戰 - 在 npm 上有成千上萬的套件模組供我們挑選。通常我們依據一些經驗又或者根據網路上文章或社群的推薦。另外我們也可以參考 `npms.io` 這個網站，上面提供了關於品質，解決 Issue 平均時間，是否通過測試，甚至還有該模組是否相依了一些過期的套件等等。


# 3 查閱 npm 套件相關資訊

一旦我們選好了模組，舉例來說我們選了 `request` 這個套件。接著我們就需要查看相關文件，看看 Issue 是否會太多或有嚴重瑕疵。
注意，當您使用了越多的套件就表示越高的風險。對於產品來說我們應該要很清楚相依套件的狀況。

這個時候就可以透過指令直接造訪官網

```bash
$ npm home request
```

要檢查相關 Issues 或後續的發展計畫可以使用

```bash
$ npm bugs request
$ npm repo request # 套件 Git Repo
```

# 4 安裝與儲存設定

當我們確認好要使用的套件之後，就需要將其安裝到我們的專案

```bash
$ npm install request --save
$ npm install request -S
```

預設 npm 會在版好的部分使用 `^` 意味著下次你使用 `npm install` 時會安裝該主版號下最新版的版本，也就是主版號不變。當然我們也可以改變設定

```bash
$ npm config set save-prefix='~'
$ npm config set save-exact true # 精準版號
```

詳細版號規則可以參考[看懂 npm 語意化版本](http://calvert.logdown.com/posts/2014/08/20/npm-semantic-versioner)

# 5 鎖定相依套件

上面提到我們可以指定精準版號，但即使我們採用這種方式，您應該意識到大部分 npm 模組的作者並不會這麼做，因為他們會希望自動更新相依的套件。

這種情形在上線的產品很容易變成問題，可能會造成本地的專案和線上產品使用了不同的版本。為了解決這個問題我們可以使用 `npm shrinkwrap`，它會產生一個 `npm-shrinkwrap.json` 檔案，這隻檔案不只會記錄我們專案相依模組的版本，還會連模組相依的版本都準確的記錄下來。如此一來下次使用 `npm install` 就能確保模組的部分完全一致。

# 6 檢查過期的相依模組

為了找出那些需要更新的模組，npm 內建 `npm outdated` 指令

```bash
$ npm outdated
$ npm up [PACKAGE_NAME] -S # 更新
```

如果您想自動完成這個任務，您可以使用 [Greenkeeper](http://greenkeeper.io/)，一旦相依的模組有更新，它就會自動發送 pr 給您。

# 7 不要在產品中安裝 `devDependencies`

```bash
$ npm i --production
```

另外一個方式是指定環境變數

```bash
$ NODE_ENV=production npm install
```

# 8 保護您的專案與 Tokens

在 npm 登入帳號的狀況下，您的 npm token 會被放到 `.npmrc`。如果您發佈到 Github 的專案允許一些 `.` 開頭的隱藏擋格式那麼請注意關於 `.npmrc` 中的敏感的資訊不要一起發佈出去。

除了上述這一點，也要注意其他原始碼的部分是否包含一些敏感資訊。預設來說 npm 會遵循 `.gitignore` 的規則過濾掉一些檔案不發佈。然而當我們加入 `.npmignore` 時，就會以 `.npmignore` 為主，注意到兩隻檔案並不會彙整在一起。

# 9 開發模組

當我們在本地端開發模組的時候，我們通常在發布之前會需要測試安裝它們，這時就會使用到 `npm link`。
那麼這個 `npm link` 會做些什麼呢？它會建立一個軟連結，接著我們就可以在測試專案中使用 `npm link package-name`

```bash
# 建立測試模組連結
/projects/request $ npm link

# 安裝測試模組
/projects/test-project $ npm link request
```

# 資源參考

* [nodejs at scale - npm best practices](https://blog.risingstack.com/nodejs-at-scale-npm-best-practices)

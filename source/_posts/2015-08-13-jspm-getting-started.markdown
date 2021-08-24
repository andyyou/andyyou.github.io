---
layout: post
title: '第一次用 jspm 就上手'
date: 2015-08-13 15:00:00
categories: Tools
tags: [jspm, javascript, bundler, package manager, nodejs]
---

# 何謂 jspm ?

jspm (javascript package manager) 號稱是完全支援(無摩擦)瀏覽器載入的套件管理工具
<!--more-->
* jspm 也是一套套件管理工具，採用 `SystemJS 來處理模組載入` 當然內建也就支援動態的 ES6 模組載入
* 可以直接從 npm, Github 載入任何模組標準所寫得模組程式(ES6, AMD, CommonJS, global) 任何自訂的`registry`例如 npm 也可以透過 Registry API 來註冊連結
* 開發時，會將 ES6 檔案和編譯後的 plugins 分開載入
* 產品上線時，優化成一個 bundle ，也可透過指令將 bundle 分層或單獨執行

# 入門

* 安裝 jspm 指令
* 建立專案
* 初始化專案的設定檔
* 安裝來自任何 registry 的套件

#### 安裝 jspm 指令

~~~bash
$ npm install jspm -g
~~~

#### 建立專案

~~~bash
$ cd my-project
$ npm install jspm --save-dev
~~~

上面這個步驟不是必須的，但官方建議在專案安裝一個屬於專案自己的 jspm 版本，這可以確保當系統更新的時候專案的 jspm 版本並不會被影響，在目錄底下執行 `jspm -v` 可以顯示該專案的版本。

> 此時我們沒有 `package.json` 所以下 `--save-dev` 不會把設定加到 package.json 中。

#### 初始化專案的設定檔

~~~bash
$ jspm init
# Package.json file does not exist, create it? [yes]:
# Would you like jspm to prefix the jspm package.json properties under jspm? [yes]:
# Enter server baseURL (public folder path) [.]:
# Enter jspm packages folder [./jspm_packages]:
# Enter config file path [./config.js]:
# Configuration file config.js doesn't exist, create it? [yes]:
# Enter client baseURL (public folder URL) [/]:
# Which ES6 transpiler would you like to use, Traceur or Babel? [traceur]:
~~~

這個指令會協助我們設定 `package.json` 和 `config.js`(jspm 預設)。注意到當目錄為空的時候 `jspm init` 會試圖自動幫我們處理下面這些項目

* baseURL: 這個設定指的是相對於在 server 上的 public folder 通常是 `/`，也就是 package.json 應該放置的專案根目錄，或者說當網址為根的時候該如何對應到此專案目錄。
* jspm packages 目錄: jspm 會將其他相依的檔案安裝在這裡。
* config 檔案路徑: 這個是 jspm 的設定檔也應該要跟 package.json 一樣放在專案的根目錄
* client baseURL: 這個 URL 設定瀏覽器如何存取被託管在 server 上的目錄
* transpiler: 設定使用的 compile to js language(ES6+ to ES5)。可以在任何時間透過 `jspm dl-loader --babel` 來修改這個選項。也可以直接在 jspm 的設定檔中透過 `babelOptions` 或 `traceurOptions` 修改。

如果你需要重新設定這些屬性，可以直接修改 `package.json` 接著執行 `jspm install` 或 `jspm init` 來更新

而如果想要重新發動 jspm 的詢問來更新檔案可以執行 `jspm init -p`

#### 安裝來自任何 registry 的套件 例如: Github, npm ...

首先 `registry` 的意義是一個可以註冊的地方或空間。對應到程式開發領域的話指的就是像 Github, npm, gem Nuget, apt, yum 。開發者可以把自己的程式(函式庫)註冊並上傳發佈的`套件管理站點或系統`。

透過下面的指令可以從任何 registry 安裝

~~~bash
$ jspm install npm:lodash-node
$ jspm install github:components/jquery
$ jspm install jquery
$ jspm install myname=npm:underscore
$ jspm install [registry]:[package name]
~~~

多個套件安裝可以在同一個指令用空白隔開，上面的例子我們看到了無論是 npm 或 github 都可以透過這種方式安裝。

大部份的 npm 套件安裝不需要再加上額外的設定，這是因為 npm 站點使用專案設定規範適用於所有 Node 和 npm-style 的程式碼，也因此相容 jspm

Github 的套件或說程式碼就可能需要對 jspm 補些設定

所有安裝項目的設定會存放在 package.json，而 jspm_packages 目錄和設定檔可以透過執行 `jspm install` 全部重建，所有相依的第三方元件仍然是透過 package.json 來紀錄設定。

> 原則上你不該把這些第三方的程式碼一起加入版控。

簡單來說，jspm 是透過 system.js 來處理載入模組這件事。而載入模組這件事為什麼需要處理，是因為在 js 的世界裡存在較多的標準，ES6+ 的標準又還沒普及，所以通用的載入器需求就產生了，白話文即不管你是用 CommonJS 或 AMD 甚至 ES6 寫的東西都要能夠被載入，但 jspm 並不是唯一可以處理這個問題的工具。

一個 nodejs 專案是透過 package.json 來管理相依的函式庫或套件，而 jspm 主要也是透過 package.json 來組織其設定，它會根據解析 package.json 的結果在 config.js 裡加上 system.js 以及自己需要的設定。
也因此 config.js 只要透過 `jspm install` 就可以根據 package.json 的資料重建設定檔。

#### 搞懂 npm, jspm, webpack 的使用上的差異

我們依照模組標準開發出一個函式庫模組並丟到 npm 給大家用，npm 這個詞意義上又分成 npm-cli 指令和 npm 這個集合套件的站點，所以這邊我們提的 npm 指的是指令的部分，例如 `npm install` 這樣的指令來處理安裝，移除，管理的指令。

而 webpack 或 browserfiy 它們是處理模組封裝和載入的工具，意思是把你寫的 js 打包，大略的實作行為就是你在程式中用 require 來載入其他檔案最後輸出一隻打包的 js，所以安裝的部分還是用 npm ，而載入則由 webpack 這類的工具負責。

那 jspm 呢？第一個不同點就是除了你可以透過 npm 安裝專案專屬 jspm 外(用來鎖定版本)，其他套件或函式庫你都是用 `jspm install [library name]` 的方式來安裝。
那設定呢？基本上上面就提到了; 如果你是採用 node 或 npm-style 的函式庫或模組是不用設定的，因為 jspm 會幫你把設定加到 config.js 裡面。

下面整理 jspm v.s npm + webpack 的流程順序

```
step 1. jspm 下載模組並安裝 (使用 npm 下載並安裝)
step 2. 在 package.json 中紀錄設定，而程式碼安裝到 `jspm_packages` (npm 一樣把設定記錄到 package.json 檔案裝到 node_modules)
step 3. 分析 package.json 後在 config.js 產生設定 (使用 webpack 自己手動寫設定)
step 4. 撰寫程式碼，載入並應用模組
step 5. 執行 jspm bundle (執行 webpack ./main.js ./bundle.js)
```

#### 實作

現在我們就可以開始在程式中撰寫載入的部分了。為了更加明白其運作。這次的練習會包含

* 建立一個 jspm 專案
* 先試著寫 ES6 語法來測試
* 安裝 jsx, react
* 撰寫一個簡單的 React Component
* bundle & 優化

~~~bash
$ mkdir jspm_new_project
> 建立一個空的目錄

$ jspm init
> 初始化專案，注意 transpiler 選 babel

$ jspm install jquery
> 試著安裝 jQuery, 我們先試著用基本的 jQuery 搭配 ES6 寫點範例

$ mkdir -p assets/js/lib
$ mkdir -p assets/js/components
> 模擬真實狀況組織 js 目錄
~~~

* 建立 `assets/js/lib/Cat.js`

~~~js
export default class Cat {
  constructor (name) {
    this.name = name;
  }

  yell () {
    var result = `${this.name}: meow`;
    console.log(result);
    return result;
  }
}
~~~

* 建立 `assets/js/main.js` 來載入我們的小貓類別

~~~js
import $ from 'jquery';
import Cat from './lib/Cat';

var cat = new Cat("Mily");

$(function() {
  $(".animal").text(cat.yell());
});
~~~

* 建立 `index.html` 來使用 main.js，伺服器部分先用簡單的 `python -m SimpleHTTPServer` 起一個 server 來測試
當然如果你會其他方式例如使用 express 或者架設 apache 等也可以

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>jspm sample</title>
  <script src="jspm_packages/system.js"></script>
  <script src="config.js"></script>
</head>
<body>

  <span class="animal"></span>
  <script>
    System.import('assets/js/main');
  </script>
</body>
</html>
~~~

* 開啟瀏覽器輸入 `http://localhost:8000/`

第一階段我們驗證了 ES6 語法和 jQuery 運作相當正常，接著我們要來試著寫一個 React Component 試試。
首先要注意的是 jspm 預設只會載入 `.js` 副檔名，而且只處理純 js 即使我們知道 Babel 能處理 JSX 但如果你直接照著寫是會出錯誤的。
要解決這個問題我們需要 [jsx loader plugin](https://github.com/floatdrop/plugin-jsx) 因此我們需要先安裝 jsx 和 react

~~~bash
$ jspm install jsx react
~~~

* 在 `assets/js/components/` 建立一個 `Dog.jsx`

~~~js
import React from 'react';

class Dog extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <div>{this.props.name}: bark!!!</div>
    );
  }
}

export default Dog;
~~~

回到 main.js 要注意上面說過 jspm 預設不處理 jsx ，而且就算你把副檔名換成 .js 還是會出錯，正確的用法是只要有用到 jsx 語法的檔案副檔名都應該是 `.jsx`
接著在 import 的時候要記得副檔名和 `!` 如下面範例 `import Dog from './components/Dog.jsx!'`。

對了！因為 main.js 也要用 jsx 語法所以記得將其副檔名也換掉喔，`index.html` 裡面的 System.import 也要加入特殊語法。

~~~js
import $ from 'jquery';
import React from 'react';

import Cat from './lib/Cat';
import Dog from './components/Dog.jsx!'

var cat = new Cat("Mily");

$(function() {
  $(".animal").text(cat.yell());
});

React.render(<Dog name="Wally"/>, document.getElementById("dog"));
/* 警告: Warning: `require("react").render` is deprecated. Please use `require("react-dom").render` instead.*/
~~~

* 修改 `index.html`

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>jspm sample</title>
  <script src="jspm_packages/system.js"></script>
  <script src="config.js"></script>
</head>
<body>

  <span class="animal"></span>
  <span id="dog"></span>
  <script>
    System.import('assets/js/main.jsx!');
  </script>
</body>
</html>
~~~

至此我們已經玩完一輪基本的使用方式了 - [完整範例](https://github.com/andyyou/jspm-sample)
上面示範的方式是在 html 頁面加上 SystemJS loader 和 config.js

基本上我們需要啟動一個 server 才能存取，不過您也可以透過下面這種方式直接存取檔案

* Mac 的 Chrome 使用者可以執行 `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --allow-file-access-from-files &> /dev/null &`
* Firefox 先切換到 about:config, 搜尋 `security.fileuri.strict_origin_policy` 把值改成 `false`

這個時候 `config.js` 的 baseURL 就要設成 `.` 這樣才能正常運作

#### 最佳化

~~~bash
$ jspm bundle assets/js/main --inject
# 一般 JS

$ jspm bundle assets/js/main.jsx! --inject
# JSX 的指令格式
~~~

之後 system.js 就會自動取得這個 bundle.js

另外還可以使用 `jspm bundle-sfx assets/js/main` 來建立一個獨立的 bundle script 如此一來就不用在 html 中使用 `config.js` 和 `system.js`

# 結論
總結來說 jspm 試圖從安裝函式庫到封裝全部一起處理，且盡可能不需要你去設定。整體用起來到還是蠻簡潔的。
要用類似 webpack-dev-server 的可以參考 [jspm-server](https://github.com/geelen/jspm-server)。似乎 webpack 有的東西都在慢慢補齊中。
不過以 React 的立場來看 webpack 的資源和參考還是多一點。

因為前端工具更新的速度實在太快了這邊想對目前用過的東西做點簡單的總結

#### Grunt gulp 本質上是任務管理工具

  - 所以我們可以組織打包，測試，任何想做的事等於幫我們建立一堆指令

#### webpack, browserify, RequireJs 處理模組與相依性管理

  - 所以只要在設定檔設定哪個檔案類型該怎麼處理(封裝編譯)，它就會幫你處理`編譯模組`以及`模組載入`
  - Grunt, gulp 和 webpack 這類工具處理問題的角度不同，一個是讓你組織各種任務，一個是針對檔案類型處理如何編譯，壓縮，封裝，載入

#### npm 本質上為模組(函式庫)的管理工具 + 簡易的任務工具

  - 安裝，移除，顯示資訊。
  - 透過 package.json 執行寫在裡面的 script

#### jspm 概念上為 npm + webpack(但目前沒有 webpack 功能那麼全面)

  - 因為除了是一個模組(函式庫)的管理工具外還使用 System.js 處理模組載入的部分

# 資源

* [jspm](https://github.com/jspm/jspm-cli/wiki/Getting-Started)
* [React with JSPM](http://blog.developsuperpowers.com/react-with-jspm/)

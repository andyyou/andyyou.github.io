---
layout: post
title: '第一次用 jspm 就上手'
date: 2015-08-13 15:00:00
categories: js
---

# 何謂 jspm ?

jspm (javascript package manager) 易於瀏覽器支援的套件管理器

* jspm 也是一套套件管理工具，採用 `SystemJS 通用模組載入` 這個 loader ，當然內建也就支援動態的 ES6 模組載入
* 可以直接從 npm, Github 載入任何模組標準所寫得模組程式(ES6, AMD, CommonJS, global) 任何自訂的`模組套件收集註冊管理系統`例如 npm 也可以透過 Registry API 來註冊連結
* 針對開發時期，會將 ES6 檔案和編譯後的 plugins 分開載入模組
* 針對產品狀態，優化壓縮成一個 bundle ，也可透過指令將 bundle 分層或獨自運行

# 入門

#### 安裝 jspm 指令

{% highlight bash %}
$ npm install jspm -g
{% endhighlight %}

#### 建立專案

{% highlight bash %}
$ cd my-project
$ npm install jspm --save-dev
{% endhighlight %}

上面這個步驟不是必須的，但官方建議在專案安裝一個屬於專案自己的 jspm 版本，這可以確保當系統更新的時候專案的 jspm 版本並不會被影響，在目錄底下執行 `jspm -v` 可以顯示該專案的版本。

> 此時我們沒有 `package.json` 所以下 `--save-dev` 不會把設定加到 package.json 中。

#### 初始化專案的設定檔

{% highlight bash %}
$ jspm init
Package.json file does not exist, create it? [yes]: 
Would you like jspm to prefix the jspm package.json properties under jspm? [yes]: 
Enter server baseURL (public folder path) [.]: 
Enter jspm packages folder [./jspm_packages]: 
Enter config file path [./config.js]: 
Configuration file config.js doesn't exist, create it? [yes]:
Enter client baseURL (public folder URL) [/]: 
Which ES6 transpiler would you like to use, Traceur or Babel? [traceur]:
{% endhighlight %}

這個指令會協助我們設定 `package.json` 和 `config.js`(jspm 預設)。注意到當目錄為空的時候 `jspm init` 會試圖自動幫我們處理下面這些項目

* baseURL: 這個設定指的是對於在 server 上 public folder 通常是 `/`，也就是 package.json 應該放置的專案根目錄，或者說當網址為根的時候該如何對應到此專案目錄。
* jspm packages 目錄: jspm 會將其他相依的檔案安裝在這裡。
* config 檔案路徑: 這個是 jspm 的設定檔也應該要跟 package.json 一樣放在專案的根目錄
* client baseURL: 這個 URL 設定瀏覽器如何存取被託管在 server 上的目錄
* transpiler: 設定使用的 compile to js language(ES6+ to ES5)。可以在任何時間透過 `jspm dl-loader --babel` 來修改這個選項。也可以直接在 jspm 的設定檔中透過 `babelOptions` 或 `traceurOptions` 修改。

如果你需要重新設定這些屬性，可以直接修改 `package.json` 接著執行 `jspm install` 或 `jspm init` 來更新

而如果想要重新發動 jspm 的詢問來更新檔案可以執行 `jspm init -p`

#### 從任何 Registry 安裝任何套件例如: Github, npm ...

首先 registry 的意義是一個可以註冊的地方，空間。對應到程式開發領域的話指的就是像 Github, npm, gem Nuget, apt, yum 這些開發者可以把自己的東西註冊並上傳發佈的`套件管理網站或系統`

透過下面的指令可以從任何 registry 安裝

{% highlight bash %}
$ jspm install npm:lodash-node
$ jspm install github:components/jquery
$ jspm install jquery
$ jspm install myname=npm:underscore
$ jspm install [registry]:[package name]
{% endhighlight %}

多個套件安裝可以在同一個指令用空白隔開，上面的例子我們看到了無論是 npm 或 github 都可以透過這種方式安裝。

大部份的 npm 套件安裝不需要再加上額外的設定，這是因為 npm 的站點套用的轉換規則，其設定檔適用於所有 Node 和 npm-style 的程式碼，也因此相容 jspm

Github 的套件或說程式碼就可能需要對 jspm 加上設定

所有安裝的項目設定會存放在 package.json，因此 jspm_packages 目錄和設定檔可以透過執行 `jspm install` 全部重建，所有相依的第三方元件都還是透過 package.json 來管理，原則上你不該把這些第三方的程式碼一起加入版控

簡單來說，jspm 是透過 system.js 來處理載入模組這件事

而載入模組這件事為什麼需要處理，是因為在 js 的世界裡太多標準，ES6+ 的標準又還沒普及，所以通用的載入器需求就產生了，但 jspm 並不是唯一可以處理這個問題的工具。

jspm 架構在 nodejs 專案之上，意思是 nodejs 專案是透過 package.json 來管理相依的函式庫或套件，而 jspm 主要也是透過 package.json 來組織其設定，它會在 config.js 上面根據解析 package.json 的結果在 config.js 裡加上 system.js 的設定。
也因此 config.js 只要透過 `jspm install` 就可以根據 package.json 的資料重建設定檔

#### 搞懂 npm, jspm, webpack 的使用上的差異
我們依照模組標準開發出一個函式庫模組並丟到 npm 給大家用，npm 這個詞意義上又分成 npm-cli 指令和 npm 這個收集套件的站點，所以這邊我們提的 npm 指的是指令的部分，即 `npm install` 等等來處理安裝，移除，管理的指令。

而 webpack 或 browserfiy 它們是封裝和載入工具，意思是把你寫的 js 打包，大略的實作行為就是你在程式中用 require 來載入其他檔案最後輸出一隻打包的 js，所以安裝的部分還是用 npm ，而載入則由 webpack 這類的工具負責。

那 jspm 呢？第一個不同點就是除了你可以透過 npm 安裝專案專屬 jspm 外(用來鎖定版本)，其他套件或函式庫你都是用 `jspm install [library name]` 的方式來安裝。
那設定呢？基本上上面就提到了; 如果你是採用 node 或 npm-style 的函式庫或模組是不用設定的，因為 jspm 會幫你把設定加到 config.js 裡面。

下面整理 jspm v.s npm + webpack 的流程順序

~~~
registry > jspm downlaod and install (using npm to install and management)
         > records in package.json and install file in node_modules (npm has same action)
         > add configuration to config.js for System.js and others of jspm (using webpack and write configuration of webpack.config.js)
         > [write your code, laod modules]
         > excute jspm bundle that according config.js, use System.js to load modules (using webpack command-line to bundle)
~~~

#### 實作

現在我們就可以開始在程式中使用其 loader 的功能來撰寫載入的部分了。為了更加明白其運作。這次的練習會如下

* 建立一個 jspm 專案
* 先試著寫 ES6 語法來測試
* 安裝 jsx, react 
* 撰寫一個簡單的 React Component
* bundle & 優化

{% highlight bash %}
$ mkdir jspm_new_project
> 建立一個空的目錄

$ jspm init
> 初始化專案，注意 transpiler 選 babel

$ jspm install jquery
> 試著安裝 jQuery, 我們先試著用基本的 jQuery 搭配 ES6 寫點範例

$ mkdir -p assets/js/lib
$ mkdir -p assets/js/components
> 模擬真實狀況組織 js 目錄

{% endhighlight %}

組織好目錄之後我們先建立 `assets/js/lib/Cat.js`

{% highlight js %}
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
{% endhighlight %} 

再來建立 `assets/js/main.js` 來載入我們的小貓類別

{% highlight js %}
import $ from 'jquery';
import Cat from './lib/Cat';

var cat = new Cat("Mily");

$(function() {
  $(".animal").text(cat.yell());
});
{% endhighlight %}

接著我們就試著開一個 `index.html` ，先用簡單的 `python -m SimpleHTTPServer` 起一個 server 來測試
當然如果你會其他方式例如使用 express 或者架設 apache 等也可以

{% highlight html %}
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
{% endhighlight%}

開啟瀏覽器輸入 `http://localhost:8000/` 第一階段我們驗證了 ES6 語法和 jQuery 運作相當正常
接著我們要來試著寫一個 React Component 試試
首先要注意的是 jspm 預設只會載入 `.js` 副檔名，而且只處理純 JS 即使我們知道 Babel 能處理 JSX 但還是會出錯誤。
要解決這個問題我們需要 [jsx loader plugin](https://github.com/floatdrop/plugin-jsx) 因此我們需要先安裝 jsx 和 react

{% highlight bash %}
$ jspm install jsx react
{% endhighlight %} 

在 `assets/js/components/` 建立一個 `Dog.jsx`

{% highlight js %}
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
{% endhighlight %}

回到 main.js 要注意上面說過 jspm 預設不處理 jsx ，而且就算你把副檔名換成 .js 還是會出錯，正確的用法是只要有用到 jsx 語法的檔案副檔名都應該是 `.jsx`
接著在 import 的時候要記得副檔名和 `!` 如下面範例 `import Dog from './components/Dog.jsx!'`，對了！因為 main.js 也要用 jsx 語法所以記得將其副檔名也換掉喔，`index.html` 裡面的 System.import 也是。

{% highlight js %}
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
{% endhighlight %}

修改 `index.html`

{% highlight html %}
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
{% endhighlight %}

至此我們已經玩完一輪基本的使用方式了 - [完整範例](https://github.com/andyyou/jspm-sample)
上面示範的方式是在 html 頁面加上 SystemJS loader 和 config.js

基本上我們需要啟動一個 server 才能存取，不過您也可以透過下面這種方式直接存取檔案

* Mac 的 Chrome 使用者可以執行 `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --allow-file-access-from-files &> /dev/null &`
* Firefox 先切換到 about:config, 搜尋 `security.fileuri.strict_origin_policy` 把值改成 `false`

這個時候 `config.js` 的 baseURL 就要設成 `.` 這樣才能正常運作

#### 最佳化

{% highlight bash %}
$ jspm bundle assets/js/main --inject
# 一般 JS

$ jspm bundle assets/js/main.jsx! --inject
# JSX 的指令格式
{% endhighlight %}

之後 system.js 就會自動取得這個 bundle.js 

另外還可以使用 `jspm bundle-sfx assets/js/main` 來建立一個獨立的 bundle script 如此一來就不用在 html 中使用 `config.js` 和 `system.js`

# 結論
總結來說 jspm 試圖從安裝函式庫到封裝全部一起處理，且盡可能不需要你去設定。整體用起來到還是蠻簡潔的。
要用類似 webpack-dev-server 的可以參考 [jspm-server](https://github.com/geelen/jspm-server)。似乎 webpack 有的東西都在慢慢補齊中。
不過以 React 的立場來看 webpack 的資源和參考還是多一點。

# 資源

* [jspm](https://github.com/jspm/jspm-cli/wiki/Getting-Started)
* [React with JSPM](http://blog.developsuperpowers.com/react-with-jspm/)



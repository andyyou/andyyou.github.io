---
layout: post
title: 'vue-loader 與 vue-cli 實作函式庫'
date: 2016-06-05 12:00:00
categories: js
---

# 從介紹打造函式庫開始

我們在開發時幾乎不斷的在使用`函式庫`，一個函式庫是針對特定用途打包好的程式碼，我們可以直接安裝並在開發專案中使用。
彙整那些我們常用的功能好使我們不必重造輪子。而透過套件管理我們則不必在反覆的複製貼上。

不過上面提到的`打包好的程式碼`說得並不精確，一個函式庫有可能是一隻檔案或者一些檔案組織在一起。
而函式庫本身的程式碼和您的專案是分開的，函式庫的程式應該要被單獨維護，以讓您可以重複使用在不同專案上。
另外也應該要允許我們針對不同的需求有些不同的設定去配合專案。把它比喻成一個 USB 裝置，允許相同規範的 port 溝通，但透過不同的設定可以提供不同的功能舉例像是麥克風，鍵盤，滑鼠等。

在第一小節我們解釋如何建置函式庫。雖然大部分的內容在其他語言也是通用的不過在這篇文章我們只專注在 Javascript

### 為什麼要打造自己的函式庫?

因為函式庫可以反覆使用，非常方便就算您不寫 open source 也是能靠這種機制去維護。如此一來關於那些已經寫過的功能下次要用的時候就不用反覆複製貼上。
就只要安裝就好。另外就是將程式中的片段程式碼拆分組織有助於日後維護。

簡單來說任何程式碼能夠完成特定目的並且會反覆使用就應該要被抽出來做成 Libaray。一個耳熟能詳的例子就是 jQuery 雖然它的 API 已經不是單單的簡化 DOM 操作，不過在一開始它的目標的確是讓跨瀏覽器之間的 DOM 操作更加簡單一致。
如果是 open source 專案，當很多開發者使用時，通常也會有更多人協助開發，發現問題，提出解決 issues。不管怎樣這對函式庫以及使用的專案來說都是好事。當然一個熱門的 open source 專案還可以產生很多效益，像是增加曝光度被公司挖掘等等。

### 規模與目標

首先在開始撰寫程式碼之前，我們應該先定義清楚關於我們的函式庫要解決什麼問題。在開發的時候也應該要時時想著這個函式庫設計要容易上手使用，盡量不要一口氣包山包海。

一開始應該先問問自己這些，要解決什麼問題? 該如何處理問題? 是要全部自己重寫? 還是可以搭配別人的函式庫使用?
不管這個函式庫的規模大小如何，我們都應該試著寫下規劃，列出我們想要的功能，去除一些不是必要的東西，概念就類似打造 MVP 產品一樣。
當這些最基本的功能完成就可以發行第一版，接著在循序的補上新功能。

### API 設計

個人習慣在設計函式庫的時候從使用者角度出發。本質上這個步驟是在勾勒我們這個函式庫的輪廓，重點就是讓使用者更方便他們才會選擇我們的函式庫，同時我們也需要思考各方面能自訂修改的部份。最後最好能夠自己使用這樣你也會發現有哪些功能是實務上會需要的。盡可能保持函式庫單純與彈性。

下面是一個實作的範例我們的目標是一個 `User-Agent` 標頭字串的函式庫就是就是用來表示瀏覽者使用的環境資訊例如 Firefox 的 `Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:46.0) Gecko/20100101 Firefox/46.0`。這個函式庫用起來大概會如下

```js
var userAgent = new UserAgent

// 假設我們的 UserAgent 是 CropBrowser/1.2 (X11; Linux; zh-tw)
var application = new UserAgent.Product('Mozilla', '5.0')
application.setComment('Macintosh', 'Intel Mac OS X 10.11', 'rv:46.0')
userAgent.addProduct(application)

// 加入 engine 資訊
var engine = new UserAgent.Product('Gecko', '20100101')
engine.setComment('Do you like')
userAgent.addProduct(engine)

userAgent.toString() // Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:46.0) Gecko/20100101 (Do you like)
```

從使用方法出發，依據功能的複雜度思考一下架構。這邊的功能大致上就是依照這些資訊幫我們把 `;` `/` `()` 這些規則處理好。

### 彈性與客製化

具備彈性會使我們的函式庫不只侷限在單一情形，不過這也是最難抉擇的地方，你需要先定義好什麼能做什麼不能以及主要的目的。最好的例子就是 chart.js 和 d3.js 比較，兩者都是處理資料視覺化圖形的函式庫不過 chart.js 相對簡單，內建就可以快速產出各式各樣的圖表。而如果你需要完整的控制圖表那麼 d3 才是比較好的選擇，所以定義好一個問題的邊界是很重要的。

接著關於使用者使用函式庫的方式有非常多種包含`設定`，`public methods`，`callback`，`events` 。通常設定會用在初始化階段，不過也有些函式庫允許我們在執行時修改選項。一般來說函式庫會提供一些 `options` 參數設定，並且修改這些設定除了變動本身的值外不應該在更改其他資料。

```js
var userAgent = new UserAgent({
  commentSeparator: ';'
})

uesrAgent.setOption('commentSeparator', '-')
userAgent.commentSeparator = '-'
```

`methods` 可以用來操作物件實例，舉例來說從物件中檢索資料(getters) 還有更新資料(setters)

```js
var userAgent = new UserAgent
userAgent.getComments() // 取得所有 comment
userAgent.shuffleProducts() // 亂數排序 Products
```

`callback` 會透過 `public methods` 傳入，比較常見的狀況就是處理一些非同步的任務。

```js
var userAgent = new UserAgent
userAgent.asynTask(function after() {
  // Run code after async task is done
  after()
})
```

事件可以辦到很多事情。有點類似 callback 通常用在需要在特定時間點發動的行為。

```js
var userAgent = new UserAgent

// 綁定一個事件讓使用者自訂判斷是否要加入該項目
userAgent.on('product.add', function onProductAdd (e, product) {
  var shouldAdd = product.toString().length < 5
  return shouldAdd
})
```

在某些情況，也許你想要讓使用者擴充我們的函式庫。為了這個目的我們可以加入一些方法(public methods)讓使用者自己擴充。例如 Angular 模組 `angular.module('MyModule')` 和 jQuery 的 `fn(jQuery, fn.MyPlugin)` 又或者就直接讓使用者存取函式庫本身

```js
(function UserAgentUpcase(UserAgent) {
  UserAgent.prototype.upcase = function () {
    return this.toString().toUpperCase()
  }
})(UserAgent);

var userAgent = new UserAgent
userAgent.upcase()
```

同樣的這個方式也允許使用者覆寫函式庫的方法

```js
(function UserAgentUpcase(UserAgent) {
  var _toString = UserAgent.prototype.toString

  UserAgent.prototype.toString = function () {
    return _toString.call(this).toUpperCase();
  }
})(UserAgent)

var userAgent = new UserAgent;
userAgent.toString();
```

在賦予存取的方式下，我們就比較不能規範使用者該怎麼擴充，有時候這可能會產生問題。因此為了確保函式庫正常運作通常我們應該寫下文件並制定一些慣例和規則。

### 測試

在設計函式庫的 API 和功能時最好的做法就是使用 TDD，簡單的說就是在寫 code 之前先把每個方法預期的結果寫下來，又或者使用 BDD 以完整的功能和行為來撰寫測試案例。不管哪一種方式確保功能正常對函式庫來說挺重要的。

這邊列出幾篇關於測試的文章

* [Unit Test Your JavaScript Using Mocha and Chai](http://www.sitepoint.com/unit-test-javascript-mocha-chai/)
* [Testing JavaScript with Jasmine, Travis, and Karma](http://www.sitepoint.com/testing-javascript-jasmine-travis-karma/)

### 模組相容性

無論你是否使用模組系統。想使用您的函式庫的開發者可能有在使用，所以會希望這個函式庫可以相容他選擇的 module loader。不過我們該選擇 `CommonJS`, `RequireJS`, `AMD`, 等等哪一種呢?

實際上我們並不需要去選擇，您可以採用 `Universal Module Defintion` (UMD) 是一種支援多種模組系統的策略。您可以在網路上找到多種風格的範例，又或者直接參考[UMD Github Repository](https://github.com/umdjs/umd)或它提供的工具使我們的函式庫兼容 UMD。

如果你已經在使用 ES2015 我強烈建議使用 Babel 搭配 [Babel 的 UMD 套件](http://babeljs.io/docs/plugins/transform-es2015-modules-umd/)去編譯，如此一來不只使用 ES2015 還可以自動幫我們的函式庫支援所有類型的模組系統。

### 說明文件

原則上每個專案都需要有完整的文件，不過因為維護文件也是很累人的工作再加上不斷的開發修改程式常常後面就忘記了。

* **基本資訊**

正常來說文件應該要介紹一些基本資訊像是專案名稱啦，和一些功能特性的描述。如此一來使用者就可以知道這個函式庫是做什麼的以及使用的一些優點。當然如果能夠補充一些關於這個函式庫的一些限制和動機還有一些發展規劃就更好了。

* **API 手冊, 教學和範例**

為了讓使用者更快速能上手最好能提供 API 手冊, 教學和範例但是通常這又有一堆事情要做，因此像是 JSDoc 這類的 `inline document` 就能夠提供一些幫助。大致上的用法就是在程式中寫下特定格式的註解然後透過其他工具去解析輸出成文件。

* **Meta-tasks 開發指令**

有一些使用者也許會希望可以改寫我們的程式碼，不管是協助開發或者客製一個私人的版本。通常在文件中也會包含一些關於開發指令像是建置或者測試。

* **協作**

當您發起 open source 的時候可能會有一些熱心的開發者參與協作此時就需要一些關於協作的規範等資訊。定義好這些規則日後當有人參與的時候也比較好維護。

* **授權**

[Choose A License](http://choosealicense.com/)是一個不錯的網站可以協助我們選擇 License

### 封裝與版控

一個好的函式庫不可缺少的就是版控。假設有一天你決定來個大改版，當使用者不想跟進的時候也可以繼續維持舊版。現在最多人採用的命版規則便是[語意化版本](http://semver.org/) 這套規則由三個數字組成各自有不同的意義 `major` `minor` `patch` 特別注意的是主版號為零（0.y.z）的軟體處於開發初始階段，一切都可能隨時被改變。這樣的公共API 不應該被視為穩定版。

* **使用 git 管理版號**

如果您使用 git 來做版控您可以透過 `tag` 的方式來打上版號又或者可以透過 `npm version patch`

```bash
$ git tag -a v1.2.0 -m 'Libaray v1.2.0'

$ npm version patch
```

* **發佈**

幾乎每一種語言都會有套件管理工具，在 Javascript 的世界裡通常使用的是 `npm` 如果您還不熟悉可以閱讀[入門教學](https://www.sitepoint.com/beginners-guide-node-package-manager/)。預設 npm 套件是公開的不過您可以[修改](https://www.sitepoint.com/beginners-guide-node-package-manager/)或者不要發佈。

要發佈套件需要 `package.json` 您可以自己編寫檔案或者 `npm init` 裡面的版號跟 git 應該要保持一致。最後一切都準備好了就可以 `npm publish` 就可以推上 npm。另外您可能也會採用 Bower 不過因為這幾年大家紛紛都轉向 npm 也因此下面僅列出一些教學連結。

* [bower 入門](http://www.sitepoint.com/package-management-for-the-browser-with-bower/)
* [Set up a private repository](https://github.com/bower/registry#installation)


# 參考資源

* [Design and build your own javascript library](https://www.sitepoint.com/design-and-build-your-own-javascript-library)
* [How to publish es6 npm modules](https://booker.codes/how-to-build-and-publish-es6-npm-modules-today-with-babel/)
* [Build javascript library using webpack and ES2015](http://krasimirtsonev.com/blog/article/javascript-library-starter-using-webpack-es6)

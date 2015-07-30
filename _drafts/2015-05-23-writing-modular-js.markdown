---
layout: post
title: "分別用 AMD, CommonJS 與 ES Harmony 開發 JavaScript 模組"
date: 2015-05-23 12:00:00
categories: Javascript
---

# 模組化
當我們說到一個應用程式是有模組化的，通常代表的是這個應用程式是由一些高度解耦，程式碼本身依據功能性分開的模組組合而成。再白話一點是比喻例如汽車是由引擎，變速箱等等這些模組組合而成的。
也許您曾經聽過應用程式如果盡可能的移除`相依性`具有`鬆散耦合`或`低耦合度`的特質會比較容易維護。

像剛剛汽車的例子，因為汽車具有模組化的特性所以當油門灌下去你發現速度沒到位你就可以很直覺的開始只檢查有關係的模組; `變速箱`, `引擎`, 還是`傳動`有問題。
當模組化被有效率的被執行後，我們就可以很簡單的明白系統裡每個部分的功能是什麼，怎麼影響其他部分，邊界在哪裡，你總不會告訴我引擎發不動要去檢查輪胎吧?

回到主題，不像一些傳統的程式語言，現階段的 JavaScript(ECMA-262) 不提供模組機制像是 C# 的 using, Java 的 import 這種簡單明瞭，有組織性的模組機制。

主要是因為在過去 JavaScript 的定位讓它不是非常需要這種機制，直到這幾年 JS 能開發的東西越來越複雜，這種需求就變得顯而易見。

因為沒有內建的模組機制與標準所以開發者們只好自己採用一些設計模式來達到這個目標。在前端的世界，大部份的解決方案還是採用命名空間的方式在 DOM 裡把所有模組的程式碼串在一起。雖然採用命名空間但還是有機會發生命名重複的問題。

對於相依函式庫的管理，除了自己寫或求助於第三方工具，我們沒有其他簡潔的方式處理。

而原生的解決方案將會在 ES6 到來，好消息是撰寫模組化的 JS 將是前所未有的簡單，而且感謝許多社群的貢獻，您可以透過一些第三方工具讓我們今天就可以開始用這種方式。

# 前言
很難在討論 AMD 和 CommonJS 的時候不提到一個大家都不願意提起的問題 - Script Loaders。

Script Loader? 不懂! 你肯定知道這東西，HTML 的 `<script>` 標籤就是一種載入器。或許你也聽過 `RequireJS`。其文件開宗名義就提到它採用一種不同於 `<script>` 的機制使其比起標籤的方式更快更好。

載入腳本(Script)為的是在今時今日讓 JavaScript 可以實踐模組化。是因為用 `<script>` 處理不了一些問題才會出現這些工具和標準，舉例來說當使用過多 `<script>` 來載入 JS 的時候瀏覽器會先停止渲染等待 JS 載入，過多的檔案將導致等待時間變長造成問題，又或者函式庫之間存在的相依性需要按照順序下載等等，而既然要模組化，那麼勢必得把程式給分開來。也因此使用一個通用的 Script Loader 很不幸是必須的。

因應需求開始出現許多 loader 來處理模組載入的問題，它們通常採用 AMD 或 CommonJS 標準。
但是完整說明像 RequireJS 等這些工具並不在這篇文章的範圍內。當然我會試著把事情交代清楚。

另外當我們開始採用模組化時，通常也會被建議 - 要把模組部署出去成為 Production 的時候最好使用優化工具例如: 把它們散落的檔案合併成一個等等。好了先記住這些問題讓我們開始吧

# AMD
AMD(Asynchronous Module Definition) 標準總體的目標是提供一種 JavaScript 模組化的機制，從名字就可以看出它是屬於非同步的方式。它基於 Dojo 使用 `XHR` + `eval()` 的經驗而誕生。這種標準的支持者希望未來任何的解決方案都可以避免掉過去這種解決方案的缺點和痛苦。

大略說就是避免掉使用 AJAX 去拉一段程式交給 eval 執行，這種方式的缺點。

AMD 模組標準本身就是`定義模組`的建議，讓模組與相依的模組或函式庫可以非同步載入。這麼做有些不同面向的優點，其中包含非同步，高度靈活性(使用 Module ID 和其定義載入模組的方式解決程式高度耦合的問題)。很多開發者喜歡這種方式，也認為 ES Harmony 模組用這樣的方式相對可靠，緩和一點，不用一口氣改太多東西。

AMD 起初也是 CommonJS 標準中的其中一個草案，但因為無法取得共識，所以後續的開發則移往 amdjs 社群。

採用 AMD 的包含 Dojo(1.7), MooTools(2.0), 甚至 jQuery(1.7)。雖然偶而會看到 CommonJS AMD 標準這個術語，但因為不是所有 CommonJS 標準的參與者都願意支持它，所以我們最好還是把 AMD 跟 CommonJS 分開單純叫他 AMD 或者支援非同步模組載入

# AMD 模組化入門
這裏有兩個關鍵的概念你必須要先了解那就是 `define` 方法用來構建模組的定義以及 `require` 方法用來處理相依模組載入的設定。
語法如下

{% highlight js %}
define(
  'module_id', /* 選擇性使用 */
  ['dependence_1', 'dependence_2'], /* 選擇性使用 - 相依的模組或函式庫 */
  function() { /* 用來產生模組物件實例或直接回傳物件的 */
    ...
  } 
)
{% endhighlight %}

如您在註解裡所見 `module id` 不是必須的參數，一般來說只有當你使用非 AMD 標準的合併工具時才需要定義。當我們沒有定義這個參數的時候我們就稱這個模組為匿名模組。

使用匿名模組時切記 DRY 的原則，盡量讓每一個模組的功能單純一點以避免檔名或程式碼重複。保持程式碼的通用性，如此一來比較容易移植到其他地方而不需要修改程式碼本身或修改 ID。

`define` 方法的第一個參數即 `module id` 型別為一個字串，用來設定模組的 id 主要的用意就是用來取代原本的 URL，如果沒有設定，module id 預設為載入器設定載入檔案的名稱。

什麼意思? 以 RequireJS 為範例來說明: 
1. 開始使用 RequireJS 的時候從 HTML 透過 `<script>` 先載入 `require.js` 
2. 通常會指定一個 `data-main` ，然後透過該檔案，開始載入其他 Script。
3. 假設我們的 `main.js` 路徑為 `scripts/main.js`，此時會用 `scripts` 目錄當作根目錄。
4. 當我們 `require(['abc'])` 的時候，組成路徑 `scripts/abc` 開始去抓檔案的時候 RequireJS 會幫我們補 `.js`。 

module id 被用來當作模組唯一的識別名稱，在大多數的實作中這東西其實不需要使用，以保持彈性。還是聽不懂，先把它當作唯一的識別路徑就好。

除此之外，像是在 AMD 中使用 CommonJS 模組，或使用 [r.js](https://github.com/jrburke/r.js/) 這類的工具，可以把 AMD 的程式碼直接用在 CommonJS 的環境。

回到 `define()` 方法的語法，第二個參數是相依性的設定，使用陣列來定義這個模組所需要參考的其他模組或函式庫。

第三個參數可以想成是為您的模組初始化的函式。下面示範一個簡單的宣告範例

## 使用 define

{% highlight js %}
// 這邊的 module id(myModule) 只是用來 demo

define('myModule',
  ['foo', 'bar'], // 陣列，定義載入的模組或函式庫
  // 相依模組（foo 和 bar）會依序被對應帶入 function 參數
  function(foo, bar) {
    /
    var myModule = {
      doStuff: function() {
        console.log("Yay! Stuff");
      }
    }

    return myModule;
  }
);

{% endhighlight %}

另外一方面 `require` 通常在`最上層`的 JavaScript 檔案中被用來載入程式碼，例如

## 使用 require

{% highlight js %}
require(['foo', 'bar'], function( foo, bar ) {
  foo.doSomething();
})
{% endhighlight %}

## 動態載入

{% highlight js %}
define(function(require) {
  var isReady = false, foobar;
  require(['foo', 'bar'], function(foo, bar) {
    isReady = true;
    foobar = foo() + bar();
  });

  return {
    isReady: isReady,
    foobar: foobar
  };
});
{% endhighlight %}

## AMD 模組套件 - 載入其他檔案類型

{% highlight js %}
define(
  ['./templates', 'text!./template.md', 'css!./template.css'],
  function(templates, template) {
    console.log(templates);
  }
)
{% endhighlight %}

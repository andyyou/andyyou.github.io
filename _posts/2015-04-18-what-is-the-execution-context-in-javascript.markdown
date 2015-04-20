---
layout: post
title: '理解 Javascript 執行環境'
date: 2015-04-18 05:30:00
categories: Javascript
---

# Javascript 中的執行環境與堆疊

在這篇筆記中我將會深入的探討 JS 底層中的一些觀念，其中最重要的就是`執行環境`(Execution Context)。當您閱讀完這篇文章後您可能會比較清楚關於直譯器的運作方式，明白為什麼有些 `函式` `變數` 可以在他們被宣告之前就拿來使用，以及這些值是怎麼決定的。

# 什麼是執行環境？
我們說當 JS 開始執行的時候，這段程式碼必須被執行在下面三種環境之一。
* 全域 Global：預設當您程式開始執行時的環境
* 函式：當我們進入一個函式 function 時的環境，也就是開始跑函式內部程式碼的時候
* Eval：把一串字串，當作指令來執行時的環境
也就是說一段 JS 程式碼只能被歸類為上面這三種狀態或類型。

讓我們直接來看看程式碼

{% highlight js %}
// Global context, JS 最外層的程式碼部分屬於全域
var greeting = "Hi";

function person() {
  // 從大括號開始到結束進入另外一個執行環境
  var _firstName = "andy";
  var _lastName = "you";

  function firstName() {
    // 另外一個執行環境
    return _firstName;
  }

  function lastName() {
    // 執行環境
    return _lastName;
  }

  alert(greeting + firstName() + ' ' + lastName());

}
{% endhighlight %}

上面這段範例沒什麼特別的，我們就是有了一個全域的執行環境即 `global context` ，和 3 個 `function context`，唯一稍微要注意的是 `global context`只會有一個。其他執行環境都可以存取全域的東西。

當然您可以有多個 `function context` 每一個 function 執行的時候就會建立一個新的 context ，OK！不管講執行環境或者 context 都好抽象，那我們就先把他們當作是一個 context 物件，那這個 context 物件講白了就是表示一個環境，一個範圍，一個狀態。它會建立一個範圍一個自己特有的領域，任何在 function 裡面宣告的變數或其他東西都不能被外面直接存取。

如果這樣還不能理解，那我們換個角度來想這件事，你把 context 當成是一張記錄表格，當我開始在 global 執行程式碼的時候。
任何變數，function 都會被記載 `global 表` 上，但是當執行到 function 內部的時候，此時會在開出另外一張 `function 表` 負責記錄 function 內部的變數等等。

不過我個人認為 `執行環境` 是最貼切的翻譯，當我在全域這個環境時我能夠取得的變數和進到另外一個 function 環境時可能會有不一樣的狀況。

# 執行環境的堆疊
對於執行環境有了初步的概念之後我們還得知道 - 瀏覽器的 JS 直譯器通常是單執行緒的，意味著一次只能夠做一件事。
也就是說當一個事件被執行的時候其他的任務，事件等等就會被丟到執行佇列中。這個東西我們就叫做`執行堆疊`

我們已經知道當 JS 開始跑的時候一開始會進入 `global 執行環境`，如果您在 global 環境中呼叫了一個 `function A` (即： `A();`)，這個時候就會建立新的 `執行環境` 然後這個新的執行環境會被放到`執行堆疊`的最上面，同樣的如果你現在在 function A 裡面又叫了 function B 那麼就又會在建立一個`執行環境`一樣放到`執行堆疊`的最上方，瀏覽器永遠會先處理堆疊上最上面的執行環境，一旦執行環境裡面的任務都執行完了那它就會被移掉換下一個

{% highlight js %}
(function foo(i) {
  if (i === 3) {
    return;
  } else {
    foo(++i);
  }
}(0));
{% endhighlight %}

![]({{ site.url }}/assets/tutorials/es1.gif)

這段程式碼簡單的呼叫自己三次每一次把參數加一，每當 `foo` 被呼叫的時候新的 `執行環境` 就被建立，然後當 `執行環境` 裡面的程式跑完的時候，就從堆疊中把 `執行環境` 拿掉，把控制權交還給上一個環境一直到回到 global 為止。

#### 關於執行環境有 5 個重點要牢記在心

* 單執行緒
* 同步執行
* 只有一個 global context
* function context 沒有限制
* 就算是自己呼叫自己只要 call function 就會建立執行環境

# 詳解執行環境
所以我們現在知道了每一次 call function 的時候就會建立一個新的執行環境，然而在 JS 直譯器內部每次呼叫一個執行環境都會有兩個階段

1. `建立階段` 當 function 被呼叫了但在開始執行內部程式碼之前
  * 建立一個 `scope chain` 作用域鍊
  * 建立變數，function，和參數
  * 設定 `this` 的值
2. `執行階段` 
  * 賦值，設定 function 的參考和解譯執行程式碼

概念上我們可以把一個 `執行環境` 想像成一個物件，那麼這個物件大概會有三個屬性如下

{% highlight js %}
executionContextObject = {
  scopeChain: { /* 變數物件 + 所有父代執行環境物件的變數物件*/},
  variableObject: {/* 函式的參數/引數，內部的變數和函式*/ },
  this: {}
}
{% endhighlight %}

> Variable Object 變數物件：根據 ECMA-262 的說明，每一個執行環境會有一個與相關連的變數物件，這個物件負責記錄執行環境中定義的變數和函式。

# Activation / Variable Object [AO/VO]
這一個執行環境物件在 function 被調用的時候建立，不過在實際的 function 被執行之前，這就是上面提到的階段 1  - 建立階段。在這個階段直譯器會建立 `executionObject` ，透過掃描函式傳入的參數，內部的函式宣告，變數宣告。結果會被記錄在`executionObject` 的 `變數物件 variableObject` 中。

#### 這裏我們大致模擬直譯器是如何執行的流程
1. 尋找呼叫 function 的程式碼
2. 在執行 function 之前建立 `執行環境`
3. 進入 `建立階段` 
  - 初始化 `scope chain`
  - 建立 `variable object`：
    * 建立 `arguments object` 檢查執行環境的參數，初始化參數的名稱，值以及建立參考
    * 掃描 function 的宣告
      + 根據找到的每一個 function 在 `variable object` 建立，在這邊其實就是建立 function `名稱`在記憶體中的參考指標
      + 如果 function 名稱已經存在那麼指標就會被覆寫
    * 掃描執行環境裡的變數 
      + 每一個變數的宣告都會被加入 `variable object` 的屬性中，並且初始化為 `undefined`，注意在這個階段並不會`賦值`
      + 如果變數名稱存在就略過，繼續處理下一個變數
  - 判斷決定 this 的值
4. 執行階段
  - 執行程式碼，賦值，一行一行跑

{% highlight js %}
function foo(i) {
  var a = 'hello';
  var b = function B() {

  };
  function c() {

  }
}
foo(22);
{% endhighlight %}

此時在建立階段我們就會得到如下的範例

{% highlight js %}
fooExecutionContext = {
  scopeChain: { ... },
  variableObject: {
    arguments: {
      0: 22,
      length: 1
    },
    i: 22,
    c: pointer to function c()
    a: undefined,
    b: undefined
  },
  this: { ... }
}
{% endhighlight %}

如您所見，在建立階段處理關於定義宣告的部分，此時並不會賦值，所以 function b 並沒有被參考。不過參數是唯一的例外，此時參數的值已經被建立。一旦建立階段完成，剩下的流程就是開始執行階段，當執行階段完成的時候`執行環境`就會如下

{% highlight js %}
fooExecutionContext = {
  scopeChain: { ... },
  variableObject: {
    arguments: {
      0: 22,
      length: 1
    },
    i: 22,
    c: pointer to function c()
    a: 'hello',
    b: pointer to function B()
  },
  this: { ... }
}
{% endhighlight %}

# 變數宣告`提升`
您可以找到很多關於定義 Javascript `hoisting` 的資料，他們通常會解釋這就是一種把宣告提升到其所在區域內頂端的行為，然而這樣並沒有解釋到細節，為什麼會發生這件事，不過呢剛剛您已經知道了關於整個直譯器解意的流程，現在您可以很清楚的明白為什麼會這樣了。

{% highlight js %}
(function () {
  console.log("foo: " + typeof foo); // function pointer
  console.log("bar: " + typeof bar); // undefined

  var foo = 'hello',
      bar = function() {
        return 'world';
      };
  function foo() {
    return 'hello';
  }
}());
{% endhighlight %}

現在我們可以回答關於上面這段程式碼的一些問題

* 為什麼我們在宣告之前可以存取 foo
  - 如果我們看看 `建立階段` 的流程我們可以知道變數在這個時期早就被建立了
* Foo 被宣告 2 次，為什麼 foo 是 function 而不是 undefined 或 string？
  - 即使 foo 宣告了2次，我們知道在建立階段 function 會先被建立。因此變數已經存在了在這個階段 string 不會被賦予 foo
  - 因此在真正執行 function 之前 foo 是會先被建立，等他真正跑完執行階段的時候 foo 才會被覆寫成 'hello'

* 為什麼 bar 是 undefined ?
  - bar 就只是一個變數，在這個階段並還沒賦值所以就是 undefined

# 總結
希望現在您可以更清楚關於 Javascript 如何運行您的程式碼，瞭解執行環境，堆疊可以讓您更清楚您的程式碼在不同狀態下取到的值，如此一來相信您在組織 JS 的時候會有更好的寫法。


# 參考
[0](http://www.cnblogs.com/leoo2sk/archive/2010/12/19/ecmascript-scope.html#comment_tip)
[1](http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/)
[2](http://davidshariff.com/blog/javascript-scope-chain-and-closures/)

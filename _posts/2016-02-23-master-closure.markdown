---
layout: post
title: '[譯] 透過重新實作來學習參透閉包'
date: 2016-02-23 12:00:00
categories: javascript
---

原文出處: [連結](http://www.sitepoint.com/quick-tip-master-closures-by-reimplementing-them-from-scratch/?utm_medium=email&utm_campaign=SitePoint%20JavaScript%20Newsletter%20%20February%2022%202016&utm_content=SitePoint%20JavaScript%20Newsletter%20%20February%2022%202016+Version+A+CID_b7922c27f8cf7091c1aec57998c5ddf6&utm_source=CampaignMonitor%20SitePoint&utm_term=Master%20Closures%20by%20Reimplementing%20Them%20from%20Scratch)

話說網路上有很多文章在探討`閉包`(Closures)時大多都是簡單的帶過。大多的都將閉包的定義濃縮成一句簡單的解釋，那就是`一個閉包是一個函數能夠保留其建立時的執行環境`。不過到底是怎麼保留的？

另外為什麼一個閉包可以一直使用區域變數，即便這些變數在該 `scope` 內已經不存在了？

為了解開閉包的神秘面紗，我們將要假裝 Javascript 沒有閉包這東西而且也不能夠用嵌套 `function` 來重新實作閉包。這麼做我們將會發現閉包真實的本質是什麼以及在底層到底是怎麼運作的。

為了這個練習我們同時也需要假裝 Javascript 本身具備了另一個不存在的功能。那就是一個原始的物件當它如果被當成 function 調用的時候是可以執行的。
你可能已經在其他語言中看過這個功能，在 `Python` 中你可以定義一個 `__call__` 方法，在 `PHP` 則有一個特殊的方法叫 `__invoke`
這些`方法(Method)`會在當物件被當作 function 調用時執行。如果我們假裝 Javascript 也有這個功能，我們可能需要這麼實作:

{% highlight js %}
let o = {
  n: 42,
  __call__() {
    return this.n;
  }
};

// 當我們把物件當作 function 一樣調用時
o(); // 42, 當然現在你會得到 `TypeError: o is not a function` 的錯誤

// 譯者註: 之後遇到這種呼叫的情況，請使用 o.__call__()
{% endhighlight %}

這邊我們得到一個普通的物件，我們假裝我們可以把它當做 `function` 來呼叫，然後當我們這個做的同時其實我們是執行一個特殊的方法 `__call__` 如果你真的要實作記得用 `o.__call__()`。

> 譯者註: 注意! 呼叫 `可調用物件` 例如上面的 `o()` 都要換成 `o.__call__()` 假如您想實作的時候。

現在讓我們先來看看一個簡單的閉包範例。

{% highlight js %}
function f() {
  // 下面這個變數是 f() 的區域變數
  // 通常，當我們離開 f 的 scope 時，這個變數 n 就應該要被回收了
  let n = 42;

  // 嵌套的 function 參考了 n
  function g() {
    return n;
  }

  return g;
}

// 讓我們透過 f() 來建立一個 g 函數
let g = f();

// 理論上這個變數 n 在 f() 執行完畢之後就應該要立即被回收，對吧？
// 畢竟 f 已經執行完畢了，而且我們也離開了該 scope
// 那為什麼 g 可以繼續參考一個已經被釋放的變數呢？
g(); // 42
{% endhighlight %}

外層的 function `f` 有一個區域變數，然後裡面的 function `g` 參考 `f` 的區域變數。

接著我們把內層的 g 回傳指派給 `f` scope 外的變數。但我們好奇的是如果 `f` 執行完畢被釋放了，那為什麼 g 仍然可以取得已被釋放的 f 的區域變數呢？

這個的魔法便是 - 一個閉包不僅僅只是一個 function。它是一個物件，具有建構子和私有資料。然後我們可以它當作 function 來使用。
那如果 Javascript 沒有閉包這種用法，我們必須自己實作它呢？這就是我們接下來要看到的。

{% highlight js %}
class G {
  constructor(n) {
    this._n = n
  }

  __call__() {
    return this._n;
  }
}

function f() {
  let n = 42;

  // 這就是一個閉包
  // 這個內層的 function 其實不只是一個 function
  // 它其實是一個可以被調用的物件，然後我們傳入 n 到它的建構子
  let g = new G(n);


  return g;
}

// 透過呼叫 f() 取得一個可以被調用的物件 g
let g = f();

// 現在就算原來從 f 拿到的區域變數 n 被回收了也沒關係
// 可被調用的物件 g 實際上是參考自己私有的資料
g(); // 42
{% endhighlight %}

> 如果您曾看過 ECMAScript 規範，可能會對`實際上是參考自己私有的資料`這句話產生一些疑問，先別急著否定。這邊不過是試著用另外一個較淺的角度解釋。

這邊我們把內部的 `function g` 用一個 `G class` 的實例物件(即 new 出來的物件) 取代，然後我們透過把 f 的區域變數 n 傳進 G 的建構子，藉此將變數儲存在新的實例物件私有的資料中。最終我們可以取得 f 的區域變數(n)。

OK! 各位觀眾這就是一個閉包的行為。閉包就是一個可調用的物件，可以把透過建構子把傳入的參數保留在私有的空間中。

# 更深入的問題？

聰明的讀者已經發現還有一些行為我們還沒解釋清楚或者說我們的模擬實作是有漏洞的。讓我們來觀察其他的閉包範例

{% highlight js %}
function f() {
  let n = 42;

  // 內部函數取得變數 n
  function get() {
    return n;
  }

  // 另外一個內部函數也同時存取 n
  function next() {
    return n++;
  }

  return { get, next };
}

let o = f();
o.get(); // 42
o.next();
o.get(); // 43
{% endhighlight %}

在這個範例中，我們得到兩個閉包同時參考變數 `n` 。其中一個函數的操作變數會影響另外一個變數取得得值。
但如果 Javascript 沒有閉包，單靠我們上面的實作行為將不會一樣。

{% highlight js %}
class Get {
  constructor(n) {
    this._n = n;
  }

  __call__() {
    return this._n;
  }
}

class Next {
  constructor(n) {
    this._n = n;
  }

  __call__() {
    this._n++;
  }
}

function f() {
  let n = 42;

  // 這邊的閉包我們一樣換成可調用的物件
  // 它們可以將參數傳入建構子，進而將值保留起來
  let get = new Get(n);
  let next = new Next(n);

  return { get, next };
}

let o = f();
o.get(); // 42
o.next();
o.get(); // 42
{% endhighlight %}

跟上面一樣，我們取代了內部 function `get` 和 `next` 的部分改成使用物件。它們是透過將值保留在物件內部進而取得 `f` 的區域變數，每一個物件具有自己私有的資料。同時我們也注意到其中一個`可調用物件` 操作 n 並不會影響另外一個。這是因為它們是傳 n 的`值 value`而不是`傳址 reference`。白話文就是複製了一分資料。並不是操作變數本身。


為了要解釋為什麼 Javascript 的閉包會參考到相同的 n 即記憶體位置是一樣的。我們需要解釋變數本身。在底層，Javascript 的區域變數跟我們從其他語言理解的觀念並不相同，它們是負責`動態分配與計算參考(reference)`的物件的`屬性`，稱為 `LexicalEnvironment` 物件。Javascript 的閉包其實會有一個參考指向到整個 `執行環境`, `上下文`, `Context` 的 LexicalEnvironment 物件，而不是特定的變數。

> 如果您對於 scope 與 context 還不是很了解強烈建議您觀賞[這篇](http://www.cnblogs.com/leoo2sk/archive/2010/12/19/ecmascript-scope.html#comment_tip)

讓我們來修改我們的`可調用物件`讓其可以取得一個 `lexical environment` 而不是 `n` 。

{% highlight js %}
class Get {
  constructor(lexicalEnvironment) {
    this._lexicalEnvironment = lexicalEnvironment;
  }

  __call__() {
    return this._lexicalEnvironment.n;
  }
}

class Next {
  constructor(lexicalEnvironment) {
    this._lexicalEnvironment = lexicalEnvironment;
  }

  __call__() {
    this._lexicalEnvironment.n++;
  }
}

function f() {
  let lexicalEnvironment = {
    n: 42
  }

  // 現在這個可調用變數是透過一個參考 lexical environment 來改變 n
  // 所以現在變更的是同一個 n 了
  let get = new Get(lexicalEnvironment);
  let next = new Next(lexicalEnvironment);
  return { get, next }
}

// 現在我們實作的物件行為跟 javascript 一致了
// 還是請注意如果您要時作，記得 o.get() 要換成 o.get.__call__() 喔
let o = f();
o.get(); // 42
o.next();
o.get(); // 43
{% endhighlight %}

上面實作我們將區域變數 n 換成 lexicalEnvironment 物件，然後具有一個屬性 n 。
這時 `Get` 和 `Next` 的物件實例所存取的便是同一個參考(reference)即 `lexical environment` 物件。
所以現在修改的就是相同的地方了。基本上這就是一個閉包的行為。

# 結論

閉包是一個物件而且當它們是函數時我們可以直接調用。而事實上任何一個 Javascript 中的函數都是一個可被調用的物件也稱作 `function object` 或者 `functor` 當它們被執行或者說被實例化時會帶有一個私有的 lexical environment 物件。而想要更了解關於這個物件的看官們可以參考[Lexical environment](http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-2-lexical-environments-ecmascript-implementation/#lexical-environment)。
在 Javascript 不是 function 創造閉包，function 本身就是一個閉包。

> 老實說譯者本身還是比較喜歡理解 context 與 variable object 的說明，接著用 `一個閉包是一個函數能夠保留其建立時的執行環境` 這句話來記憶。
不過原作者從這個角度來解釋的確是可以概略的理解整個運作機制，希望這篇文章能讓你有所收穫。

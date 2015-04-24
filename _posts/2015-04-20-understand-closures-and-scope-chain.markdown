---
layout: post
title: '參透Javascript閉包與Scope Chain'
date: 2015-04-20 14:00:00
categories: Javascript F2E
---

從[上一篇](http://andyyou.github.io/javascript/2015/04/18/what-is-the-execution-context-in-javascript.html)文章中我們知道了每一個 function 有一個對應的`執行環境` 其中包含著一個由在該範圍內所有的變數，function 參考，參數定義所組成的 `variable object`(變數物件 VO)。

另外每一個執行環境中還定義了一個 `scope chain` 屬性，它就是一個記錄包含 `自己的 VO` + `所有上層執行環境的 VO` 的集合。

如果我們用普通話的來說：一個`環境`的概念，我們可以想成是因為裡面的變數記錄在影響這個環境，這些紀錄又全被放在 `variable object` 裡面。

{% highlight js %}
scope = VO + 所有上層的 VO  /* 範圍/環境 = 這些紀錄的總和 */
/* 所以所謂的 scope chain */
scope chain = [[VO] + [VO1] + [VO2] + [VO n+1]];
{% endhighlight %}

# 確立一個 scope chain 的變數物件們

現在我們知道 scope chain 就是用來保存這些變數物件，且我們的第一個變數物件就是當前執行環境物件自己的`變數物件`，剩下的 `變數物件` 就是上層執行環境或說父代執行環境的

{% highlight js %}
function one() {
  two();

  function two() {
    three();

    function three() {
      alert("I am at function three");
    }
  }
}
{% endhighlight %}

這個範例很簡單的示範從 global context 我們呼叫的 `one()`，`one()` 呼叫 `two()`，接著在呼叫 `three()`，最後在 function three 發動一個 alert，下圖是當我們執行到 alert 時堆疊的概念圖

![]({{ site.url }}/assets/tutorials/es2.png)

此時的 scope chain 就會是

three() scope chain = [[three() VO] + [two() VO] + [one() VO] + [Global VO]];

# Lexical Scope
JS 中有一個挺重要的特性那就是直譯器採用 `Lexical Scoping`，它和 `Dynamic Scoping` 相反... 

簡單的來說這個 `Lexical Scoping` 只不過是在說：函式內部定義的程式碼是根據定義時決定其值而不是動態決定。用上一篇的概念來解釋那就是變數 `決定值` 的時候是去哪個範圍(scope)找。

還是很難懂！好吧！會搞得這麼複雜呢我想應該是源自於對於 scope 翻譯翻的不好
參考維基百科的定義


>In computer programming, the scope of a name binding – an association of a name to an entity, such as a variable – is the part of a computer program where the binding is valid: where the name can be used to refer to the entity.

OK 我們知道了英文說 scope 其實就只是在說明變數名稱該怎麼樣跟物件實例做關聯，關聯的`範圍`。至於這個 `Lexical Scoping` 讓我們來點實際範例看看什麼叫做 `根據定義時決定` 

{% highlight js %}
function start() {
  alert(args);
}

function server() {
  var args = "parameter here.";
  start();
}

server(); // ReferenceError: args is not defined
{% endhighlight %}

如我我們照著上一篇執行環境的流程順序來看那麼 `start()` 的 VO 其實就只能夠去參考 `start() 變數物件` 和 `Global 的變數物件` 而已。

說明完 `Lexical Scoping` 我們回到上面的 one two three 的例子，不管呼叫的順序是怎樣，`three()` 永遠只能`靜態`的去參考 `two()` 層的定義，當然還有自己的，以此類推一層一層往上。

{% highlight js %}
(function a () {
  var a = 1;
  function b() {
    var b = 2;
    console.log(a);
    console.log(b);
  }
  b();
}());
{% endhighlight %}

`scope chain` 的用途大概就是像上面這樣，就只是層層往上參考。
而 `Lexical Scoping` 會這麼困難倒不是因為觀念，而是因為實作的時候，context 是在呼叫的時候才開始建立，配上靜態的 `Lexical scoping` 定義常常就會導致一些非預期的結果或行為。

最常見的例子就是

{% highlight js %}
var alerts = [];

for (var i = 0; i < 5; i++) {
  alerts.push(function inner(){
    alert(i);
  })
}

alerts[0](); // 5
alerts[1](); // 5
alerts[2](); // 5
alerts[3](); // 5
alerts[4](); // 5
{% endhighlight %}

第一次看這個範例碼通常都會覺得 `alert(i);` 會是輸出從 0 - 4
這是最常發生對 function inner 混淆的地方。 `inner` 是在 `global context` 這邊被定義的，因為 `for` 沒有自己的 scope
OK 且該執行環境是在被呼叫的時候才建立，這個時候 i 早就是 5 了。

現在您明白其中的原由了。

# 解析變數的值
下面這個範例輸出 `a + b + c` = 6

{% highlight js %}
function one() {
    var a = 1;
    two();
    function two() {
        var b = 2;
        three();
        function three() {
            var c = 3;
            alert(a + b + c); // 6
        }
    }
}

one()​;​
{% endhighlight %}

我們剛剛沒有認真的解釋關於 scope chain 的部分，現在先看看上面的例子，乍看之下我們知道 `a` `b` 並不在 function three 裡面，那麼這個範例是怎麼輸出 6 的

從 `alert(a + b + c);` 這行程式，當直譯器開始要找 `a` 的時候，它會不斷的到 scope chain 裡面去尋找，這個過程如下圖

![]({{ site.url }}/assets/tutorials/es2.png)

一開始會到自己的 VO 去找，找不到換下一個一直到 Global 為止。
如果都找不到則丟出 `ReferenceError` 的錯誤，所以上面這段小範例 `a` `b` `c` 都會找到值。

# 關於閉包
在 Javascript 中，閉包常常被認為是種魔術，且只有進階的開發者才真的搞懂它，不過事實上對於閉包的了解其實源自於對於 scope chain 的了解， [Crockford](http://javascript.crockford.com/private.html) 說: 

>An inner function always has access to the vars and parameters of its outer function, even after the outer function has returned…

簡單的說就是，位在內部的 function 永遠可以存取到外部的變數和參數，即使外部 function 已經執行完畢。
再根據 MDN 說明，其實閉包就是一個特殊的物件，它有兩個含義：
  
  * 它是一個 function。
  * 它產生了一個 context 執行環境，配合上面的說明你就知道其實他只是幫你你記錄上一層有宣告的變數，沒錯就是那個 variable object。

>Closures are functions that refer to independent (free) variables. In other words, the function defined in the closure 'remembers' the environment in which it was created.

{% highlight js %}
function factory() {
  var brand = "BMW";

  return function car() {
    alert("I am a " + brand + " car");
  }
}

var carMaker = factory();
carMaker(); //  I am a BMW car
{% endhighlight %}

`global context` 有一個稱為 `factory()` 的函式，接著有一個變數叫做 `carMaker` ，這個 carMaker 儲存了 facotry 回傳的值。通常開發者會感到困惑的地方是為什麼 `brand` 還會存在，不是說 function 一旦執行結束就後就會不見嗎？那為什麼 brand 還會在。

然而如果我們來仔細看看關於執行環境的部分我們會看到

{% highlight js %}
// Global Context
global.VO = {
  factory: pointer to factory(),
  carMaker: 是 global.VO.factory 的回傳值
  scopeChain: [global.VO]
}

// Factory 執行環境
factory.VO = {
  car: pointer to car(),
  brand: 'BMW',
  scopeChain: [factory.VO, global.VO]
}

// car 執行環境
car.VO = {
  scopeChain = [car.VO, factory.VO, global.VO]
}
{% endhighlight %}

現在我們先看到當呼叫 `carMaker()` 的時候，實際上我們拿到 factory 回傳的值，這個回傳值回傳一個指向 `car()` 的指標，接著當我們進入 car 內部執行的時候這個 scope chain 是 `[car.VO, factory.VO, global.VO]` 現在呢需要 `brand` 這個變數的值所以會先找自己的 `car.VO` 找不到再往下找 `factory.VO` 就可以找到了。

從另一個角度來看其實就是雖然 function factory 本身的實例執行完後就消失了，可是因為 VO 還被參考，所以 GC 不會將其回收。

說到這邊我們已經深入的解釋完關於 scope chain ，lexical scoping 以及關於 colsures 和變數之間是如何運作的了。
剩下的文章我們將來看看一些牽扯到上面議題的有趣情況

# prototype chain 如何影響變數解析
Javascript 幾乎所有東西都使用 `prototype` 的方式來實作繼承，除了 `null`, `undefined`。當我們試圖存取一個物件的屬性時，直譯器會試著解析在物件實例中的屬性，那如果找不到他就會繼續找 prototype chain，直到找到屬性或者檢索完畢整個 chain 所記錄的關聯。

那麼第一個有趣的問題來了，直譯器在解析一個屬性的時候到底是用 `scope chain` 還是 `prototype chain`？ 答案是都會用。當試著解析一個屬性或者識別的時候，`scope chain` 會先被用來找尋物件的所在，當物件被找到的時候接著就用該物件的 `prototype chain` 來找屬性名稱。

下面我們用兩段程式碼來解釋整個流程

{% highlight js %}
var bar = {}

function foo() {
  bar.a = "Set from foo()";

  return function inner() {
    alert(bar.a);
  }
}
foo()(); // 'Set from foo()'
{% endhighlight %}

首先是 `bar.a = "Set from foo()";` 這一行建立全域物件 `bar` 的 `a` 屬性。直譯器會到 `scope chain` 找尋 `bar.a` 並預期會在 `global context` 找到它。

現在換另外一個範例

{% highlight js %}
var bar = {};

function foo() {
  Object.prototype.a = "Set from prototype";

  return function inner() {
    alert(bar.a);
  }
}

foo()(); // 'Set from prototype()'
{% endhighlight %}

在執行時期，當 `inner()` 被呼叫的時候會先試圖在 scope chain 裡面解析 `bar.a`，而 `bar` 的實例，會在 global context 被找到 `bar` 然後搜尋 `bar` 裡面的屬性 `a`，然而 `a` 並沒有被設定在 `bar` 裡面，所以直譯器的下一步會檢索物件的 `prototype chain` 然後在 `Object.prototype` 裡面找到 `a`

上面的過程就是整個識別解析的流程，先在 scope chain 找到物件然後查看 `prototype chain` 直到屬性被找到為止否則就回傳 `undefined`

# 何時該使用閉包
閉包是一個非常強大的概念，通常我們會在某些情況下使用

* 封裝
這讓我們可以將一些不想外露的細節封裝在執行環境中，只露出想要 public 的部分。
例如：

{% highlight js %}
var classicModulePattern = function(){
  var privateVariable = 1;
  function privateFunction(){
    alert('private');
  }
  return {
   publicVariable:2,
   publicFunction:function(){
    classicModulePattern.anotherPublicFunction(); 
   },
   anotherPublicFunction:function(){
    privateFunction();
   }
  }
}();

classicModulePattern.publicFunction();
{% endhighlight %}

* Callbacks 回呼
callback 可能我們最常用的一種閉包，典型瀏覽器中通常是採用單執行緒的 Event Loop，正常情況下，一個事件完成才會執行下一個事件。
 callback 讓我們能夠延遲函式的調用，非同步風格的寫法，我們常常用在回應當一個事件完成的時候。舉例來說當你對伺服器呼叫一個 AJAX ，我們通常會使用 callback 來處理伺服器回應的部分。

# 閉包參數
我們也可以把一個閉包當作參數傳進 function 中，舉個例子我們透過傳入不同的閉包參數就可以完成不同的排列處理

# 何時不該使用閉包？
雖然閉包很好用但因為效能的因素您還是應該謹慎使用

* 過多的作用域
 多個巢狀 function 是一個典型的狀況。記住每一個當你需要取得一個變數時，scope chain 一定會一層一層檢索，直到找到該物件或值，所以越多層會導致找尋時間變長。

* 記憶體回收
Javascript 具有記憶體回收(Garbage Collection)的機制，指的是開發者不需要處理關於記憶體的議題。不過通常有自動回收機制，因為無法直接控制記憶體的部分容易導致程式記憶體洩漏進而造成效能問題。
不同的 JS 引擎實作 GC 的方式明顯有差異，ECMAScript 並沒有定義該如何實作回收記憶體的方式，但為了提高效能與盡可能降低記憶體洩漏的問題大部份的引擎都遵循一樣的宗旨。
一般來說記憶體回收處理器會在當物件不再被參考的時候將其釋放

> Memory Leak (中文翻成記憶體漏洩)。內部記憶體泄漏指由於疏忽或錯誤造成程式未能釋放已經不再使用的內部記憶體的情況。
內部記憶體泄漏並非指內部記憶體在物理上的消失，而是應用程式分配某段內部記憶體後，由於設計錯誤，導致在釋放該段內部記憶體之前就失去了對該段內部記憶體的控制，從而造成了內部記憶體的浪費。
更確切的說 Memory Leak 造成的原因是某個被配置(allocated)的記憶體無法再被參考(referenced)，也無法被釋放(released)。那塊被配置的記憶體就無法被系統再使用，所以要看一個程式有否Memory Leak，很簡單的方法就是去看作業系統的實體記憶體使用圖，如果隨著時間增加，記憶體的使用量呈現明顯增加的趨勢，這個程式就極有可能有潛在的Memory Leak問題。

* 循環引用
循環引用是在描述一種狀況，當 A 物件參考到 B，但是 B 物件又參考回 A 物件。
針對舊版的 IE 參照一個 DOM 元素常常會造成記憶體洩漏。為什麼？因為在 IE JScript 引擎和 DOM 分別各自有自己的記憶體回收器，所以當從 JS 中參考一個 DOM 元素時，JS 回收器認為這是 DOM 回收器的工作，而 DOM 回收器又把這個任務指給 JS 回收器。結果就是兩個回收器循環引用。
上面扯遠了，那這跟閉包有什麼關係。原因是在閉包中很容易寫出循環引用，讓我們來看一個實際的例子

{% highlight js %}
function example() {
    var el = document.getElementById('el');
    el.onclick = function() {
      this.style.backgroundColor = 'blue';
    }
    // el=null; 
}
{% endhighlight %}

看起來沒有循環參考的問題，但實際上呢？el物件的屬性參考到了一個函式，這函式卻擁有存取el物件的能力。因此循環參考就此形成。要破壞這種記憶體洩漏其實不難，上例程式碼中的 `el=null` 就可以達成這目的，當然也可以在一開始就不使用 `el` 變數。


---
layout: post
title: '重讀 Axel 的 Javascript 中的 Expression vs Statement 一文'
date: 2016-03-06 12:00:00
categories: javascript
---

# 前言

[原文](http://www.2ality.com/2012/09/expressions-vs-statements.html)在此，對於 Axel 的文章一直有種雖然短卻難以讀透的感覺。這篇文章是再讀一次的翻譯搭配自己的理解說明，如有錯誤歡迎指教。

> 註: 下面一些範例當我們在瀏覽器 console 執行時，回傳值與程式執行的順序在 Chrome 與 Firefox 會有差別。注意一下`箭頭`符號就知道哪個是 `return` 了。
例如

  {% highlight bash %}
  > function foo () {console.log('foo');}
  > foo();

  // 在 Chrome 是
  foo
  undefined

  // 在 Firefox 是
  undefined
  foo
  {% endhighlight %}


# 1. Statements 述句和 Expressions 表達式

一直以來，在讀技術文章的時候您一定不陌生這兩個詞，因為小弟對於這種細枝末節並不是很重視，加上計算機背景又不深厚。所以對於一些文章和概念的掌握度一直不是很精確。這次重讀一遍 Axel 的文章，希望能夠對 javascript 有更深入的理解。

事實上，在 javascript 中能夠清楚的分辨 `expressions` 和 `statements` 的差異對於撰寫程式碼是有一定的幫助，可以避免掉入一些陷阱。

簡單來說一個表達式 `expressions` 會產生一個值，我們會在撰寫它的地方期望得到一個`值`。舉例來說像是調用 function 中的引數(arguments)，或者指定式 `=` 的右邊都屬於 expressions 的位置。

> 參數(parameters)，引數(arguments)

  > * parameters 即在 function 中定義必須在呼叫程序時傳遞，可以用來取值的符號(變數名稱)
  > * arguments 是實際呼叫時，傳入的值

下面的每一行都是一個 expression：

{% highlight js %}
myvar
100 + x
fn("a", "b")
{% endhighlight %}

而大體來說述句 `statements` 即執行動作，完成特定任務。賦值，迴圈和 `if 述句`都是 `statements` 的例子。
在我們實際撰寫的程式碼中到底是怎麼區分的呢？讓我們看看 MDN 上定義的 `if 述句`

{% highlight js %}
if (condition)
   statement1
[else
   statement2]
{% endhighlight %}

要明白這些事情我們得先從 `syntax` 開始講起，基本上程式是透過一系列規定好的語法組成，稱為 syntax ，它類似於我們人類語言中的`文法`，不管是中文還是英文。一個程式要遵循著 `syntax` 並且由一系列 `statements` 組成的。

javascript 直譯器在解析程式碼時對於語法結構，即這些程式碼出現的位置會有對應的處理方式。

另外，拿上面的例子來說，任何 javascript 預期會有 `statements` 的地方你都可以使用 `expressions`，例如在 `statement1` 的地方呼叫一個 function。
這個 function 就稱作 `expression statement` 屬於一種特殊的 `statement` ，這個 function 自然可以 return 一個值，同時也可以在內部產生一些 `side effect`，不過如果我們重點擺在一些 `side effect` 部分時，通常就會回傳 `undefined`。如下圖

![](http://i.imgur.com/TxEZ6kT.png)

> 通常一個 statement 是獨立的，只會完成某項任務，不過如果它影響了整個程式例如: 異動了機器內部的狀態，或者影響後面的 statement，這些造成的改變我們就稱為 side effect (副作用)

反過來，我們`不可以`在預期是 `expression` 的地方換成 `statement`。例如我們不可以在 function 的引數的地方改成 `if 述句`。

* syntax
  - statements
    + expression statements
  - expressions

# 2. statements 與 expressions

讓我們看一下這兩段類似功能的程式碼，我們可能會更加清楚它們之間的分別。

### if 條件式語句和條件運算子(三元運算子)

{% highlight js %}
var x;
if (y >= 0) {
  x = y;
} else {
  x = -y;
}
{% endhighlight %}

上面這幾句程式碼無疑都是 `statements`，另外 `expression` 也有個對應的寫法和上面這段程式碼完全等價

{% highlight js %}
var x = (y >= 0 ? y : -y);
{% endhighlight %}

在等號和分號之間的就是一個 `expression`，其中的 `()` 不是必須的，但加上去比較容易閱讀。

### 分號; 與 逗號,

在 javascript 中 `statement` 之間我們可以用 `;` 分號來區分和串連。

{% highlight js %}
foo(); bar()
{% endhighlight %}

而 `expression` 也有一個鮮為人知的 `,` 運算子可以用來串連。

{% highlight js %}
foo(), bar()
{% endhighlight %}

兩個 expression 都會執行，但是返回最後面的。

{% highlight bash %}
> 'a', 'b'
'b'

> var x = ('x', 'y')
> x
'y'
{% endhighlight %}

# 3. 容易產生誤會的 expressions (看起來像 statements)

有些 `expressions` 看起來像是 `statements`。下面我們會列出一些容易產生疑義的例子逐一討論。主要是因為它們會因為不同的位置產生不同的行為。

### 物件實字(Object Literal)與程式碼區塊(Block)

下面這段範例是一個物件實字，屬於 `expression` ，用來產生一個物件

{% highlight bash %}
{
  foo: bar(3, 5)
}
{% endhighlight %}

> `Object Literal` 是一個透過 `{}` 與 `,` 逗號分隔的鍵值對列表就是 `var o = {name:'Object'}` 這樣的寫法。

同時它還是一個符合規範的 `statement`，因為它具備了：

* block: 一段 statement 放在 `{}` 中
* label: 我們可以在任何一段 statement 之前放上一個 label，在這邊 label 是 `foo:`
* statement: 一個 expression statement `bar(3, 5)`

所以 `{}` 到底是一個 block 還是物件實字，你可能會說是`物件`那讓我們來看看下面這個奇怪的例子

{% highlight js %}
// 在看這個奇怪的範例之前讓我們先看看一些 javascript 的行為
// 當我們把非數字相加時
> 1 + 'string'
'1string'

> 1 + undefined
NaN

> 1 + null
1

> 1 + [2,3,]
"12,3"

> 1 + {name: 'andyyou'}
"1[object Object]"

// 上面的範例我們得知，除了 undefined 和 null，基本上 js 會把物件先 `toString()` 再相加。

> [].toString()
""

> [1, 2, 3].toString()
"1,2,3"

> var o = {};
> o.toString();
"[object Object]"

// 有了上面的基礎知識之後，讓我們來看看這令人嚇尿的行為

> [] + {}
"[object Object]"

// 好！這題如我們所料，[] 產生 "" 加上 {} 產生 "[object Object]"

// 先問你個問題: + 兩邊的運算元能不能互換而結果不變
// 你可能回答: 是！！！
// 但....

> {} + []
0
{% endhighlight %}

上面程式碼最後一句的 `{}` 是一個 `block` 所以執行完之後接 `+[]`。

{% highlight js %}
> +[]
0
{% endhighlight %}

嚇尿了吧！除了 if, 迴圈外 javascript 也具有獨立的 block。
下面這段程式碼說明了 `label` 和 `block` 的用法:

{% highlight js %}
function test (printTwo) {
  printing: {
    console.log('One');
    if (!printTow) break printing;
    console.log('Two');
  }
  console.log('Three');
}
{% endhighlight %}

執行的結果

{% highlight bash %}
> test(false)
"One"
"Three"

> test(true)
"One"
"Two"
"Three"
{% endhighlight %}

從上面驗證了 `{}` 的語法如果遇到 `statements` 的位置，就會被當成 `statements`，而如果在 `expressions` 的位置就會被當解析成一個值。

{% highlight js %}
> {} + [];
// 就是一個最好的例子，{} 被當作 statement 就是一個 block

// 如果換成

> var x = {};
// 那他就是一個 expression 代表一個值 - 一個物件
{% endhighlight %}

讓我們接著看下一個例子。

### Function expression 與 function 宣告

下面的程式碼是一個 `function expression`

{% highlight js %}
function () {}
{% endhighlight %}

你也可以給 `function expression` 一個名稱

{% highlight js %}
function foo () {}
{% endhighlight %}

在當作 `function expression` 時上面的 function 名稱 `foo` 只存在在 function 內部能使用，舉例來說像是一個遞迴。
你可能困惑了，我們到底在說啥？看看下面的例子，我們要說的是當 `function` 放在 `statements` 和 `expressions` 不同位置時的差異

{% highlight js %}
var fn = function me(x) { return x <= 1 ? 1 : x * me(x-1)} // = 等號右邊是一個 expression 的位置
fn(10); // 3628800

console.log(me); // ReferenceError: me is not defined
{% endhighlight %}

具名的 function expression 和函數宣告的寫法看起來是沒有區別的。但實際上這兩者的效果截然不同，`function expression` 產生一個值(一個 function)。函數宣告則產生一個行為，即建立一個變數，然後它的值是一個 function。而且只有 function expression 可以被立即調用，函數宣告不行。

從上面這幾點看來能夠區分 expression 和 statement 挺重要的。

# 4. 使用物件實字與 function expression 當作 statements

我們已經看到有一些 `expression` 和 `statement` 語法上是沒有區別的。這意味著相同的程式碼會有不同行為取決於它出現在 expression 的位置
或者是 statement 位置。為了防止產生疑義。javascript 會禁止 expression statement 使用 `{}` 或 `function` 開頭。

換句話說就是在 javascript 認定為 statement 的位置，使用了 `expression` 會變成 `expression statement`。這並不是 `expression`，所以產生一些特殊的狀況 `{}` 會被當作 block 解釋，`function` 開頭的語法會被當作函數定義。
所你當你想要使用這兩者為開頭撰寫 expression statement 時，你可以放上 `()` 可以確保位於一個 expression 的位置。

這就是 statement 或者 expression 所延伸的問題，也可以說造成我們極度混亂的根源。讓我們來看看 `eval` 和立即調用函式：

### eval

`eval` 會解析他的引數當做一句 `statement`。如果你希望 eval 回傳一個物件你就需要在物件實字外圍放上`()`

{% highlight js %}
> eval("{foo: 123}");
123

> eval("({foo: 123})");
{foo: 123}
{% endhighlight %}

下次再閱讀文件的時候是不是更有感覺了。

### 立即調用函式

立即調用函式的部分

{% highlight js %}
(function () { return "abc" }())
'abc'
{% endhighlight %}

如果你省略了 `()` 如下，你就會得到一個語法錯誤的訊息。function 宣告不可以匿名。

{% highlight js %}
function () { return "abc" }()
SyntaxError: function statement requires a name
{% endhighlight %}

就算替 function 加上名稱還是噴錯

{% highlight js %}
function foo() { return "abc" }()
SyntaxError: syntax error
{% endhighlight %}

因為函數宣告不能立即調用(IIFE)，不過除了使用 `()` 還有些技巧，當我們硬要要把某段程式當做 `expression` 執行時，可以使用一元運算子 `+` 或 `!`，不過和`()`的方式比起來這會影響回傳結果，如果你不在意的話這也是一種方式。

{% highlight js %}
> +function () { console.log("hello") }()
NaN
hello
{% endhighlight %}

這個 NaN 是 `+` 遇上 `undefined` 的結果，喔！對了還有一種是透過 `void` 的方式

{% highlight js %}
> void function () { console.log("hello") }()
undefined
hello
{% endhighlight %}

### 連續使用立即調用函式

當你在連續呼叫 IIFE 的時候必須要注意`不要忘記分號 ; 結尾`

{% highlight js %}
(function () {}())
(function () {}())
// TypeError: undefined is not a function
{% endhighlight %}

上面的程式碼會產生錯誤因為 javascript 以為第二行的 `()` 是要拿第一行產生的結果當作一個函數來呼叫。

{% highlight js %}
(function () {}());
(function () {}())
// OK
{% endhighlight %}

下面範例因為 javascript 自動補上分號的功能，使用一元運算子的話 `;` 可以省略。

{% highlight js %}
void function () {}()
void function () {}()
// OK
{% endhighlight %}

javascript 會自動補上分號是因為接在第一行之後的 `void` 並不是可以接下去的語句(符合規範能串在一起的寫法)。

另外關於 javascript 自動補上分號有幾項建議如下：

1. 在 return，break，continue，++，-- 五種 statement 中，`換行字元`可完全等於 `;`。
2. var，if，do while，for，continue，break，return，with，switch，throw，try，debugger 關鍵字開頭，以及空的 statement，上一行會自動補上分號。
3. 遇到 expression statement 和 function expression 情況非常複雜，後面請務必要加上分號。
4. 凡 `(` 和 `[` 開頭的 statements 前面或上一句不加非常危險。

想要更[深入明白 ASI](https://segmentfault.com/a/1190000004548664)，請參考。

# 總結

* syntax : 語法(文法)，該怎麼組織 statements 與 expressions。
* expressions : 會產生一個值，其意義就是代表一個值的表式例如 `x + y`。
* statements : 完成某項任務的操作。賦值，條件判斷，宣告都算是 statements; `if (condiction) { console.log('WoooW!') }`。
* expression statements : 屬於一種 statement，其產生一個值(或說回傳一個值)，並完成某項任務。例如：`x += 1` 或者在 statement 執行一個 side effect 的函數呼叫。
* 在 statements 位置放入 expressions 要小心(即 expression statement)，因為 javascript 對於 `expression` 和 `expression statement` 解釋行為是不一樣的。
* 下面這兩種語法對於其位置尤其需要注意
  - `function`
    + statement 位置:  當作函數宣告，即建立一個變數它的值是一個 function。，不能立即調用。
    + expression 位置: 為 function expression 產生一個為 function 的值，可以被立即調用(IIFE)。
  - `{}`
    + statement 位置:  block 一個程式碼區塊，例如 for, label 的 block。
    + expression 位置: 物件實字，建立一個值 - 物件。

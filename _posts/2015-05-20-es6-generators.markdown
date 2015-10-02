---
layout: post
title: 'ES6 Generators 基礎'
date: 2015-05-20 05:30:00
categories: Javascript
---

在 Javascript ES6 的新功能中，有一個新品種的 function 稱為 `generator` 這個名字有點奇怪，不過它的行為在第一次看到的時候似乎更加奇怪。這篇筆記目的在解釋關於 generator 基本的運作原理。

# 執行到完成
在我們談論關於 generator 時，第一件事情是對於標題下的 `執行到完成`， generator 是如何不同於普通函式。

不過您是否看懂上面這一小段，您一直都對於 function 有一個相當基本的認知那就是一旦函式開始執行，它就會一直執行到完成為止。

{% highlight js %}
setTimeout(function() {
  console.log("Settimeout excuted");
}, 1); 

function foo() {
  for(var i=0; i<1000; i++) {
    console.log(i);
  }
}

foo();
// 1 - 1000
// Settimeout excuted
// 即使只差 1ms 還是要等 foo 跑完。
{% endhighlight %}

這邊 `for` 迴圈如果數字再大一點那就需要點時間，至少大於 1ms，然後您就發現 `setTimeout` 明明說好是 1ms 後發動卻無法中斷 `foo` 所以 `setTimeout` 就被卡在 `event-loop` 等待直到輪到它為止。

那如果 `foo()` 可以被中斷呢? 不會破壞我們的程式嗎?
那的確是個挑戰，當我們採取多執行緒來寫程式的時候，不過好佳在我們用 Javascript 所以我們不需要擔心那些，因為 Javascript 永遠是單執行緒，意思是在一個時間點永遠只有一個 function 或指令在執行。

注意: `Web Worker` 是讓你可以啟動另一個完全分離的執行緒，給一部分的 JS 在其中執行的機制，跟您主要的執行緒是平行的。
不在我們的程式使用多執行並發是因為兩個執行緒只能透過非同步事件來互相溝通。雖然是兩個平行的執行緒一旦要溝通還是要遵守 `event-loop` 的規則，一次只有一個動作，且還是一旦執行就要執行到完成。

# 執行 - 暫停 - 執行
透過 ES6 generator 我們可以有不同的函式，它可以在執行到一半`暫停`，然後再`回復`執行。讓其他程式可以在暫停這段期間先跑。

如果您曾經得讀過關於並發或者執行緒程式設計的文章，您也許看過 `cooperative` 這個術語，其基本的意思是一個進程(process)，在我們 JS 的範例是 function 會自己決定何時應該允許中斷暫停。因此可以和其他的程式碼協同合作。這個觀念的對比是 `preemptive` 指出一個進程可能會違反原本的設計而中斷。

ES6 generator function 在其並發行為裡是可以協同合作的。在 generator function 裡面您可以使用新的關鍵字 `yield` 來從內部暫停。沒有東西可以從外部暫停一個 generator ，必須要透過 yield 從內部暫停。

然而一旦 generator 用 yield 暫停了自己，它就`不能靠自己回復`。必須要有個外部的控制行為來使其回復執行。稍後會解釋該如何做。

所以基本上，一個 generator 函式可以被暫停，重啟，隨您高興開開關關幾次。
事實上您可以用一個無限迴圈來搭配 generator ，在一般 JS 程式中出現無限迴圈通常是寫錯了，不過搭配 generator 卻是合理的而且有時候您的確就是想要這麼做。

更重要的是，這個`暫停`和`重啟`不只單單是控制 generator 的執行流程，而且還提供了兩種方式在執行過程中傳遞輸入和輸出的訊息

在一般函式中您可以傳入參數(Parameters)然後 `return` 一個結果。在 generator 您可以透過 yield 把資料丟出來，然後傳回其他資料再回復執行。

# 怎麼寫？
這一小段讓我們來開始介紹關於這些新功能的語法(syntax)

首先是這個新的 `generator function` 的宣告

{% highlight js %}
function *foo() {
  // ...
}
{% endhighlight %}

注意到 `*` 了嗎? 這個新語法看起來有點奇怪，在其他語言(C, Object-C)中看起來像是函式要回傳一個指標。不過不要搞混，這只是一個符號用來判斷這是一個特殊的函式 generator。

您可能看過其他文章使用 `function* foo(){}` 而不是 `function *foo(){}`，兩種宣告都正確。

generator function 大概就是一個普通的 function ，只是在內部多了一些新的語法可以使用。

而最主要的新玩具就是我們上面提到的 `yield`，直接來看點範例

{% highlight js %}
function *foo() {
  var a = 1 + (yield "fooo");
  console.log(x);
}
{% endhighlight %}

當 generator 執行到 `yield` 時會暫停，這個時候會把右邊的 expression 把就是 `fooo` 字串送出來，當 generator 再次啟動的時候無論資料有沒有送進去 generator 就會取得另外一個 `yield expression`，把 1 + `yield expression` 計算的結果。

> `yield` 的意思我喜歡用`佔位`的概念來形容，有點像 `hook` 的觀念。

剛剛我說的有點讓你混淆，讓我們再來釐清一次 `yield` 第一個功能是`暫停`，當函式走到 `yield` 的時候會先停止，然後把右邊的 expression 丟到外面。
停一下！這個 expression 跟待會要接回來的資料沒有關係。把 yield 想成`佔位符`，意思是停在這邊等別人把值丟進來，同時在我停下來的時候也可以丟個東西出去。
有點類似 HTTP 的運作概念，執行到 yield 的時候對外部發送個 request 然後等待外部把資料送回來。再停一下！什麼外部？就是 generator 的實體物件。
他會負責把資料再丟回來。丟回來的時候記住就不會再被那個 "fooo" 混淆了， "fooo" 丟出去後就沒有他的事了。

這個例子太難懂? 讓我們看點更完整的基本用法

{% highlight js %}
function *gen() {
  console.log('start');
  var o = yield "called";
  console.log("I am back and bring " + o);
}
var a = gen();          // 第一次呼叫時是返回一個 generator 物件
var b = a.next();       // 開始執行，到 yield 時會暫停執行並返回，返回值是一個物件
console.log(b.value);   // 他的 value 屬性是 yield 右側的 expression 的執行結果
console.log(b.done);    // 是否完成
var c = a.next("something from outside"); // 帶個值回去
console.log(c.done);    // 完成
a.next(); // 如果再呼叫 next()，就會拋出例外
{% endhighlight %}

現在您應該看懂了兩種溝通方式了吧

您可以在任何 expression 的位置單純使用 `yield`，將其置放在 expression/statement 之中，然後輸出的部分就會是 undefined。

> 一個片段程式碼產生一個值稱之為 expression，expression 類似語言中的片語，一個短句。
  statement 則是一句完整的句子，在 JS 中用 `;` 結束當作一個句子。
  通常一個 statement 是獨立的，只會完成某項任務，不過如果它影響了整個程式例如: 異動了機器內部的狀態，或者影響後面的 statement，這些造成的改變我們就稱為 side effect (副作用)

{% highlight js %}
function foo(x) {
  console.log("x: " + x);
}

function *bar() {
  yield; // 只會暫停
  foo(yield); // 暫停並等待傳入參數到 foo()
}
{% endhighlight %}

# Generator Iterator
Iterator 迭代器實際上是一種特殊的行為，也可以表示一個設計模式。這個行為指的是讓我們可以透過呼叫 `next()` 
在一個排序的集合中，特定時間點下一次只取得一個值。舉例來說我們在`[1, 2, 3, 4, 5]`這個陣列上使用 iterator。 
第一次呼叫 `next()` 時我們會取得 `1`，第二次 `2` 以此類推
當所有元素值都被回傳過後，`next()` 將會回傳 `null`, `false` 或者其他通知我們已經跑完所有元素的訊號。

剛剛提到我們在外部用來控制 generator function 的那個實體物件就是 `generator iterator` ，聽起來好像挺複雜的不過讓我們來看看實際上的例子

{% highlight js %}
// 假設我們有一個 generator function
function *foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
}
{% endhighlight %}

為了逐步從 `*foo` 這個 generator function 中取得 yield 傳出來的資料我們需要一個迭代器

{% highlight js %}
var it = foo(); // 再次強調，第一次呼叫 function 傳回一個迭代器
{% endhighlight %}

所以！！第一次像我們平常一樣呼叫 function 的時候並不會真的執行。

在我們的觀念裡這的確有點陌生。您可能也會好奇想知道為什麼不是用 `var it = new foo()` 因為剛剛不是說回傳一個 iterator 實體物件嗎? 好吧我真的不知道，等我知道了在告訴你。這邊暫時先不討論這個問題 XD

接著讓我們開始來使用 iterator

{% highlight js %}
var message = it.next();
console.log(message); // #=> {value: 1, done: false}
{% endhighlight %}

第一次迭代之後我們會拿到 `yield` 傳出來的資料。再次強調一遍不要把 yield 的觀念當作是 function ，把它分成兩次一次負責輸出，取得資料之後您可以修改操作然後再把您的值丟回去。留在 function yield 右邊的那個 expression 丟出來後就沒用了。不要被它干擾。

每一次我們呼叫 `next()` 都會取得一個物件這個物件有 `value` 和 `done` 兩個屬性。`done` 用來判斷迭代器是否執行完畢。

{% highlight js %}
console.log( it.next() ); // { value:2, done:false }
console.log( it.next() ); // { value:3, done:false }
console.log( it.next() ); // { value:4, done:false }
console.log( it.next() ); // { value:5, done:false }
{% endhighlight %}

執行到第五次我們發現 `done` 還是 `false` 那是因為技術上來說 generator 還沒有執行完成。`yield` 傳出資料了還在等待你傳回去繼續執行。所以我們仍然要呼叫最後一次。
所以最後一次如下：

{% highlight js %}
console.log( it.next() ); // { value:undefined, done:true }
{% endhighlight %}

現在我們執行完全部的流程了但是我們最後一次並沒有拿到任何資料
因為我們已經用盡了 `yield ____` 

在這個關鍵點，您也許想知道我可以從 generator 回傳值嗎？並且如果我這麼做那這個值會在 `{value: , done: true}` 這個物件的 `value` 嗎?

答案是 Yes 可以

{% highlight js %}
function *foo() {
    yield 1;
    return 2;
}

var it = foo();
console.log( it.next() ); // { value:1, done:false }
console.log( it.next() ); // { value:2, done:true }
{% endhighlight %}

等等...但也不可以

依賴 `return` 恐怕不是個好主意，因為當我們使用 `for..of` 的時候最後一個回傳的值會被捨棄

{% highlight js %}
function *foo() {
    yield 1;
    yield 2;
    return 3;
}

var it = foo();
for(var i of it) {
  console.log("使用 for of " + i); 
}
// 使用 for of 1
// 使用 for of 2
{% endhighlight %}

為了完整起見讓我們來看看完整的輸入和輸出是如何操作的

>現在我要來回答您怎麼丟資料回去呢? 就是每個 `next()` 帶入的參數

{% highlight js %}
function *foo(x) {
  // you can use this to inspect
  // console.log(`x: ${x}, y: ${y}, z: ${z}`);

  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);;
  return (x + y + z);
}

var it = foo(5); // 取得 iterator 物件，並不執行

console.log( it.next() ); // { value:6, done:false }
// 第一次呼叫 x: 5, y: undefined, z: undefined
// 執行到 var y 那邊停住，傳出 yield(x+1) = 6

console.log( it.next( 12 ) );   // { value:8, done:false }
// 送 12 進去所以 y = 2 * 12 = 24，第二次呼叫 x:5, y: 24, z: undefined
// 到 var z 那邊停住，輸出 8 等待輸入...

console.log( it.next( 13 ) );   // { value:42, done:true }
// 送 13 進去所以 z = 13 所以第三次呼叫 x: 5, y: 24, z: 13
// 第三次完成並取得 return value 42

{% endhighlight %}

你可以看到我們仍然可以透過參數來初始化 x，第一次初始化並建立 iterator 順便讓 x 等於 5。

第一次 `next()` 我們沒有傳入任何值因為第一次還沒有任何 `yield` 在等你傳值進去。那如果我們傳值了呢? 沒什麼不行，因為這個值會被丟掉。ES6 表示 generator function 會忽略用不到的值。不過有些還沒完全實作 ES6 的瀏覽器可能會出錯。

`yield (x + 1)` 先往外丟出 6 ，然後`等`你第二次呼叫 `next(12)` 所以 y 會是 `2 * 12 = 24` 接著 `yield (y / 3)` 就是 `yield (24 / 3)` 丟出 8 一樣等你把 13 丟進去所以 z = 13

最後 `return (x + y + z)` 等於 `42`，看這邊可能會頭暈。多看幾次。

# for..of
ES6 也提供一種方便的迭代語法，`for...of`

{% highlight js %}
function *foo() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    yield 5;
    return 6;
}

for (var v of foo()) {
    console.log( v );
}
// 1 2 3 4 5

console.log( v ); // still `5`, not `6` :(
{% endhighlight %}

如您所見，`foo()` 會先建立迭代器且 `for..of` 會自動去擷取它然後為您自動迭代取出每一個 yield express 吐回來的值。直到 `done:true` 出現。當 `done` 為 `false` 的時候他會自動擷取 `value` 屬性，注意不是物件。一旦 `done: true` 迴圈就停止，而且不會包含最後的 return 的值。

注意上面您可以看到 `for..of` 迴圈會忽略丟掉 `return 6`，而且因為沒有 `next()` 可以使用所以在這種情況下你就不能用 `for..of` 必須要自己操作。

# 結論
OK! 現在您已經懂了 generator 的基本用法了。別擔心如果你現在有點混亂是正常的，我第一次看也是。
很自然的您會想知道，這個新玩具可以在實際專案中做些什麼?

在您熟悉玩過上面這些範例程式碼之後您可能會問

1. 如何把它用在錯誤處理方面?
2. Generator 可以呼叫其他 Generator 嗎?
3. 如何使用非同步的方式操作 Generator?

如果我有時間我會繼續翻譯[系列文章](http://davidwalsh.name/es6-generators)

# 資源
參考翻譯自[The Basics Of ES6 Generators](http://davidwalsh.name/es6-generators)

---
layout: post
title: '理解 Javascript 的 Function.prototype.bind'
date: 2014-09-11 15:51:00
categories: Javascript
---
Function.prototype.bind 函式繫結大概是當您開始學習 Javascript 時最後關注到的議題。
通常是當您遇到一種狀況：需要在其他 `Function` 保留 `this` 的執行環境(Context)。
講執行環境可能太抽象，舉例來說就是當您需要在函式的另外一個函式中呼叫 `this.action()` 的時候。
(這邊如果看不懂請耐著性子看下去)
不過通常這時您可能也不知道您需要的就是 `Function.prototype.bind()`。

第一次您遇到上述的問題，您可能會傾向于把 `this` 儲存成一個變數，接著即便您切換了 Context 還是可以參考到這個物件。
如果您看不懂上面在說什麼，請先參考[這篇文章](http://www.cnblogs.com/leoo2sk/archive/2010/12/19/ecmascript-scope.html#comment_tip)。
許多人會採用 `self`, `_this` 或者 `context` 當作變數名稱，並且把 this 放進去。
這些方法都是可行的且並沒有什麼不妥，但有一個更不錯的方式。

## 我們實際上要解決的問題是？
下面有一份簡單的範例程式碼，情況是某人忘記把 Context 存成一個變數：

{% highlight js %}
/**
 * 我們舉例一個機器人物件，機器人有一些基本的 function 來執行動作
 * 不過問題是當我們要求機器人執行動作的時候，他需要先到確定還有沒有能量。
 * 
 *
 * Note: 這段程式碼只是希望能夠用具像化一點的比喻來說明。
 */
var Robot = {
  /** private */
  power: 100,

  walk: function () {
    console.log('Robot walked');
    this.power -= 10;
  },

  fly: function () {
    console.log('Robot flied');
    this.power -= 20;
  },

  check: function (excute) {
    if (this.power > 0)
      excute();
  },
  
  /** public */
  showoff: function () {
    this.check(function () {
      this.walk(); /* 實際執行的動作。 */
      this.fly();
    })
  }
}

Robot.showoff();
{% endhighlight %}

如果照著上面把實際要執行的動作當作 callback 傳給 `check()`，當您要再次呼叫 `this.walk()` 的時候
就會發現出現錯誤訊息

~~~~~
TypeError: Object #<Object> has no method 'walk'
~~~~~

這是因為我們再次傳進去的匿名函式不知道關於 `this` 的東西，在這裏我們並沒有善用閉包來保存 Context。
而對很多人可能就會把上面的範例修改為如下

{% highlight js %}
var Robot = {
  /** private */
  power: 100,

  walk: function () {
    console.log('Robot walked');
    this.power -= 10;
  },

  fly: function () {
    console.log('Robot flied');
    this.power -= 20;
  },

  check: function (excute) {
    if (this.power > 0)
      excute();
  },
  
  /** public */
  showoff: function () {
    var that = this;
    this.check(function () {
      that.walk(); /* 實際執行的動作。 */
      that.fly();
    })
  }
}

Robot.showoff();
{% endhighlight %}

宣告成區域變數之後，閉包就會幫助我們 Keep 這個 Context，這也是相對直覺的方式，同上面說的這沒有任何不妥。
不過我們知道了一件事，就是我們需要保存 Robot 這個物件參考的 Context，給 Callback 即範例中的 excute。
當我們呼叫 `that`.walk() 的時候其實就是在使用閉包。根據 MDN 說明，其實閉包就是一個特殊的物件，它有兩個含義：

1. 它是一個 function。
2. 它產生了一個 Context ，概略的說就是幫你記錄上一層有宣告的變數。

這裏就不詳細說明關於閉包，不理解的推薦[這篇文章](http://www.cnblogs.com/leoo2sk/archive/2010/12/19/ecmascript-scope.html#comment_tip)和[這篇](http://blog.taian.su/2012-10-17-explaining-javascript-scope-and-closures-by-robert-nyman/)

閉包的方式已經可以運作了，但是我們覺得他不夠漂亮，因此我們就來使用 `Function.prototype.bind()`。

讓我們來重構上面的程式範例

{% highlight js %}
var Robot = {
  /** private */
  power: 100,

  walk: function () {
    console.log('Robot walked');
    this.power -= 10;
  },

  fly: function () {
    console.log('Robot flied');
    this.power -= 20;
  },

  check: function (excute) {
    if (this.power > 0)
      excute();
  },
  
  /** public */
  showoff: function () {
    // var that = this;
    this.check(function () {
      this.walk();
      this.fly();
    }.bind(this));
  }
}

Robot.showoff();
{% endhighlight %}

## 我們剛剛做了什麼？？
當我們呼叫了 `.bind()` 的時候，其實它非常單純的建立了一個新的 function，只不過這個 function 把 this 的值綁定進去。
所以我們同時把我們想要的 Context(即 Robot 物件) 給保存了下來。接著當我們的回呼函式在執行的時候 this 就是參考到 Robot 這個物件。

如果你有興趣了解 `Function.prototype.bind()` 內部運行機制，他看起來大概就像下面這樣

{% highlight js %}
Function.prototype.bind = function (scope) {
    var fn = this;
    return function () {
        return fn.apply(scope);
    };
}
{% endhighlight %}

接著我們再來看看一個非常簡單的案例：

{% highlight js %}
var foo = {
    x: 3
}

var bar = function () {
  console.log(this.x);
}

bar(); // undefined

var boundFunc = bar.bind(foo);

boundFunc(); // 3
{% endhighlight %}

這個範例是說，`bind()` 幫我們建立了一個新的 function ，並且當我們執行時這個 function 的 `this` 是指向 `foo`，而不是全域。
如果您還是不清楚可以大略理解為：把 function 掛到某個物件底下(當然不是真的加進去)，只是這樣一來可以透過 `this` 取得該物件的 Context。

## 實務應用
當我們學習某些東西時，我不只需要理解觀念，也會試著將其套用在實務上以驗證自己是否明白。來看看一些實務上的應用吧！

## Click 事件處理
其中一個用途是拿它來追蹤點擊次數，然後可能是要把它存在某個物件裡類似下面這樣

{% highlight js %}
var logger = {
  x: 0,
  increment: function () {
    this.x++;
    console.log(this.x);
  }
}
{% endhighlight %}

然後指派一個按鈕的 Click 處理函式去呼叫 logger 物件

{% highlight js %}
document.querySelector('button').addEventListener('click', function () {
  logger.increment();
});
{% endhighlight %}

不過上面這種做法，我們已經建立了一個不必要的匿名函式，並且因為這個匿名函式使用了 logger 呼叫了 `increment()` 所以產生了一個閉包用以確保了 this 是正確的參考物件。
不明白！？再看看下面這個最根本的寫法吧

{% highlight js %}
document.querySelector('button').addEventListener('click', logger.increment); // NaN
{% endhighlight %}

原本是這樣的，當你 Click 的時候執行一個 function ，不過如果你用上面這種寫法的話，意思是你只是把 function 傳進去，根據 `this` 的定義他其實是指的是`誰(哪個物件)呼叫這個函式` this 就是指向它。
而這裡呼叫的人根本不是 `logger` 這個物件。也因此使用了一個匿名函式，建立了一個閉包，就是為了保留住 logger 物件的狀態。

好了！講完上面這些我們來看看更乾淨的寫法：

{% highlight js %}
document.querySelector('button').addEventListener('click', logger.increment.bind(logger)); 
{% endhighlight %}

現在，對於 `bind()` 應該比較不陌生了吧！本篇是在寫 React 時產生了一點疑問所研究的筆記，因為在 React 中蠻多機會的使用 `bind`。
如官方的範例

{% highlight js %}
componentDidMount: function() {
  $.ajax({
    url: this.props.url,
    dataType: 'json',
    success: function(data) {
      this.setState({data: data});
    }.bind(this),
    error: function(xhr, status, err) {
      console.error(this.props.url, status, err.toString());
    }.bind(this)
  });
}
{% endhighlight %}
---
layout: post
title: 'React 關於 Refs '
date: 2014-02-19 21:56:00
categories: Reactjs
---

# 關於 Refs
當你透過 `render()` 回傳你的 UI 結構之後，你可能想要從外部調用這個元件實例的方法。通常情況下為了取得一些元件或計算後資料你可能這樣做，但其實是不必要的，因為 React 通常會確保資料是最新的 `props` 且透過 `render()` 傳遞到子元件。不過的確有些情況還是會需要從外部調用方法。
想像下面這種狀況，當你想讓一個已存在的某元件的子元件 `<input />` 在你清空欄位後馬上 `focus` 該 `<input />`：

{% highlight js %}
var App = React.createClass({
  getInitialState: function() {
    return {userInput: ''};
  },
  handleKeyUp: function(e) {
    this.setState({userInput: this.getDOMNode().value});
    // this.setState({userInput: e.target.value}); # 官方範例使用 e.target.value 會導致無法正常運作。
  },
  clearAndFocusInput: function() {
    this.setState({userInput: ''}); // Clear the input
    // 我們希望在這邊可以 focus <input />

  },
  render: function() {
    return (
      <div>
        <div onClick={this.clearAndFocusInput}>
          Click To Focus and Reset
        </div>
        <input
          ref="theInput"
          value={this.state.userInput}
          onKeyUp={this.handleKeyUp}
        />
      </div>
    );
  }
});
React.renderComponent(<App />, document.getElementById('example'));
{% endhighlight %}

請注意，在這個範例我們想要通知 `<input />` 做些事情。這件事件沒辦法透過 `props` 辦到。我們想要讓 `<input />` focus ，然而這邊遇到了一點問題，那就是 `render()` 回傳的不是子元素 `<input />` 的元件，在這個時候它只是一子元件的結構描述。
記得當你從 `render()` 回傳一個結構，它並不包含替你產生子元素的物件實例。關於在內部的標簽(內部的元件)，它們都只是結構描述。
不過你可能已經想到把 `<input />` 先存在變數裡了，注意！！你不應該把『這些事情或物件』存起來，然後期待這可能有一天會用到。如下

{% highlight js %}
// 錯誤範例: DO NOT DO THIS!
render: function() {
  var myInput = <input />;          // 我可能會呼叫這個物件的 Method
  this.rememberThisInput = myInput; // 在未來某個時間點我就可以直接呼叫他
  return (
    <div>
      <div>...</div>
      {myInput}
    </div>
  );
}
{% endhighlight %}

在這個錯誤範例中，`<input />` 只是描述結構。這個描述是用來建立 `<input />` 背後的實際物件。
如果我們把這段程式完成如下並試著觀察這段程式碼的運作，事實上當你呼叫 `this.rememberThisInput` 是會出 ERROR 的！

{% highlight html %}
<html>
  <head>
    <title>Hello React</title>
    <script src="http://fb.me/react-0.8.0.js"></script>
    <script src="http://fb.me/JSXTransformer-0.8.0.js"></script>
    <script src="http://code.jquery.com/jquery-1.10.0.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <div>
      原生 Javascript 範例示範 focus 所以我們可以透過 element.focus() 讓元素 focus。
      <div id='test-trigger'>Click!</div>
      <input type='text' id='test' />
    </div>
    <script>
    /* 原生 Javascript 範例示範 focus */
    var HandleClick = function (e) {
      var el = document.getElementById('test');
      el.focus();
    };
    var el = document.getElementById('test-trigger');
    if (el.addEventListener) {
      el.addEventListener('click', HandleClick, false);
    } else {
      el.attachEvent('onclick', HandleClick);
    }
    </script>

    <script type="text/jsx">
    /** @jsx React.DOM */
    var App = React.createClass({
      getInitialState: function () {
        return {userInput: ''}
      },
      handleKeyUp: function (e) {
        this.setState({userInput: this.getDOMNode().value});
      },
      clearAndFocusInput: function () {
        this.setState({userInput: ''});
        // console.log(this.rememberThisInput);
        // this.rememberThisInput.focus();
      },
      render: function () {
        var myInput = <input value={this.state.userInput} onKeyUp={this.handleKeyUp} />
        this.rememberThisInput = myInput;
        return (
          <div>
            <div onClick={this.clearAndFocusInput} >Click to Focus and Reset</div>
            {myInput}
          </div>
        );
      }

    });
    React.renderComponent(<App />, document.getElementById('example'));
    </script>
  </body>
</html>
{% endhighlight %}

那我們怎麼通知 `<input />` 背後的這個物件呢？

# ref 屬性(attribute)
React 支援一個非常特別的屬性，你可以把它附加到任何在 `render()` 裡面的元件上(就是標簽 tag 上)。這個特殊的屬性可以讓你存取到對應的『背後的實際物件』，它保證可以在任何時間點存取到當下的物件。
下面是一個範例：

#### 1 在 `render()` 裡將回傳任意的元素設定 `ref` 屬性(attribute)

{% highlight html %}
<input ref='myInput' />
{% endhighlight %}

#### 2 在程式碼(典型的範例是在處理事件或函式裡)中你就可以透過 `this.refs` 存取這個『背後的物件』。

{% highlight js %}
this.refs.myInput
{% endhighlight %}

# 完整的範例

{% highlight js %}
/** @jsx React.DOM */
var App = React.createClass({
  getInitialState: function () {
    return {userInput: ''};
  },
  handleKeyUp: function (e) {
    this.setState({userInput: this.getDOMNode().value});
    console.log(e); // 官方範例使用的 e.target.value 是錯誤的！

  },
  clearAndFocusInput: function (e) {
    this.setState({userInput: ''});
    this.refs.theInput.getDOMNode().focus();
  },
  render: function () {
    return (
      <div>
        <div onClick={this.clearAndFocusInput}>
          Click To Focus and Rest
        </div>
        <input ref='theInput'
               value={this.state.userInput}
               onKeyUp={this.handleKeyUp} />
      </div>
    );
  }
});
React.renderComponent(<App />, document.getElementById('example'));
{% endhighlight %}

在這個範例裡，`render()` 方法裡回傳的結構中包含了一個 `<input />` 的結構描述，不過這次不同的是這個物件可以透過 `this.refs.theInput` 取得。然後在 `clearAndFocusInput` 函式裡使用 `this.refs.theInput` 。

# 總結
比起使用 `this.props` 和 `this.state` ，`this.refs`是操作物件或傳送訊息給特定子元素最方便的方式，但是建議不要透過他們去操作你的資料。一般來說被動處理計算的資料應該使用 `this.props` 和 `this.state`。
## Refs 的用途
* 可以定義任何 public 的 method 在你的元件類別裡(例如 reset 方法)，然後透過 `refs` 去呼叫。
* 在你需要呼叫 DOM 的 API 時取得該元素。`this.refs.myInput.getDOMNode()`。
* Refs 會自動記錄，如果你的子元素被破壞這個 `refs` 也會被破壞。不用擔心記憶體的問題，除非你做了一些瘋狂的事情，像是把整個物件都加入參考。

## 注意事項
* 不要在 `render()` 方法裡面使用 `this.refs` 或者當任何元件的 `render()` 正在運行的時候。
* 如果你想要使用 `Google Closure Compiler` (Javascript minify) ，請檢查不要用任何指定屬性屬性的方式使用 refs，這意味著當你設定了一個 `ref='myRefstring'`那麼你最好使用 `this.refs['myRefString']` 這種方式。
* 如果你是第一次用 React 開發，通常你會傾向讓 `this.refs` 去幫你達到你要的功能，如果遇到這種情形，請審慎思考關鍵在哪，該如何設計階層結構，該 `state` 管理控制哪些資料。
通常把`state`放置在元件最高階層，控制好關於『自己』的狀態會讓程式碼變得乾淨，清晰易懂。良好的設計 state 會導致你不會一直去使用 this.refs 強迫控制元件，且會讓資料易於控制和正確。
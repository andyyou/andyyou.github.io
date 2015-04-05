---
layout: post
title: 'React 元件運作與生命週期'
date: 2014-02-16 23:15:00
categories: Reactjs
---
# 與瀏覽器之間的運作
React 針對瀏覽器提供了十分強大的抽象化概念，讓你在大部份的情況下不必再直接操作 DOM ，不過有些時候或許還是需要單純的存取底層的 API(DOM API)，可能是使用第三方函式庫或者事已經寫好的程式碼。

# 關於虛擬 DOM
React 快速的原因是因為它從來不直接影響 DOM。React 會負責在記憶體中持續維護一份 DOM 的表現結構。`render()` 方法負責回傳關於 DOM 的描速，React 就能得知其和記憶體中結構的差異，接著他會計算出最快的更新方式然後交給瀏覽器去影響 DOM。
此外，React 完整實作了對應的事件系統，所有物件的事件保證符合 W3C 的規範，且關於事件氣泡傳遞(bubbles)的行為在任何瀏覽器也都一致。甚至可以在 IE8 使用 HTML5 的事件。大多數的時候你的程式操作應該都會 React 所建構的"仿瀏覽器"的世界裡，因為它俱有高效能和相對容易使用。不過有些情況下，你可能會需要存取使用底層基本的 API，例如使用 jQuery 第三方套件，React 提供了一個後門允許你可以直接操作底層的 API。

 # Refs 和 getDOMNode()
為了與瀏覽器互動，你需要使用指向 DOM 節點的參考物件。每一個 `Mounted` 的 React 元件都會有 `getDOMNode()` 的功能 ，你可以透過呼叫它取得該 DOM 的參考物件。

注意：`getDOMNode()` 只能在元件已經掛載完畢時使用(換句話說這表示該物件已經被渲染放置到 DOM 裡了)。如果你嘗試在元件尚未掛載完畢前呼叫這個 API 將會發生例外。

為了取得 React 元件的參考，你可以使用 `this` 取得目前元件或者使用 `refs`，使用 `refs` 則需要設定一個名稱，如下範例：

{% highlight js %}
/** @jsx React.DOM */

var MyComponent = React.createClass({
  handleClick: function() {
    // 透過原生 API 明確的指示 input 為 focus 狀態。
    this.refs.myTextInput.getDOMNode().focus();
  },
  render: function() {
    // ref 屬性替元件增加一個參考然後你就可以在元件掛載完畢後使用 `this.refs`
    return (
      <div>
        <input type="text" ref="myTextInput" />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.handleClick}
        />
      </div>
    );
  }
});

React.renderComponent(
  <MyComponent />,
  document.getElementById('example')
);
{% endhighlight %}

# 更多關於 Refs
想要了解更多關於 `refs` 及如何有效的使用可以參考 [refs 文件](http://facebook.github.io/react/docs/more-about-refs.html)

# 元件的生命週期
元件的生命週期有三個主要的部分：
* `Mounting`：元件正準備要被寫入 DOM
* `Updating`：元件偵測到狀態的改變準備重新渲染。
* `Unmounting`：元件正要被從 DOM 中移除。

React 根據生命週期提供了對應的方法(事件)讓你可以在對應的階段做一些處理。`Will` 的方法用在某些狀況準備發生之前，`Did` 的方法則表示該狀況已經發生後。

# Mounting 掛載流程
* `getInitialState()`：當物件被調用時此方法會在寫入 DOM 之前被觸發，通常用來管理狀態的元件可以用這個方法初始化一些資料。
* `componentWillMount`：當元件內部的結構處理完畢準備寫入 DOM 之前觸發。
* `componentDidMount(DOMElement rootNode)`：當元件被寫入 DOM 之後觸發。當初始化需要操作 DOM 元素就可以用這個方法。

# Updating 更新流程
* `componentWillReceiveProps(nextProps)`：已掛載的元件收到新的 `props` 時被觸發。在這個方法裡你通常會去比較 `this.props` 和 `nextProps` 然後再用 `this.setState` 去改變狀態。
* `shouldComponentUpdate(nextProps, nextState)`：這個函式需要回傳一個布林值，當元件判斷是否需要更新 DOM 時會被觸發。你可以在這個方法裡面去比較 `this.props`，`this.state`，`nextProps`，`nextState` 來決定是否需要更新，回傳 false 則會跳過此次觸發不更新，如果你什麼都不回傳預設會當做 `false` 。
* `componentWillUpdate`：例如在上面 `shouldComponentUpdate` 你回傳了 `true` ，元件確定要更新了，在準備更新前這個方法會被觸發。
* `componentDidupdate(prevProps, prevState, rootNode)`：更新後觸發。

# Unmounting 卸載流程
* `componentWillUnmount()`：當元件準備要被移除或破壞時觸發。

# 掛載後才能使用的方法
* `getDOMNode()`：使用此方法會傳回一個 DOM 元素物件，透過這個方法你可以取得一個參考物件直接操作 DOM 節點。
* `forceUpdate()`：任何已掛載的元件，當你知道元件內部有些狀態已經改變但他不是透過 `this.setState()` 去修改值的時候可以呼叫這個方法強迫更新。

注意：`componentDidMount()` 和 `componentDidUpdate()` 的 `rootNode` 參數只是提供你一個比較方便的方式存取 DOM ，這和使用 `this.getDOMNode()` 是一樣的。
補上一段實作各種方法的範例您可以試著把註解的地方取消看看變化

{% highlight js %}
/** @jsx React.DOM */
var Test = React.createClass({
  getInitialState: function () {
    console.log("> getInitialState()");
    return {user: 'AndyYou'};
  },
  componentWillMount: function () {
    console.log("> componentWillMount()");
  },
  componentDidMount: function (node) {
    console.log("> componentDidMount(node)");
    console.log(node.className);
    console.log(node.value);
    console.log(node.id);
    console.log(this.getDOMNode().className);
    console.log(this.getDOMNode().value);
    console.log(this.getDOMNode().id);
  },
  componentWillReceiveProps: function (nextProps) {
    console.log("> componentWillReceiveProps(nextProps)");
    console.log(nextProps);
  },
  handleChange: function (e) {
    console.log(e.target.value);
    this.setState({user: e.target.value});
  },
  shouldComponentUpdate: function (nextProps, nextState) {
    console.log("> shouldComponentUpdate(nextProps, nextState)");
    console.log("nextProps: ");
    console.log(nextProps);
    console.log("nextState: ");
    console.log(nextState);
    return true; /* need return true/false */
  },
  componentWillUpdate: function (nextProps, nextState) {
    console.log("> componentWillUpdate(nextProps, nextState)");
  },
  componentWillUnmount: function () {
    console.log("> componentWillUnmount()");
  },
  render: function () {
    return (
      <input type='text' id='foobar' value={this.state.user} className='nav' onChange={this.handleChange} />
    );
  }
});
var test = React.renderComponent(<Test title='Untitled' />, document.getElementById('example'));
// test.setProps({title: 'No'});
// React.unmountComponentAtNode(document.getElementById('example'));
// test.setState({user:'Calvert'});
{% endhighlight %}

# 支援的瀏覽器和兼容
在 Facebook 我們支援了包含 IE8 在內的舊瀏覽器，我們已經落實瀏覽器兼容很長一段時間了，這讓我們可以實作出有實用且遠見的 Javascript。這表示我們並沒有太多 Hack 特定瀏覽器產生鬆散的分支代碼，根據這些經驗我們可以確信我們的程式碼在任何瀏覽器都是可以正常運作的。舉個例子常見 `+new Date()` 這種寫法我們會改用 `Date.now()`。
React Open Source 專案和 Facebook 內部使用的的是一樣的，我們已經證實並正在使用。
此外我們並不試圖讓實作兼容功能變成函式庫的一部份，如果每個函式庫都重新實作這些功能，為了支援老舊瀏覽器，你會反覆載入相同功能的程式碼。如果你需要支援舊的瀏覽器，可能你已經在使用 `es5-shim`。

# 兼容
`es5-shim.js` 可以從 [kriskowal's es5-shim ](https://github.com/kriskowal/es5-shim) 取得。下面是 React 支援老舊瀏覽器需要的東西
* Array.isArray
* Array.prototype.forEach
* Array.prototype.indexOf
* Array.prototype.some
* Date.now
* Function.prototype.bind

`es5-sham.js` 也可以從 [kriskowal's es5-shim](https://github.com/kriskowal/es5-shim) 取得，React 需要：
* Object.create

React 的非最小化建置則需要 [paulmillr's console-polyfill](https://github.com/paulmillr/console-polyfill)
* console.l*


# 跨瀏覽器的議題
雖然 React 對於瀏覽器的抽象化過程處理的非常不錯，但是有些瀏覽器的限制或怪異的行為我們還是找不到解法

## IE8 的 onScroll 事件
IE8 的 onScroll 事件不會造成事件的氣泡傳遞而且 IE8 也沒有定義對應的處理事件。目前關於在 IE8 的這個事件已經被忽略了。

> # Polyfills
Polyfilling 是由 RemySharp 所提出的術語，它是用來描速關於複製缺少的 API 和 API 功能的行為。你可以使用它撰寫應用程式的程式碼而不用擔心其他瀏覽器是不是支援。事實上，polyfills 並不是新技術也不是和 HTML5 捆绑到一起的。

> # Polyfills 是什麼？
讓我們直接來看實務 Polyfills 指的是什麼。例如使用 `json2.js` 就是一種 Polyfills。
``` js
if (typeof JSON.parse !=='function') {
  // Crockford’s JavaScript implementation of JSON.parse
}
```
上面這段程式碼表示; 如果瀏覽器本身可以執行 `JSON.parse`，那麼 `json2.js` 就不會重新定義或者干擾 `JSON` 物件。如果沒有原生的 API 可用，`json2.js` 就會執行一段 JavaScript 來實現這個功能，它和原生的 JSON API 是完全兼容的。最終的結果就是你可以在網頁上使用 json2.js 而不用考慮瀏覽器執行的是哪種程式碼。


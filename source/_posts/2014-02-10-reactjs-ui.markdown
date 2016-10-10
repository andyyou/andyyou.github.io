---
layout: post
title: 'React 互動式動態 UI'
date: 2014-02-10 21:43:00
categories: Program
tags: [reactjs]
---
# 互動式動態 UI

你已經學會如果使用 React [呈現資料](http://andyyou.logdown.com/posts/178292-react-displaying-data)了。現在讓我們的界面增加互動的功能。

<!--more-->

# 範例

~~~js
/** @jsx React.DOM */

var LikeButton = React.createClass({
  getInitialState: function() {
    return {liked: false};
  },
  handleClick: function(event) {
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? 'like' : 'unlike';
    return (
      <p onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </p>
    );
  }
});

React.renderComponent(
  <LikeButton />,
  document.getElementById('example')
);
~~~

# 事件處理與合成事件
跟你在 HTML 裡替標簽增加事件處理程序一樣，在 React 中你必須要駝峰式的命名來設定事件。例如：`onClick`。
React 保證實做的合成事件在 IE8 以上瀏覽器裡定義的所有事件行為是一致的。也就是說不管你使用哪種瀏覽器 React 都知道如何按照 Spec 規範傳遞(Bubble)和截取事件，並傳遞給事件處理程序確保一切都和 W3C 規範的一樣。
如果你想使用 React 在觸控裝置上(例如: 智慧型手機和平板)，你可以呼叫 `React.initializeTouchEvents(true)` 開啟他們。

# 表面之下的機制：自訂繫結和委派
React 內部做了一些處理以確保你的程式執行的效能不會太糟和容易理解。

* 自動繫結：在 Javascript 中當一個 `callback` 函式被建立時你通常需要明確的將它跟某個物件的方法關聯在一起使得我們可以確定資料是正確的。在 React ，每一個方法都是自動跟元件綁定。React 會暫存已綁定的方法以增加 CPU 和記憶體的使用效率。
* 事件委派：React 實際上並沒有附加任何事件到元素本身。當 React 啟動時，它會在最上層啟動一個事件監聽器監聽所有的事件。當一個元件載入或取消載入的時候，事件處理程序會自動從內部增加或移除。一旦事件被觸發，React 會知道如何調派以及使用對應的程序，如果裡面沒有對應的事件，React 則不會執行任何動作。如果你想學習關於增加處理速度的知識可以參考[David Walsh's excellent blog post](http://davidwalsh.name/event-delegate)。

# 元件就是[狀態機](http://www.dev.idv.tw/mediawiki/index.php/%E7%8B%80%E6%85%8B%E6%A9%9F%E7%9A%84%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88%E9%A2%A8%E6%A0%BC)
React 認為 UI 就是一個簡單的狀態機。從 UI 的角度思考，UI 本身俱有多種狀態，並且負責把這些狀態輸出呈現。這樣做可以輕易讓你的 UI 保持一致。
在 React 中，你只要單純更新元件的狀態，接著根據狀態渲染輸出新的 UI。React 透過較有效率的方式協助你更新 DOM 。

# 狀態如何運作
比較普遍的方式通知 React 資料已經變動了是透過呼叫 `setState(data, callback)` ，這個方法會把資料 `data` 整合進 `this.state` 接著重新渲染元件。當原件完成這個動作，你可以額外的加入 `callback` 做後續處理，當然也可以不加。大多的狀況你不會需要提供 `callback` 因為 React 會自動及時的更新資料。

# 什麼元件應該有狀態？
大部分的原件都應該會從 `props` 取得一些資料，然後輸出。然而有些時候你還是需要回應使用者的操作，伺服器的請求，或一些隨著時間變化的資料。這個時候我們就會使用狀態 `state` 。
試著讓大多數的元件盡可能沒有狀態。透過這種方式你可以獨立各種狀態的邏輯盡可能減少複雜的邏輯，這會使你的應用程式比較容易理解。
一個比較常見的模式是先建立一些沒有狀態的元件，它們只負責輸出資料，然後在它們的上層有一個負責管理狀態的元件再把狀態資訊透過 `props` 傳給子元素。這個有狀態的元件內部封裝所有邏輯和方法，透過屬性宣告的方式底下的元件只要負責渲染資料。

# 該如何使用狀態？
狀態應該負責管理資料，實務上就是元件內的事件處理程序在資料發生異動的時候被觸發然後更新 UI 。在實際的程式裡這些資料通常是很小的 JSON。當建立一個管理狀態的元件時，盡可能思考最簡化的表達方式，且資料只放在 `this.state` 中。在 `render()` 方法內部則根據這個狀態單純的計算出你需要的資訊就好。
你會發現思考並以這種方式寫程式往往是最正確的。因此在狀態內部增加任何多餘的資料或計算意味著你需要去處理多餘的東西以確保資料是一致的，不要一味的依賴 React 計算處理資料。

# 狀態不應該這樣用？
`this.state` 應該只存放 UI 需要呈現的資料，不應該包含：
* 已處理完成的資料：不用預先計算處理狀態的資料。在 `render()` 裡面計算資料是比較容易確保 UI 的資料是正確的。舉例來說如果有一個項目清單的陣列，你想要輸出清單數量，應該單純的在 `render()` 裡面使用 `this.state.listItems.length + ' list items'` 而不是把值直接存在 `this.state`。
* React 元件：在 `render()` 裡面根據 `props` `state` 去建置。
* 重複存取在 `props` 裡的資料：`props` 應該是正確資料的來源，因為 `props` 可能隨著時間改變。適當的把 `props` 存在 `state` 可以協助我們取得之前的資料。

> 補充說明：
在上面的文章中我們了解到通常會使用一個主要的元件負責管理狀態，而其他子元件需要的時候則直接使用主元件的 `props` 和方法，下面的補充範例我們將簡單的示範一些關於使用事件的方式：

~~~js
/** @jsx React.DOM */
/* 建立一個 Clicker 元件類別 */
var Clicker = React.createClass({
  /* 透過 render() 輸出三個超連結並使用主元件的事件。 */
  render: function () {
    var bind = this.handleBind.bind(this);
    return (
      <div>
        { /* 下面這個超連結示範了一般使用主元件的方法  */}
        <a onClick={this.handleNormal}>Normal</a>
        { /* 使用 bind() 的目的是為了在 function 內再使用主元件的函式 */}
        <a onClick={bind}>Bind</a>
        { /* 在 0.4.0 版之後為了簡化程式碼，預設就是 autoBind。 */}
        <a onClick={this.autoBindClick}>AutoBind</a>
      </div>
    );
  },
  handleNormal: function (event) {
    alert('Normal Event');
  },
  handleBind: function (event) {
    alert(this.ALERT_BIND); /* 取用主元件的函式 */
  },
  autoBindClick: function (event) {
    alert(this.ALERT_AUTOBIND);
	},
  ALERT_BIND: 'Bind Event',
  ALERT_AUTOBIND: 'Auto Bind Event'
  // oldAutoBindClick: React.autoBind(function () {...})
});
React.renderComponent(<Clicker />, document.getElementById('example'));
~~~

# 其他資訊
[React.autoBind() 移除](https://github.com/mcsheffrey/react-tutorial/commit/2e0d9f2de28f713c9a47455c49f8ddc3576bdcf6)
[Javascript bind()](http://blog.csdn.net/qy1387/article/details/7854589)

---
layout: post
title: 'React 關於表單'
date: 2014-02-15 01:00:00
categories: Program
tags: reactjs
---

# 表單

表單元件像是 `<input>`，`<textarea>`和`<option>`和其他元件有些不同，因為他們可以被使用者操作而產生變化。這些元件提供一個介面好讓我們以表單的形式和使用者產生互動。

<!--more-->

# Props 的輸入與輸出
表單元件提供一些 `props` 會根據使用者的操作影響屬性值和呈現。
* `value` ： `<input>` `<textarea>` 支援。
* `checked` ： `checkbox`，`radio` 支援。
* `selected`：`<option>` 支援。

在 HTML 裡面 `<textarea>` 的 value 是嵌在子元素就是像這樣 `<textarea>value here</textarea>`，但是在 React 你可以用 value 取代。
表單元件透過 `onChange` 設定 callback 可以達到監聽的功能，就是一旦這些值改變了就會觸發 `onChange`。`onChange` 會在使用者有以下狀況時被觸發：
* `<input>` 或 `<textarea>` 的 value 改變時。
* `checked` 改變時。
* `selected` 改變時。

就像所有的 DOM 事件，`onChange` 支持所有原生元件且可以監聽到因氣泡傳遞產生的觸發。

# 元件約束
一個 `<input>` 一旦設定了 `value` 就是一個約束的元件。輸出後元素 `<input>` 的值將會永遠映射到 `this.props.value` 舉例來說

~~~js
/** @jsx React.DOM */
var Test = React.createClass({
  render: function () {
    return (
      <input type='text' value='Hello!' />
    );
  }
});
React.renderComponent(<Test />, document.getElementById('example'));
~~~

`value` 一旦設定，你會發現不能改變了，像上面範例 value 永遠等於 Hello! 。任何輸入都無法改變值，因為 React  已經定義 `value` 是 `Hello!`，如果你想要讓 `<input>` 可以被使用者操作你應該使用 `onChange` 事件：

~~~js
/** @jsx React.DOM */
var Test = React.createClass({
  getInitialState: function () {
    return {value: 'Hello!'}
  },
  handleChange: function (event) {
    this.setState({value: event.target.value});
  },
  render: function () {
    // var value = this.state.value;
    return (
      <input type='text' value={this.state.value} onChange={this.handleChange} />
    );
  }
});
~~~

讓我們整理一下你在目前學習過程可能會產生的疑惑，就是 `props` 和 `state` 的差異，首先是 `props` 不應該由元件本身去操作異動，應該只能夠在使用元件的時候當作帶入的參數(雖然的確是可以使用 `setProps()` 去設定，不過這違反官方的設計模式)。接著的工作就是負責傳遞資料。
而 `this.state` 可以當作參數傳遞資料也可以在元件內部去設置改變。換句話說，你應該只能在元件內部呼叫 `setState()` ，常見的應用都是用 `this.state` 處理關於使用者操作互動產生的結果。此外要注意的是如果你真的在子元件也使用了 `state` 那它跟主元件本身的狀態是分開獨立的。
另一個問題是為什麼 `props` 可以用 `propTypes` 驗證，而 `state` 沒有，主要是因為關於 `state` 完全是由元件設計者控制的，想像一下一般的情況下你設計了一個元件，而當其他人要使用的時候他應該只需要使用 `React.renderComponent(<YourComponent />, document.getElementById('example'))` 輸出元件即可，根據你提供的文件，它可以在屬性設定自己的參數 `<YourComponent name="Andy" />` ，而 `this.state` 根本沒機會外露(修改元件除外)。所以如果你真的需要驗證 `state` 的時候就在適合的生命週期事件中處理就好了，例如 `componentWillUpdate` 事件。
根據上面這些說明，我們得到結論：就是一旦 `<input>` 的 `value` 被綁定就不會變動了，我們稱為元件約束，所以應該在 `value` 綁入一個變數，而這個變數按照模式的規劃應該使用 `this.state` 取得值是比較正確的。

~~~js
handleChange: function(event) {
    this.setState({value: event.target.value.substr(0, 140)});
  }
~~~

修改 `handleChange()` 範例就可以實作限制在 140 個字元。

# 不被約束的元件
如果一個 `<input>` 沒有設定 `value` 那它就是一個不被約束的元件。此時 input 的 value 就會是使用者輸入的值。

~~~js
  render: function() {
    return <input type="text" />;
  }
~~~

上面這段程式碼並沒有設定 value ，使用者輸入的任何值都會立刻反應。想要在使用者改變欄位值的同時做些處理可以使用 `onChange` 。
如果你想在初始化的時候不要帶入空值，且可以隨著使用者操作回饋，你可以使用 `defaultValue`。

~~~js
render: function() {
  return <input type="text" defaultValue="Hello!" />;
}
~~~

其他類似的屬性像是`<input>` 還有 `defaultChecked`，`<select>` 也有 `defaultValue` 。

# 進階議題
## 為什麼要約束元件？
React 在設計像是 `<input>` 這類表單元件時面臨一個表現時的挑戰。舉例來說在 HTML

~~~html
  <input type="text" name="title" value="Untitled" />
~~~

在傳統 HTML 是表示初始化值設定為 `Untitled` 當使用者更新欄位時，元素取得的屬性將會改變，然而如果你使用 `node.getAttribute('value')` 其實他還是會返回 `Untitled`。讓我們直接看下面這段原始碼

~~~html
<html>
  <head>
    <title>Hello React</title>
    <script>
    var val1, val2;
    function Log () {
      console.log(this);
      console.log(this.getAttribute('value'));
      console.log(document.getElementsByName(this.name)[0].value);
      console.log(this.value);
    }
    </script>
  </head>
  <body>
    <input type='text' name='i1' value='Untitled' onchange="Log.apply(this)" />
  </body>
</html>
~~~

試著輸入值觀察 console 。注意：HTML 的 `onchange` 行為是在你改變了值，滑鼠移出欄位之時才觸發。

React 是透過 `<input value=' '/>` 的 `value` 影響元素，但是當我們改變欄位的時候卻只能從 `node.value` 而不是 `node.getAttribute('value')` 取得使用者改變的資料，所以如果要把這個值表現在元素上就只能把這個 `node.value` 放到 state 變數，然後屬性就使用 `this.state.value`。這也就是為什麼需要約束元件。
也因此當我們如下程式碼的時候

~~~js
render: function() {
  return <input type="text" name="title" value="Untitled" />;
}
~~~

因為我們接不到 `event.target.value` 所以 React 會一直保持 `Untitled`。

## 關於 Textarea
在 HTML一般設定 `<textarea>` 是用子元素去設定

~~~html
<textarea name="description">This is the description.</textarea>
~~~

開發者可以很輕易的使用多行的內容，但因為 React 是 Javascript 所以當你想要換行的時候可以使用 `\n`。

先看下面這段程式碼

~~~js
/** @jsx React.DOM */
var Textarea = React.createClass({
  render: function () {
    return (
      <textarea value='value' defaultValue='default'>This is dog</textarea>
    );
  }
});
React.renderComponent(<Textarea />, document.getElementById('example'));
~~~

其實我們這三個設定都可以用，但是子元素會等於 `defaultValue` 所以當子元素和 `defaultValue` 都用的時候會產生錯誤 `If you supply `defaultValue` on a <textarea>, do not pass children.` 而 `value` 會覆寫 `defaultValue` 。為了避免混淆我們一般建議只用 `value`，接下來的用法就和上面提到的一樣。

## 關於 Select
一般我們使用 `<select>` 要指定選項是透過在 `<option selcted>`，在 React 為了讓元件方便操作我們改用下面的方式

~~~js
/** @jsx React.DOM */
var Dropdown = React.createClass({
  render: function () {
    return (
      <select value={this.props.selected}>
        <option value="A">Apple</option>
        <option value="B">Banana</option>
        <option value="C">Cranberry</option>
      </select>
    );
  }
});
React.renderComponent(<Dropdown selected='C' />, document.getElementById('example'));
~~~

如果要是預設值也是用 `defaultValue` 如下

~~~js
/** @jsx React.DOM */
var Dropdown = React.createClass({
  render: function () {
    return (
      <select value={this.props.selected} defaultValue="B">
        <option value="A">Apple</option>
        <option value="B">Banana</option>
        <option value="C">Cranberry</option>
      </select>
    );
  }
});
React.renderComponent(<Dropdown />, document.getElementById('example'));
~~~

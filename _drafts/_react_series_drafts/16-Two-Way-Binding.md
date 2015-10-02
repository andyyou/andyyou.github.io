## 雙向繫結(Two-way binding)輔助函式
`React` 可以搭配 `ReactLink` 是一種實作雙向資料繫結較簡單的方式。

> 注意: 如果您剛開始使用這個 framework ，`ReactLink` 並不是必須的東西，且應該謹慎使用。

在 React 中，資料流遵循單一方向: 從上而下，從父元件到子元件。這是因為在[馮‧紐曼架構](http://zh.wikipedia.org/wiki/%E5%86%AF%C2%B7%E8%AF%BA%E4%BC%8A%E6%9B%BC%E7%BB%93%E6%9E%84)下資料流通常一次只有一個方向。
您可以單純地就把上面提的這些說名歸納為一個 "one-way data binding" 的機制。
在實務上說明就是資料都是從 `this.state` 或 `this.props` 來的。

由於大部份應用程式的需求都是關於取得資料並在程式中操作這些資料，或者抽象的說需要讓資料流回程式中。
什麼意思？舉例來說:
當我們開發了一個表單 `form` 當表單每次收到使用者變更的資料後您就需要更新 React 的 `state`。(取得 form 資料 -> 更新 state)
又或者你希望在 Javascript 和 React 中調整佈局，改變某些 DOM 的尺寸大小。(取得關於 Size -> 調整佈局)

要完成上面這些任務，透過做法是透過監聽 `change` 事件，取得資料(這些資料通常來自 DOM)，接著執行 `setState()` 。
封閉的資料流可以帶來一些好處如: 明確易懂，便於維護。
用實際的範例來說: 只要你能確定資料來源永遠是 `this.state` ，而每一階層需要資料的元件都是跟 `state` 取得，如此一來只要變更 `state` 其他的元件就應該要自己負責更新資料。

Two-way binding 雙向的資料繫結 -- 由於很多時候 DOM 上顯示的值必須和某個 `state` 中的值一致，所以所謂的 Two-way binding 即程式在背後自動幫您同步這兩個值。
改變 form 時 model 會修正，更新 model 後 form 的 UI 也會跟著更新。
使用 `Two-way binding` 的好處是程式碼變得簡潔並適用于多種應用。
根據上述的這些情況 React 提供了 `ReactLink`: 這個語法糖衣讓您可以輕鬆完成這樣的功能，或者我們可以說 `ReactLink`
就是幫您`連結`資料來源到 React 的 `state`。

> 注意: ReactLink 只不過幫您把 `onChange/setState()` 這兩件事包起來做成一個簡單的慣例。它並不會修改底層運行的模式和資料流。

## ReactLink: Before 和 After
這邊我們先不使用 `ReactLink` 示範一個簡單的表單範例:

```js
/**
  * @jsx React.DOM
  */
var NoLink = React.createClass({
  getInitialState: function () {
    return { message: 'Hello!' }
  },
  handleChange: function (event) {
    console.log(event.target.value);
    this.setState({message: event.target.value});
  },
  render: function () {
    var message = this.state.message;
    return <input type='text' value={message} onChange={this.handleChange} />;
  }
});

React.renderComponent(
  <NoLink />,
  document.getElementById('example')
);
```

當然這個部分運作的非常良好，且關於資料的運作與流向非常清楚。不過你也發現了，一旦欄位增加程式碼就會變的有些冗長。
接著我們換成使用 `ReactLink`:

```js
var WithLink = React.createClass({
    /* 載入 React.addons.LinkedStateMixin 中的方法 (this.linkState)*/
    mixins: [React.addons.LinkedStateMixin],
    getInitialState: function () {
      return {message: 'Hello!'}
    },
    render: function () {
      return (
        <div>
          <input type='text' valueLink={this.linkState('message')} />
          {this.state.message}
        </div>

      )
    }
});
```
`LinkedStateMixin` 加入了一個叫 `linkState()` 的方法到 React 元件中。然後 `linkState()` 的功能是回傳了一個 `ReactLink` 物件，其實這個物件不過就是包含著 `state` 的值和實做一個 `onChange` 的 callback。

`ReactLink` 物件就像 `props` 一樣，會被傳到整個階層樹狀結構中，因此您可以輕易的設定任何值的 two-way binding 。

值得提醒的是關於 `checkbox` 的 `value` 屬性有比較特殊的行為，只有在 checkbox 被勾選，表單提交時才會送出 `value` (屬性預設會是 `on`)。
關於 `value` 屬性值是不會改變的不管是勾選或不選。針對 checkbox ，您應該改用 `checkedLink` 而不是 `valueLink`:

```js
<input type="checkbox" checkedLink={this.linkState('booleanValue')} />
```
總結來說我們就只是透過 `valueLink` 搭配 `ReactLink` 物件來完成 two-way binding 這件事。


## 關於內部運作
在這一小節我們將要探討關於 `ReactLink` 的兩個面向: 建立 `ReactLink` 物件與如何使用。
為了證明 `ReactLink` 很簡單，我們將實作一些程式碼範例讓您更加清楚其運作。

### ReactLink 不使用 LinkedStateMixin

```js
/**
 * @jsx React.DOM
 */

var WithoutMixin = React.createClass({
  getInitialState: function () {
    return {message: 'Hello!'}
  },
  handleChange: function (newValue) {
    this.setState({message: newValue})
  },
  render: function () {
    var valueLink = {
      value: this.state.message,
      requestChange: this.handleChange
    };
    return (
      <div>
        <input type='text' valueLink={valueLink} />
        {this.state.message}
      </div>
    )
  }
});

React.renderComponent(
  <WithoutMixin />,
  document.getElementById("example")
)
```
誠如您所見，即使不使用 `addons` 提供的輔助函式我們一樣可以輕鬆使用 `ReactLink`。
`ReactLink` 只是一個非常單純的物件，它就只是包含 `value` 和 `requestChange` 這兩個屬性。同樣的 `LinkedStateMixin` 也很單純，就只是根據您傳入的參數，
決定存取哪一個 `this.state` 和幫您實作一個使用 `this.setState()` 的方法。
最後我們透過 `valueLink` 或 `checkedLink` 告訴 React 這個元素的值要使用 `ReactLink` 物件，請在適當的時候幫我呼叫裡面的 callback 。

> 觀念上我們可以歸納為: React 本身只有 one-way data binding 的機制，即資料流都是遵從單一方向。而 two-way binding 的機制只不過是簡化 onChange/setState 這兩件事。


### ReactLink 不使用 valueLink

```js
/**
 * @jsx React.DOM
 */

var WithoutLink = React.createClass({
  mixins: [React.addons.LinkedStateMixin],
  getInitialState: function () {
    return {message: 'Hello!'}
  },
  render: function () {
    var valueLink = this.linkState('message');
    var handleChange = function (e) {
      valueLink.requestChange(e.target.value);
    };
    return (
      <div>
        <input type='text' value={valueLink.value} onChange={handleChange} />
        {this.state.message}
      </div>
    )
  }
});

React.renderComponent(
  <WithoutLink />,
  document.getElementById('example')
);
```

關於 `valueLink` 屬性也相當單純，它只是簡單的處理 `onChange` 事件以及呼叫 `this.props.valueLink.requestChange()` 接著使用 `this.props.valueLink.value` 取代原本的值。

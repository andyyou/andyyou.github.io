# 空值(null)的控制輸入
通常我們會在 value 的屬性是上設定值，一旦綁定值我們稱為綁定元件。這麼做可以防止使用者隨意改變 value 除非您希望這麼做。
我們在 Form 的章節討論過，React 一旦綁定值之後您就必須透過事件去修改內部的 value 參考。
您可以遇過下面這樣的問題就是已經綁定值了，但是 input 還是可以改變，這種時候要注意，你可能不小心設定值為未定義或為空。

```js
/** @jsx React.DOM */

React.renderComponent(<input value="hi" />, mountNode);

setTimeout(function() {
  React.renderComponent(<input value={null} />, mountNode);
}, 1000);
```

> 官方的的範例可能無法清楚說明狀況讓我們在看看下面的範例:
```js
/**
 * @jsx React.DOM
 */
var What = React.createClass({
  _onClick: function () {
    this.setProps({v: null});
  },
  render: function () {
    return (
      <div>
      <input value={this.props.v} />
      <input type='button' value="Before Click you can't change value" onClick={this._onClick} />
      </div>
    )
  }
});

React.renderComponent(
  <What v="Andy"/>,
  document.getElementById('example')
);
```

# componentWillReceiveProps 在掛載後不會被觸發
componentWillReceiveProps 在節點掛載後不會被觸發。請參閱[生命週期](http://facebook.github.io/react/docs/component-specs.html)確認您所需要的事件。
為什麼掛載後無法收到新的 props 這是因為 componentWillReceiveProps 通常是在 render 之前拿來處理比較新舊 props，所以在一但 render 開始執行，或初始化時並不會被觸發

# 在 getInitialState 使用 Props 是一種反面模式(Anti-Pattern)
反面模式簡單的說就是一種不好的做法，props 通常來自父元件，而為了初始化狀態常導致我們去複製"這個資料來源"至 state 。
這邊我們得提醒只要有可能，儘量讓值是動態計算以確保資料同步差異。讓我們看看範例以理解各種情形該怎麼做

不好的範例:
```js
/** @jsx React.DOM */

var MessageBox = React.createClass({
  getInitialState: function() {
    return {nameWithQualifier: 'Mr. ' + this.props.name};
  },

  render: function() {
    return <div>{this.state.nameWithQualifier}</div>;
  }
});

React.renderComponent(<MessageBox name="Rogers"/>, mountNode);
```
上面就是說，我們不該把這樣的簡單計算又在 state 存一份。正確的做法應該如下:
```js
/** @jsx React.DOM */

var MessageBox = React.createClass({
  render: function() {
    return <div>{'Mr. ' + this.props.name}</div>;
  }
});

React.renderComponent(<MessageBox name="Rogers"/>, mountNode);
```
如果邏輯更複雜則您應該把他獨立成一個 method

然而如果是像下面這樣，這就不是一個反面模式了，如果您能夠清楚分離目的，一樣的資料同步不應該是最終目標:
```js
/** @jsx React.DOM */

var Counter = React.createClass({
  getInitialState: function() {
    // naming it initialX clearly indicates that the only purpose
    // of the passed down prop is to initialize something internally
    return {count: this.props.initialCount};
  },

  handleClick: function() {
    this.setState({count: this.state.count + 1});
  },

  render: function() {
    return <div onClick={this.handleClick}>{this.state.count}</div>;
  }
});

React.renderComponent(<Counter initialCount={7}/>, mountNode);
```
count 和 initialCount 最終是兩個意義不同的值。

# 在元件中使用 DOM 事件監聽
注意: 這裡指的是附加原本的 DOM 事件而不是 React 提供的模擬事件機制。通常我們在和 jQuery 整合的時候會有這個需求。

看看下面這段調整 window 尺寸的範例碼:
```js
/**
 * @jsx React.DOM
 */
var Box = React.createClass({
  getInitialState: function () {
    return {
      windowWidth: window.innerWidth
    };
  },
  handleResize: function (e) {
    this.setState({windowWidth: window.innerWidth})
  },
  componentDidMount: function () {
    window.addEventListener('resize', this.handleResize);
  },
  componentWillUnmount: function () {
    window.removeEventListener('resize', this.handleResize);
  },
  render: function () {
    return (
      <div>
        Current: window width: {this.state.windowWidth}
      </div>
    )
  }
});

React.renderComponent(
  <Box />,
  document.getElementById('example')
);
```
`componentDidMount` 會在元件掛載後被執行，在掛載後才會有 DOM 元素，此時我們才能附加一般的 DOM 事件上去。
注意關於事件的 callback 是綁定在元件而不是元素，React 會自動把方法綁定到當前的元件實例物件。

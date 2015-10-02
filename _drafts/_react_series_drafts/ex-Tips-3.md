# 透過 AJAX 載入初始化資料
在 `componentDidMount` 截取下載資料。當回應傳回時，儲存資料在 state ，接著觸發 render 更新介面。
當一個非同步的請求 (request) 的回應 (response) 正在進行處理時，我們可以在更新 state 之前透過 this.isMounted() 來檢查確保元件已經被掛載。
下面的範例是我們從 Gist 取得資料:

```js
/** @jsx React.DOM */

var UserGist = React.createClass({
  getInitialState: function() {
    return {
      username: '',
      lastGistUrl: ''
    };
  },

  componentDidMount: function() {
    $.get(this.props.source, function(result) {
      var lastGist = result[0];
      if (this.isMounted()) {
        this.setState({
          username: lastGist.owner.login,
          lastGistUrl: lastGist.html_url
        });
      }
    }.bind(this));
  },

  render: function() {
    return (
      <div>
        {this.state.username}'s last gist is
        <a href={this.state.lastGistUrl}>here</a>.
      </div>
    );
  }
});

React.renderComponent(
  <UserGist source="https://api.github.com/users/octocat/gists" />,
  mountNode
);
```

# 在 JSX 中的 false
這邊我們要特別提到關於 false 會根據不同的環境因素有所變化，或說因為上下文的不同:
下面我們列出各種不同情況的 false


```js
/** @jsx React.DOM */
React.renderComponent(<div id={false} />, mountNode);
```

```js
/** @jsx React.DOM */
React.renderComponent(<input value={false} />, mountNode);
```

```js
/** @jsx React.DOM */
React.renderComponent(<div>{false}</div>, mountNode);
```

其差別如下
![](http://i.imgur.com/j0QGmL8.png)

最重要的應該是最後一個當 children 為 false 是不輸出任何東西，其原因是我們在實務上蠻長遇到類似這種用法
```
<div>{x > 1 && 'You have more than one item'}</div>
```

# 元件之間的互動
在處理主從關係或者說父元件和子元件關係的溝通時，我們可以很單純的使用 props 來傳遞。(這邊指的是從上而下)
但如果反過來子-父溝通: 假如現在您的 GroceryList 雜貨清單元件有一些項目地的列表，他們是透過一個陣列產生的。當項目被點擊的時候，你希望可以顯示名稱:
```js
/** @jsx React.DOM */

var GroceryList = React.createClass({
  handleClick: function(i) {
    console.log('You clicked: ' + this.props.items[i]);
  },

  render: function() {
    return (
      <div>
        {this.props.items.map(function(item, i) {
          return (
            <div onClick={this.handleClick.bind(this, i)} key={i}>{item}</div>
          );
        }, this)}
      </div>
    );
  }
});

React.renderComponent(
  <GroceryList items={['Apple', 'Banana', 'Cranberry']} />, mountNode
);
```

注意到我們使用 `bind(this, arg1, arg2, ...)` 我們很單純的把其他參數傳到 `handleClick` ， OK 就是這樣用，這並不是 React 特有的概念或方法，就只是 Javascript。
而為了讓彼此沒任何關聯的元件溝通，您可以設定一個自定的全域事件系統，然後在 `componentDidMount()` 訂閱，在 `componentWillUnmount()` 解除訂閱，接著當事件被觸發時呼叫 setState()
這有點類似 Notificiation 的機制，實作方面可以先參考[這篇](http://blog.caesarchi.com/2011/06/nodejs-eventemitter.html)

# 將元件函式開放
這裡提供另外一種方式(不常用)來讓元件之間互相溝通: 開放子元件的方法好讓父元件呼叫

假設有一個 todos 的列表，在上面點一下會被移除。如果只剩下一個未完成的 todo 那就執行動畫
```js
/** @jsx React.DOM */

var Todo = React.createClass({
  render: function() {
    return <div onClick={this.props.onClick}>{this.props.title}</div>;
  },

  //this component will be accessed by the parent through the `ref` attribute
  animate: function() {
    console.log('Pretend %s is animating', this.props.title);
  }
});

var Todos = React.createClass({
  getInitialState: function() {
    return {items: ['Apple', 'Banana', 'Cranberry']};
  },

  handleClick: function(index) {
    var items = this.state.items.filter(function(item, i) {
      return index !== i;
    });
    this.setState({items: items}, function() {
      if (items.length === 1) {
        this.refs.item0.animate();
      }
    }.bind(this));
  },

  render: function() {
    return (
      <div>
        {this.state.items.map(function(item, i) {
          var boundClick = this.handleClick.bind(this, i);
          return (
            <Todo onClick={boundClick} key={i} title={item} ref={'item' + i} />
          );
        }, this)}
      </div>
    );
  }
});
```

# 元件的參考
如果您正在使用 React 元件到一個大型未使用 React 的程式中或者正在轉移重寫，您可能會需要保留元件的參考。 React.renderComponent 會回傳一個已掛載元件的參考
```js
var myComponent = React.renderComponent(<MyComponent />, myContainer);
```
務必注意。元件的建構子並沒有回傳物件實例，就只是 `descriptor` 就是我們一開始說的標記物件，一個輕量化用來告訴 React 該輸出什麼 DOM 結構的描述物件

```js
/** @jsx React.DOM */

var myComponent = <MyComponent />; // This is just a descriptor.

// Some code here...

myComponent = React.renderComponent(myComponent, myContainer);

```

# this.props.children undefined
簡單的說您不能透過 this.props.children 存取元件結構中的子元素，直接看程式碼理解

```js
/** @jsx React.DOM */

var App = React.createClass({
  componentDidMount: function() {
    // This doesn't refer to the `span`s! It refers to the children between
    // last line's `<App></App>`, which are undefined.
    console.log(this.props.children);
  },

  render: function() {
    return <div><span/><span/></div>;
  }
});

React.renderComponent(<App></App>, mountNode);
```

以上這些注意事項應該能夠解決大部份開發時遇到的問題。

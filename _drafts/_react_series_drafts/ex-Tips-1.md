# 其他備註(1)

## 介紹
關於 React 其他備註的部分我們提供一些額外的資訊，它可能解答在實務上遇到的問題和提醒您一些常見的陷阱。

## 協作
歡迎您提交 pull request 到 [React repository](https://github.com/facebook/react) 並根據 [文擋](https://github.com/facebook/react/tree/master/docs) 的格式。
如果您認為提交內容可能需要檢討，在您提交 pull requrest 之前，您可以在 freenode #reactjs 頻道或者 Google 群組找到協助。


# 行內格式(Inline Styles)
在 React，CSS 的元素行內格式並不需要使用字串，取而代之的是我們可以用一個物件來設定。不過要注意屬性的風格換成了 `camelCased` 且值的部分通常是字串:

```js
/** @jsx React.DOM */

var divStyle = {
  color: 'white',
  backgroundImage: 'url(' + imgUrl + ')',
  WebkitTransition: 'all' // 注意大寫 'W'
};

React.renderComponent(<div style={divStyle}>Hello World!</div>, mountNode);
```

編碼風格使用 camelCased 這樣就會跟從 JS 存取 DOM 屬性一致(例如: node.style.backgroundImage)，上面程式碼中有一個例外，關於瀏覽器的 CSS 前綴字必須用大寫，這也是為什麼 `WebkitTransition`
使用 W 。

## 在 JSX 中使用 If-Else
 `if-else` 語法並不能在 JSX 中使用。這是因為 JSX 只是一種語法糖衣，最終的目標只是把這些 tag 轉換成 JS 物件來呼叫，看看下方範例:

```js
/** @jsx React.DOM */

// This JSX:
React.renderComponent(<div id="msg">Hello World!</div>, mountNode);

// Is transformed to this JS:
React.renderComponent(React.DOM.div({id:"msg"}, "Hello World!"), mountNode);
```

這同時意味著 `if` 語法並不適用于此，看看下面範例:

```js
/** @jsx React.DOM */

// This JSX:
<div id={if (condition) { 'msg' }}>Hello World!</div>

// Is transformed to this JS:
React.DOM.div({id: if (condition) { 'msg' }}, "Hello World!");
```
上面這樣並不是合法的 JS，接著您可能會想使用三元表達式:

```js
/** @jsx React.DOM */

React.renderComponent(<div id={condition ? 'msg' : ''}>Hello World!</div>, mountNode);

```

如果三元表達式不夠用您可以用下面這種方式使用 if

```js
/** @jsx React.DOM */

var loginButton;
if (loggedIn) {
  loginButton = <LogoutButton />;
} else {
  loginButton = <LoginButton />;
}

return (
  <nav>
    <Home />
    {loginButton}
  </nav>
)
```

# 自我結尾標簽(Self-Closing Tag)
在 JSX ， `<MyComponent />` 單獨出現是合法有效的，但 `<MyComponet>` 則否。
每一個元件標簽都需要結尾 `/` 包含`<div />` `<div></div>` 這些都是。

# JSX 根節點的最大數量
目前為止，一個元件的 `render` 您只能回傳一個元素節點，如果您想要回傳一個列表的 div 那你就必須把它們全部包在一個 div，span 之類的元素或者其他元件裏面。
不要忘記 JSX 會編譯成一般 JS 如果回傳兩個 function 語法就會有錯誤，同樣的不要放超過一個子元素在三元表示式中。

# 關於樣式屬性的 px 縮寫
當我們需要設定 px 值在行內樣式的屬性時，React 會自動幫您加上 "px"
```js
/** @jsx React.DOM */

var divStyle = {height: 10}; // 輸出 "height:10px"
React.renderComponent(<div style={divStyle}>Hello World!</div>, mountNode);
```

但是有些 CSS 屬性不需要單位，下列這些屬性並不會自定附加 px 後綴字。


    columnCount
    fillOpacity
    flex
    flexGrow
    flexShrink
    fontWeight
    lineClamp
    lineHeight
    opacity
    order
    orphans
    widows
    zIndex
    zoom

# 子元件屬性的型別
通常一個元件的子元件(this.props.children)是一個元件的陣列:

```js
/** @jsx React.DOM */

var GenericWrapper = React.createClass({
  componentDidMount: function() {
    console.log(Array.isArray(this.props.children)); // => true
  },

  render: function() {
    return <div />;
  }
});

React.renderComponent(
  <GenericWrapper><span/><span/><span/></GenericWrapper>,
  mountNode
);
```

然而當只有一個子元件時，this.props.children 將會只剩單一的元件，就不再會有陣列來包住它。

```js
/** @jsx React.DOM */

var GenericWrapper = React.createClass({
  componentDidMount: function() {
    console.log(Array.isArray(this.props.children)); // => false

    // warning: yields 5 for length of the string 'hello', not 1 for the
    // length of the non-existant array wrapper!
    console.log(this.props.children.length);
  },

  render: function() {
    return <div />;
  }
});

React.renderComponent(<GenericWrapper>hello</GenericWrapper>, mountNode);
```

為了可以輕鬆地處理關於 `this.props.children` 地相關任務，官方也提供了 React.Children 輔助類別。

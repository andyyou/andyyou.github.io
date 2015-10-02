# 深入 JSX
JSX 是一種在 Javascript 中使用的 XML 語法，目的是用來轉換成原生的 Javascript。React 官方推薦使用。

> 是為了簡化類似 OOP 實例化多層架構物件。舉個例子像下面的虛擬碼
```js
List l = new List();
Item i1 = new Item();
Item i2 = new Item();
l.Add(i1);
i.Add(i2);
```
這樣如果情況更複雜的時候會很難維護，換個方式如果是用 XML 的方式
```html
<List>
  <Item name='i1' />
  <Item name='i2' />
</List>
```
感覺會比較好維護。 JSX 就是把下面的 XML 語法轉換成 Javascript 的一種工具。

注意: 不要忘記在程式碼開頭的 `/** @jsx React.DOM */` 他不是一般的註解，這是告訴 JSX 編譯器要編譯這個檔案給 React 使用。
如果你沒有加入這段編譯指示，程式將不會被編譯。因此您可以安心的使用 `JSX transformer`，因為它並不會隨便編譯其他的 Javascript 檔案。

> 透過觀察下面這段簡單的程式碼，我們試著把 `/** @jsx React.DOM */` 移除。會發現整段 `<script type='text/jsx'>` 都不會執行。其他功能都不會被影響。所以在已有的專案裡面使用 React ，就算他出錯了基本上也不會導致其他功能異常。
```html
<!-- ex-1.html -->
<html>
  <head>
    <title>Hello React</title>
    <script src="http://fb.me/react-0.8.0.js"></script>
    <script src="http://fb.me/JSXTransformer-0.8.0.js"></script>
    <script src="http://code.jquery.com/jquery-1.10.0.min.js"></script>
  </head>
  <body>
    <div id='content'></div>
    <h1>Static H1</h1>
    <script>
    $(function () {
      $('h1').click(function () {
        console.log('h1 click');
      });
    });
    </script>
    <script type='text/jsx'>
      /** @jsx React.DOM */
      console.log('jsx');
      var HelloWorld = React.createClass({
        render: function () {
          return (
            <div>Hello World</div>
          )
        }
      });
      React.renderComponent(<HelloWorld />, document.getElementById('content'));
    </script>
  </body>
</html>
```

# 為什麼使用 JSX ?
JSX 並不是強迫使用的，在不使用 JSX 的情況下，我們就得使用 `React.DOM` 提供的函式來建立註記標簽。[上一篇](http://andyyou.logdown.com/posts/178292-react-displaying-data)我們提到 React 除非必要，平常不會直接操作 DOM 元素。它採用在內部模擬 DOM 來進行比較，計算如何有效率的操作 DOM 的方式在運作。使用 JSX 的最終目的是要幫助你用 XML 的編排方式轉成 `React.DOM` 的語法。
例如我們想建立一個超連結
```
var link = React.DOM.a({href: 'http://facebook.github.io/react'}, 'React');
```

> 如果情況在複雜一點可以比較一下下面的程式碼:
```
/* JSX*/
React.createClass({
  render: function () {
    return (
      <div>
        <h1>Title</h1>
        <a href='http://andyyou.logdown.com/'>AndyYou Blog</a>
      </div>
    )
  }
})
/* Origin */
React.createClass({
  render: function () {
    return (
      React.DOM.div(null,
        React.DOM.h1(null, "Title"),
        React.DOM.a( {href:"http://andyyou.logdown.com/"}, "AndyYou Blog")
      )
    )
  }
})
```

官方推薦使用 JSX 的理由如下:
* 可以輕易的檢視整個 DOM 的結構，就跟你使用 HTML 一樣。
* 方便維護修改。
* 概念上非常相似于 `MXML` 和 `XAML`。

# 關於轉換
JSX 的目的是從類似 XML 的語法轉換成原生的 Javascript。XML 標簽元素和屬性會被轉換成 function 和物件。看看下面的虛擬碼

```js
var Nav;
// 編譯前 JSX
var app = <Nav color='blue' />
// 編譯後 JS
var app = Nav({color:'blue'})
```

注意: 為了能正確的使用 `<Nav />`，`Nav` 變數必須在 scope 裡。
JSX 也可以在元素中嵌入子元素。

```js
var Nav, Profile;
// Input (JSX):
var app = <Nav color="blue"><Profile>click</Profile></Nav>;
// Output (JS):
var app = Nav({color:"blue"}, Profile(null, "click"));
```

> 我們根據上面的說明實際撰寫一段可以運作的程式碼如下
```
/** @jsx React.DOM */
var Profile = React.createClass({
  render: function () {
    return (
      <a href='http://andyyou.logdown.com/'>AndyYou</a>
    );
  }
});
var Nav = React.createClass({
  render: function () {
    return (
      <nav>
        <li><Profile /></li>
        <li>Home</li>
        <li>About</li>
      </nav>
    );
  }
});
React.renderComponent(<Nav />, document.getElementById('nav'));
```

使用 [JSX 編譯工具](http://facebook.github.io/react/jsx-compiler.html) 就可以看到 JSX 是如何被轉換的。或者你也可以使用我們提供的[線上工具](http://facebook.github.io/react/html-jsx.html)。
查閱[第一篇教學](http://andyyou.logdown.com/posts/177982-react-getting-started)有關於如何使用編譯工具。

> 注意: 關於上面的虛擬碼是協助你理解關於 JSX 是如何轉換的，你不能夠直接使用虛擬碼的部分。

> 上面範例中加入了一段實作程式碼以方便學習實際演練。

# React 和 JSX
`React` 和 `JSX` 是兩種獨立的技術，但 JSX 是 React 衍伸的一個重要的觀念。 JSX 正確來說被用在兩個地方：
* 構建 React DOM 元件(React.DOM.*)。
* 在使用 `React.createClass()` 設計元件結構。

# React DOM 元件
如果要建立一個 `<div>` 的標記物件，實際上是使用 `React.DOM.div`。

```js
var div =  React.DOM.div;
var app = <div className='appClass'>Hello, React!</div>
```

# React 組合元件
如果你想建立一個組合元件，你應該先透過 `React.createClass({/*....*/})` 建立一個類別，然後才能實例化。
```
var MyComponent = React.createClass({/*...*/});
var app = <MyComponent someProperty={true} />;
```
JSX 會自動根據變數名稱或 `displayName` 推斷元件的名稱，而 [displayName](http://facebook.github.io/react/docs/component-specs.html#displayName) 一般會在 debug 時使用。
查閱[複合式元件](http://facebook.github.io/react/docs/multiple-components.html)學習到更多關於多個元件組成的介紹。

注意：由於 JSX 是一種 Javascript，所以不能夠在標簽裡面直接使用 `class` 和 `for` 當作屬性名稱，替代的方式是 React 會用 `className` 和 `htmlFor` 取代。

# 易於使用的 DOM / 透過 JSX 使程式更加簡潔
如果每一個元素都要自行定義，事情肯定是非常單調乏味(舉例來說: div, span, h1, h2, ...)。JSX 提供一個便利的處理方式就是在 `@jsx` 註解的區塊指定一個變數，JSX 將會用這個指定的領域裡面搜尋 DOM 元件。
```js
/**
 * @jsx React.DOM
 */
// 有了上面的 @jsx React.DOM 就可以使用一般的 DOM 元素（div、span、h1、ul、li、table 等）
// 如果想用其他非 HTML 標簽才需要定義，如下面的Nav
var Nav;
// Input (JSX):
var tree = <Nav><span /></Nav>;
// Output (JS):
var tree = Nav(null, React.DOM.span(null));
```
提醒：JSX 只會單純的把元素轉成函式來呼叫，例如: `React.DOM.div()`，而且並不是表示 DOM 。在註解裡面指定的參數只是為了解決常用的標簽元素。一般來說 JSX 是不俱有 DOM 概念的。

# Javascript 表達式
## 屬性表達式
為了使用 Javascript 來取得屬性的值，JSX 使用 `{...}` 大括號而不是 `"..."` 雙引號。
```js
// Input (JSX):
var person = <Person name={window.isLoggedIn ? window.name : ''} />;
// Output (JS):
var person = Person({name: window.isLoggedIn ? window.name : ''});
```
## 子元素表達式
同樣的，Javascript 也可以拿來放入子元素：
```js
// Input (JSX):
var content = <Container>{window.isLoggedIn ? <Nav /> : <Login />}</Container>;
// Output (JS):
var content = Container(null, window.isLoggedIn ? Nav(null) : Login(null));
```

## 註解
在 JSX 加入註解跟 Javascript 一樣。
```js
var content = <Container>{/* this is a comment */}<Nav /></Container>;
```

> 注意：我們已經知道 JSX 會被轉成 Javascript，上面提到的註解，事實上並不能隨處亂加，舉個例子:

```
var Simple = React.createClass({
  render: function () {
    return (
      <a href='http://andyyou.logdown.com/'>AndyYou</a>
    );
  }
});
var Container = React.createClass({
  render: function () {
    return (
      <div>
        <li><Simple />{ /* 亂加的註解 */}</li>
      </div>
    );
  }
});

React.renderComponent(<Container />, document.getElementById('content'));
```

> 觀察被編譯過的檔案我們可以看到錯誤

```
var Simple = React.createClass({displayName: 'Simple',
  render: function () {
    return (
      React.DOM.a( {href:"http://andyyou.logdown.com/"}, "AndyYou")
    );
  }
});
var Container = React.createClass({displayName: 'Container',
  render: function () {
    return (
      React.DOM.div(null,
        React.DOM.li(null, Simple(null ), /* 亂加的註解 */)

      )
    );
  }
});
React.renderComponent(Container(null ), document.getElementById('content'));
```


# 重點
JSX 類似于其他嵌入 Javascript XML 式的語言或專案。JSX致力于下列的方向：
* JSX 專注于語法解析轉換。
* JSX 不相依於其他外部的函式庫。
* JSX 不會影響既有的 Javascript 語法。

JSX 類似於 HTML 但是並不是完全跟其相同，詳見 [JSX的陷阱](http://facebook.github.io/react/docs/jsx-gotchas.html) 可以知道一些關鍵的不同。

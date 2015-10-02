# 學習手冊
本篇教學會協助你建立一個簡單，但是實用的留言框功能，你可以放置到你的 blog 中。類似于[Disqus](http://disqus.com/)，[LiveFyre](http://web.livefyre.com/)，或者 Facebook comments。
留言框提供下列功能：
* 留言框的界面(view)。
* 一個表單(form)可以送出留言。
* 為你的後端程式提供一個 Hooks ，

> Hooks 簡易說明：Hooks 英文翻譯為鉤子，在程式術語中所表達的是在程式特定位置埋入一段預留的程式碼，用來呼叫其他對應的程式碼。可以大略想成在某個片段先空出一個位置，這個位置可以在事後再放入動作，不放也沒關係。

同時也有下列的特點:
* 優化留言：留言在儲存到伺服器之前就出現在列表中，這會讓使用者有變快的感覺。
* 及時更新：當其他使用者留言時，我們將及時的取出他們的留言並放置到界面中，不用等使用者自己更新頁面。
* Markdown 格式：使用者可以用 `Markdown` 格式來留言，這可使得留言的排版更多元整齊。

# 直接閱覽原始碼

* [Github](https://github.com/petehunt/react-tutorial)

# 起步
在這一篇教學中我們會使用預先建置好放置在 CDN 的 Javascript 檔案。開啟你最愛的文字編輯器例如：Sublime text 然後建立一個 HTML 文件如下：
```html
<!-- template.html -->
<html>
  <head>
    <title>Hello React</title>
    <script src="http://fb.me/react-0.8.0.js"></script>
    <script src="http://fb.me/JSXTransformer-0.8.0.js"></script>
    <script src="http://code.jquery.com/jquery-1.10.0.min.js"></script>
  </head>
  <body>
    <div id="content"></div>
    <script type="text/jsx">
      /**
       * @jsx React.DOM
       */
      // The above declaration must remain intact at the top of the script.
      // 上面的宣告註解仍然得維持在 <script> 標簽中的頂部。
      // Your code here！您的程式碼。
    </script>
  </body>
</html>
```
為了完成教學剩下的部分，我們開始在 `<script>` 標簽內撰寫 Javascript。

# 第一個元件
整個 React 本身就是在模組化和設計可組成的元件，例如這個留言框的範例接下來就會遵循元件的架構。
```
- CommentBox
  - CommentList
    - Comment
  - CommentForm
```
在使用 React 開發時一開始定義好整個架構可以協助您更快速準確的開發。
讓我們來建立 `CommentBox` 留言框這個元件，它只需要一個簡單的 `<div>`：

```js
// tutorial1.js
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        Hello, world! I am a CommentBox.
      </div>
    );
  }
});
React.renderComponent(
  <CommentBox />,
  document.getElementById('content')
);
```
> 小提醒：在 React 中使用 JSX `<div>` 或 `<CommentBox />` 的標簽時 `/` 結尾的關閉標簽一定要加，否則會造成錯誤。

# JSX 語法
首先就是你應該注意到那些在 Javascript 中類似 XML 的語法，這些特殊的 JSX 語法是為了讓我們更方便直覺的維護程式碼的糖衣語法，我們會需要使用預先編譯器負責去轉換這些糖衣語法為原生的 Javascript。

> 糖衣語法：指程式語言中添加的某種語法，這種語法對語言的功能並沒有影響，但是更方便程式設計師使用。通常來說使用語法糖能夠增加程序的可讀性，從而減少程序代碼出錯的機會。

事實上，如果不使用 `JSX` 你依然可以撰寫 React 。不過得改用原生的 React 物件去組織程式碼，而寫出來的程式碼和透過 JSXTransformer 轉換的語法基本上是會一樣的。如下範例：

```js
var CommentBox = React.createClass({
  render: function () {
    reutrn (
      React.DOM.div({
        className: 'commentBox',
        children: 'Hello, world! I am a CommentBox.'
      })
    );
  }
});

React.renderComponent(
  CommentBox({}),
  document.getElementById('content')
)
```
> 小提醒：開發時我們會使用 JSXTransformer 以方便開發，而當要部署 Production 的時候，建議要先行編譯轉換以提升效能。

JSX 不是一定要使用的，不過官方建議 JSX 的語法比純 Javascript 更簡潔易維護，如果你想了解更多可以閱讀 [JSX Syntax](http://facebook.github.io/react/docs/jsx-in-depth.html)。

# 剛剛我們做了什麼？
`React.createClass()` 透過傳入一個 Javascript 物件包含一些方法(method)就可以建立新的 React 元件物件。在所有方法裡面最重要的就是 `render()` ，它會傳回一個 React 元件的樹狀結構，最後被輸出成 HTML 。
在 JSX 裡面的 `<div>` 標簽並不是實際的 DOM 元素。他只是 React div 元件的實例物件(`React.DOM.div`)。
你可以把它當成一種標記或者資料片段，他的功用只是讓 React 知道該怎麼處理這些標記，進而產生對應的 HTML。
這也是 React 本身提供的一套安全機制。它並不會直接產生 HTML 字串，它是透過解析標記最後透過內部一套機制產生 DOM 物件 ，所以預設就可以防止關於 XSS 攻擊。
因此你不需要撰寫一段標準完整的 HTML 語法，只要傳回一個你或其他人寫的元件結構。這就是 React 所謂『可組成』的概念，前端發展可維護性程式碼的核心概念之一。
`React.renderComponent()` 這個方法是用來將`根元件`實例化，並且把 React 產生的標記注入到第二個參數指向的原始 HTML DOM 元素。以上例來說就是第一個參數是先初始化 `<CommentBox />` 把它 `new` 出來後，再把 React 解析產生的 HTML (嚴格說是 DOM 元素)注入到 `document.getElementById('content')` 這個元素中。

# 組成元件
讓我們來繼續建構 `CommentList` 和 `CommentForm`:

```js
var CommentList = React.createClass({
  render: function() {
    return (
      <div className="commentList">
        Hello, world! I am a CommentList.
      </div>
    );
  }
});

var CommentForm = React.createClass({
  render: function() {
    return (
      <div className="commentForm">
        Hello, world! I am a CommentForm.
      </div>
    );
  }
});
```

接著修改 `CommentBox` 元件，使用我們剛剛完成的 `CommentList` 和 `CommentForm`。
```js
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList />
        <CommentForm />
      </div>
    );
  }
});
```
注意到我們在上面的範例混合了 HTML 的 `<h1>` 標簽和 `<CommentList>` 等元件，HTML 標簽是正規的 React 元件，就跟你定義的元件一樣，但這之中有一點點不同。 `JSX` 的編譯器會自動把 HTML 標簽轉成 `React.DOM.tagName` ，這是為了防止污染全域的命名空間。

> 補充：如果你對於不使用 JSX 的寫法比較有興趣下面列出一段大概的用法：

```js
var CommentBox = React.createClass({displayName: 'CommentBox',
  render: function () {
    return (
      React.DOM.div( {className:"commentBox"},
        React.DOM.h1(null, "Comments"),
        CommentList(null ),
        CommentForm(null )
      )
    );
  }
});
```

# 元件屬性(Component Properties)
建立 `Comment` 元件。我們想要顯示留言者的名字和訊息，並且重複使用相同的程式碼給每一則獨立的留言。讓我們加入一些留言到 `CommentList`：

```js
var CommentList = React.createClass({
  render: function() {
    return (
      <div className="commentList">
        <Comment author="Pete Hunt">This is one comment</Comment>
        <Comment author="Jordan Walke">This is *another* comment</Comment>
      </div>
    );
  }
});
```
現在我們使用類似 XML 的語法格式在 `CommentList` 放入一些子元素 `Comment` 和一些資料到屬性。每一個 `<Comment>` 表示一則留言訊息。
在結構上資料是透過父元素傳給子元素的，要完成把資料傳給子元素有一個重要的方式叫做 `props` 就是 Properties 的縮寫，或者說可以透過 `props` 取得父元素的資料。

> 小筆記：邏輯上 CommentList 是整個留言列表所以資料會繫結到這個元件上，再透過迭代的方式去產生底下的 `Comment` 元件。上面的範例我們先使用 hardcode 的方式讓讀者理解概念。

# 使用 props
建立 `Comment` 元件類別，它將會從 `CommentList` 讀取資料然後渲染標記。再複習一次整個 React 的架構就是分別建立各個元件，然後透過組合的方式來實現 `reuse` 的原則，設計上我們通常會讓父元素取得資料，這樣就可以在內部直接迭代渲染輸出 HTML 。
而在取得資料的部分，通常我們會使用 React 提供的 props 來實現 。如下面範例：

```js
var Comment = React.createClass({
  render: function() {
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        {this.props.children}
      </div>
    );
  }
});
```
在 JSX 中透過大括號，你可以在裡面使用 Javascript 表示式（例如取得任何一個屬性或子元素）。你現在可以放入一些文字或 React 元件到這個結構中。然後我們可以透過屬性的名稱去存取資料，使用的方式是利用 `this.props` 或在這個巢狀結構下的任何元素 ex: `this.props.children`。

# 加入 Markdown 功能
Markdown 是一種簡單的文字格式。舉個例子用 `*` 包圍的文字會變成斜體， HTML 中為強調語氣的語意。要增加這個功能我們可以使用第三方的函式庫 `Showdown`。透過這個函式庫可以把 Markdown 格式的文字轉換成 HTML。下面我們直接使用 CDN 上的檔案載入此函式庫。

```html
<!-- template.html -->
<head>
  <title>Hello React</title>
  <script src="http://fb.me/react-0.8.0.js"></script>
  <script src="http://fb.me/JSXTransformer-0.8.0.js"></script>
  <script src="http://cdnjs.cloudflare.com/ajax/libs/showdown/0.3.1/showdown.min.js"></script>
</head>
```

接著就可以在程式碼中使用它

```js
var converter = new Showdown.converter();
var Comment = React.createClass({
  render: function() {
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        {converter.makeHtml(this.props.children.toString())}
      </div>
    );
  }
});
```

這邊我們呼叫了 Showdown 的函式庫，我們需要轉換 `this.props.children` ，讓他從 React 的換行文字先轉成純文字，接著 Showdown 就可以轉換。在這個過程中，我們通常會明確的呼叫 `toString()`。
但是這邊有一個問題，我們輸出的留言看起來會變成 `<p>This is <em>another</em> comment</p>`。我們希望這些標簽可以正確被的輸出成 HTML。
這是 React 預設防止 XSS 攻擊的機制。這邊提供一種方式，但是框架會警告你不要使用它：

```js
var converter = new Showdown.converter();
var Comment = React.createClass({
  render: function() {
    var rawMarkup = converter.makeHtml(this.props.children.toString());
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        <span dangerouslySetInnerHTML={{__html: rawMarkup}} />
      </div>
    );
  }
});
```
這是一個特殊的 API 企圖讓寫入 HTML 變得比較困難一點，但因為 Showdown 需要輸出 HTML 的關係我們必須要開一個後門。

> 提醒：使用這個功能你必須依賴 Showdown 本身的安全性，以及確保被轉換的資料不會有風險。

# 連結資料模型
到目前為止，我們已經在程式碼直接寫入了一些留言，實務上我們的資料通常會從資料庫來，這邊我們用一些 JSON 格式的資料來模擬實際的狀況。

```js
var data = [
  {author: "Pete Hunt", text: "This is one comment"},
  {author: "Jordan Walke", text: "This is *another* comment"}
];
```

在這邊我們會透過模組化的方式去取得資料，實務上舉例就是通常我們會取得一批資料傳給 CommentList 透過程式去一筆一筆輸出。避免使用 hardcode 的方式像上面的例子 `<Comment>`。
修改 `CommentBox` 和 `renderComponent()` ，換成使用 `props` 取得資料後傳給 CommentList。

```js
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.props.data} />
        <CommentForm />
      </div>
    );
  }
});

React.renderComponent(
  <CommentBox data={data} />,
  document.getElementById('content')
);
```
現在資料已經是透過 JSON 格式來取得了並且和 `CommentList` 繫結，接著就讓我們動態的輸出這些留言吧。

> 小筆記：撰寫 React 的時候概念上我們可以把 `createClass` 就理解成是在建立一個類別，`renderComponent()` 就是在實例化物件，此時該傳入的資料(參數)我們就透過標簽屬性(attributes)帶入。

```
var CommentList = React.createClass({
  render: function() {
    var commentNodes = this.props.data.map(function (comment) {
      return <Comment author={comment.author}>{comment.text}</Comment>;
    });
    return (
      <div className="commentList">
        {commentNodes}
      </div>
    );
  }
});
```

透過 map 函式去處理陣列，處理後的結果如下圖。
![](http://i.imgur.com/cFk2lkt.png)
從下面編譯過的程式碼，我們可以知道 React 會自動去迭代輸出陣列。

```js
var CommentList = React.createClass({displayName: 'CommentList',
  render: function() {
    var commentNodes = this.props.data.map(function (comment) {
      return Comment( {author:comment.author}, comment.text);
    });
    return (
      React.DOM.div( {className:"commentList"},
        commentNodes
      )
    );
  }
});
```
範例到此相信有經驗的開發者已經能夠掌握 React 的核心概念了。我們在從範例的角度說明到目前為止我們做了什麼：首先，我們建立了 `CommentBox`，在裡面部署了 `CommentList` 和 `CommentForm` 物件，`CommentList` 透過 `this.props.data` 從父元素 `CommentBox` 那邊取得資料。這個屬性把資料繫結到此物件上。接著在 `CommentList` 類別裡面我們用得到的資料組合出 `Comment` 的陣列，並讓 `Comment` 這個元件去負責每一則留言的呈現。最後我們又用了 `CommentForm` 元件來實踐送出留言的功能。
對於一些比較少OOP經驗的開發者，模組化剛開始可能會覺得有點亂。筆者透過一張圖大略的說明整個基本的流程：
![](http://i.imgur.com/Jluvg4A.png)

# 從伺服器取得資料
讓我們移除寫死的資料改用一些從伺服器端來的動態資料。在這一步我們會透過 url 來取得資料。

```js
React.renderComponent(
  <CommentBox url="comments.json" />,
  document.getElementById('content')
);
```

這個元件將跟之前的不一樣，因為它必須要自己重新載入並輸出。一開始這個元件並沒有任何資料，直到發出的 Request 從伺服器取得資料，這個時候元件就需要重新渲染。在上面範例中我們只是單純地取得某個 JSON 檔案。

# 狀態回應
到目前為止每一個元件都根據自身的 `props` 取得的資料渲染了一次，`props` 本身是靜態不會變動的。它們從父元素取得，而且是父元素擁有的。為了完成互動功能，需要-元件的狀態屬性 `this.state` 。這個屬性本身是 private ，只能透過呼叫 `this.setState()` 去更改。當狀態改變的時候，元件就會重新渲染輸出。
`render()` 是用來處理關於 `this.props` 和 `this.state` 的資料。我們會把資料放在 props 和 state 裡面接著透過 `render()` 去處理該如何呈現。整個框架必須確保所有資料和UI上呈現的是一致的。
當伺服器獲得資料，我們就需要把我們有的資料新增或修改到留言框上。接著讓我們加入一個留言資料的陣列到 CommentBox 的狀態屬性上。

```js
var CommentBox = React.createClass({
  getInitialState: function() {
    return {data: []};
  },
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm />
      </div>
    );
  }
});
```
`getInitialState()` 這個方法再整個元件的生命週期中只會執行一次，目的是用來設定初始化 `this.state` 的資料。按上面的例子就是你傳回了一個 `{data: []}` 物件給 this.state 。之後便可以使用 `this.state.data` 取得這個陣列。

# 更新狀態
當元件第一次被建立的時候，我們想透過 GET 方式從伺服器取得一些 JSON 格式的資料，替 state 更新對應的最新資料。
在實務上這通常是動態從資料庫，API 或其他服務取得，不過在這個例子為了簡化我們使用靜態的 JSON 檔案。
在根目錄建立一個 `comments.json` 如下

```json
[
  {"author": "Pete Hunt", "text": "This is one comment"},
  {"author": "Jordan Walke", "text": "This is *another* comment"}
]
```
我們會搭配使用 jQuery 來協助我們發出非同步請求給伺服器，以取得資料。
注意：因為加入 jQuery 這已經是一個 AJAX 應用程式，你會使用網頁伺服器來執行這個範例，而不是單純在瀏覽器執行檔案。最簡單的方式是在目錄下執行 `python -m SimpleHTTPServer`，或者熟悉 `Grunt` 的開發者可以使用[grunt-init-simple-server](https://github.com/AndyYou/grunt-init-simple-server)。

```js
var CommentBox = React.createClass({
  getInitialState: function() {
    return {data: []};
  },
  componentWillMount: function() {
    $.ajax({
      url: 'comments.json',
      dataType: 'json',
      success: function(data) {
        this.setState({data: data});
      }.bind(this),
      error: function(xhr, status, err) {
        console.error("comments.json", status, err.toString());
      }.bind(this)
    });
  },
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm />
      </div>
    );
  }
});
```

`componentWillMount` 方法會在元件渲染前自動被呼叫執行。在這個例子裡動態更新的關鍵是呼叫 `this.setState()` 。
我們把本來的陣列資料移除，取代用從伺服器取得資料的方式，這裡為了保持簡單是使用 `comments.json` 。
為了展示這種及時反應的效果，這邊加了一段程式碼達到輪詢的功能，意思是程式本身會不斷的重複去查詢 `comments.json` 的資料。 實務上你應該使用 `secket.io` 才不會造成效能很糟糕。

```js
var CommentBox = React.createClass({
  loadCommentsFromServer: function() {
    $.ajax({
      url: this.props.url,
      dataType: 'json',
      success: function(data) {
        this.setState({data: data});
      }.bind(this)
    });
  },
  getInitialState: function() {
    return {data: []};
  },
  componentWillMount: function() {
    this.loadCommentsFromServer();
    setInterval(this.loadCommentsFromServer, this.props.pollInterval);
  },
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm />
      </div>
    );
  }
});

React.renderComponent(
  <CommentBox url="comments.json" pollInterval={2000} />,
  document.getElementById('content')
);
```

我們把 AJAX 取得資料的片段獨立成一個方法，然後第一次載入的時候會呼叫一次，接著設定每兩秒執行一次。

# 新增留言
是時候來建置我們的留言表單了，這個留言表單元件應該要詢問使用者他的名字和留言訊息，接著提交給伺服器儲存留言。

```js
var CommentForm = React.createClass({
  render: function() {
    return (
      <form className="commentForm">
        <input type="text" placeholder="Your name" />
        <input type="text" placeholder="Say something..." />
        <input type="submit" value="Post" />
      </form>
    );
  }
});
```
表單運作的流程是：當使用者提交表單時，我們應該清除裡面的資料，接著發出一個 Reauest 給伺服器，最後更新留言列表。

```js
var CommentForm = React.createClass({
  handleSubmit: function() {
    var author = this.refs.author.getDOMNode().value.trim();
    var text = this.refs.text.getDOMNode().value.trim();
    if (!text || !author) {
      return false;
    }
    // TODO: send request to the server
    this.refs.author.getDOMNode().value = '';
    this.refs.text.getDOMNode().value = '';
    return false;
  },
  render: function() {
    return (
      <form className="commentForm" onSubmit={this.handleSubmit}>
        <input type="text" placeholder="Your name" ref="author" />
        <input
          type="text"
          placeholder="Say something..."
          ref="text"
        />
        <input type="submit" value="Post" />
      </form>
    );
  }
});
```

* 事件：
React 在元件上繫結事件是用 `camelCase` 命名規則，當表單驗證過並提交時我們使用了一個 `onSubmit` 事件來處理。這裡我們永遠回傳一個 false 來阻止瀏覽器預設的提交行為。（如果你比較喜歡透過 event 參數使用 `e.preventDefault()`，你也可以用它來取代 `return false`）

* Refs：
使用 `ref` 屬性只設定子元素的名稱，就可以透過 `this.refs` 參考到該元件。我們可以呼叫 `getDOMNode()` 來取得原生的 DOM 元素。

* 透過 `props` 使用 `callback`：
當使用者送出一則留言，我們將需要更新留言列表加入新的訊息， 在 `CommentBox` 實作這些邏輯是比較合理的，因為 `CommentBox` 負責掌控整個元件的狀態 `this.state`。
所以在這個範例裡，我們需要從子元素把資料傳給父元素 `CommentBox` ，讓 `CommentBox` 去更新狀態，為了完成這個目的，我們在 `props` 放入一個 `callback` 函式，如此一來子元素便能透過呼叫這個函式把資料帶給父元素。
看到這邊可能會有些混亂，沒關係讓我們先直接看 `CommentBox` 的程式碼:

```js
var CommentBox = React.createClass({
  loadCommentsFromServer: function() {
    $.ajax({
      url: this.props.url,
      dataType: 'json',
      success: function(data) {
        this.setState({data: data});
      }.bind(this)
    });
  },
  handleCommentSubmit: function(comment) {
    // TODO: submit to the server and refresh the list
  },
  getInitialState: function() {
    return {data: []};
  },
  componentWillMount: function() {
    this.loadCommentsFromServer();
    setInterval(this.loadCommentsFromServer, this.props.pollInterval);
  },
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm
          onCommentSubmit={this.handleCommentSubmit}
        />
      </div>
    );
  }
});
```
然後在 `CommentForm` 呼叫

```js
var CommentForm = React.createClass({
  handleSubmit: function() {
    var author = this.refs.author.getDOMNode().value.trim();
    var text = this.refs.text.getDOMNode().value.trim();
    this.props.onCommentSubmit({author: author, text: text});
    this.refs.author.getDOMNode().value = '';
    this.refs.text.getDOMNode().value = '';
    return false;
  },
  render: function() {
    return (
      <form className="commentForm" onSubmit={this.handleSubmit}>
        <input type="text" placeholder="Your name" ref="author" />
        <input
          type="text"
          placeholder="Say something..."
          ref="text"
        />
        <input type="submit" value="Post" />
      </form>
    );
  }
});
```

> 總結上面敘述就是我們應該把處理寫入留言(提交到伺服器)的邏輯和程式碼寫在 `CommentBox` 元件裡面，就是 `handleCommentSubmit` 這個方法，接著透過 `<CommentForm onCommentSubmit={this.handleCommentSubmit} />` 的方式把方法傳給子元件 `CommentForm` ，`CommentForm` 有需要送出留言的時候就可以透過 `this.props.onCommentSubmit()` 去呼叫。資料統一都由父元素管理是比較合理的做法。到目前為止我們概略的了解關於狀態和資料還有處理的方法應該放在父元素，你可以在父元素裡面在放置其他元件，如果子元素需要動用父元素的功能或資料就透過屬性(attributes)帶入參數的方式傳進去。

現在讓我們來完成 `callback` 函式該執行的任務

```js
var CommentBox = React.createClass({
  loadCommentsFromServer: function() {
    $.ajax({
      url: this.props.url,
      dataType: 'json',
      success: function(data) {
        this.setState({data: data});
      }.bind(this)
    });
  },
  handleCommentSubmit: function(comment) {
    $.ajax({
      url: this.props.url,
      dataType: 'json',
      type: 'POST',
      data: comment,
      success: function(data) {
        this.setState({data: data});
      }.bind(this)
    });
  },
  getInitialState: function() {
    return {data: []};
  },
  componentWillMount: function() {
    this.loadCommentsFromServer();
    setInterval(this.loadCommentsFromServer, this.props.pollInterval);
  },
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm
          onCommentSubmit={this.handleCommentSubmit}
        />
      </div>
    );
  }
});
```
# 優化UI更新
這個範例到這邊已經全部完成了(實際呼叫 AJAX 後端操作並沒有實作在範例裡)，但是感覺有點慢，因為我們必須要等待發出的 Request 完成處理留言才會出現。我們可以優化這個部分讓使用者感覺更快。

```js
var CommentBox = React.createClass({
  loadCommentsFromServer: function() {
    $.ajax({
      url: this.props.url,
      dataType: 'json',
      success: function(data) {
        this.setState({data: data});
      }.bind(this)
    });
  },
  handleCommentSubmit: function(comment) {
    var comments = this.state.data;
    var newComments = comments.concat([comment]);
    this.setState({data: newComments});
    $.ajax({
      url: this.props.url,
      dataType: 'json',
      type: 'POST',
      data: comment,
      success: function(data) {
        this.setState({data: data});
      }.bind(this)
    });
  },
  getInitialState: function() {
    return {data: []};
  },
  componentWillMount: function() {
    this.loadCommentsFromServer();
    setInterval(this.loadCommentsFromServer, this.props.pollInterval);
  },
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm
          onCommentSubmit={this.handleCommentSubmit}
        />
      </div>
    );
  }
});
```
上面這段程式碼的重點在加入

```js
var comments = this.state.data;
var newComments = comments.concat([comment]);
this.setState({data: newComments});
```
透過這段程式使用者的留言在送出時本地端就先更新了，再發出 Request 給伺服器。後續元件本身會自己在去跟伺服器讀取最新的資訊。

# 恭喜
你已經完成建置這個留言框的功能，也初步對如何使用 React 有了認識，學習更多關於[為什麼使用 React](http://facebook.github.io/react/docs/why-react.html) 或查閱 [API 參考](http://facebook.github.io/react/docs/top-level-api.html)。

最後附上完成的範例程式碼：
```html
<!-- template.html -->
<html>
  <head>
    <title>Hello React</title>
    <script src="http://fb.me/react-0.8.0.js"></script>
    <script src="http://fb.me/JSXTransformer-0.8.0.js"></script>
    <script src="http://code.jquery.com/jquery-1.10.0.min.js"></script>
    <script src="http://cdnjs.cloudflare.com/ajax/libs/showdown/0.3.1/showdown.min.js"></script>
  </head>
  <body>
    <div id="content"></div>
    <script type="text/jsx">
      /**
       * @jsx React.DOM
       */
      var converter = new Showdown.converter();
      var data = [
        {author: "Pete Hunt", text: "This is one comment"},
        {author: "Jordan Walke", text: "This is *another* comment"}
      ];
      var CommentBox = React.createClass({
        loadCommentsFromServer: function () {
          $.ajax({
            url: this.props.url,
            dataType: 'json',
            success: function (data) {
              this.setState({data: data});
            }.bind(this),
            error: function (xhr, status, err) {
              console.error('comments.json', status, err.toString());
            }.bind(this)
          });
        },
        handleCommentSubmit: function (comment) {
          var comments = this.state.data;
          var newComments = comments.concat([comment]);
          this.setState({data: newComments});
          $.ajax({
            url: this.props.url,
            dataType: 'json',
            type: 'POST',
            data: comment,
            success: function (data) {
              this.setState({data:data});
            }.bind(this)
          });
        },
        getInitialState: function () {
          return {data: []};
        },
        componentWillMount: function () {
          this.loadCommentsFromServer();
          setInterval(this.loadCommentsFromServer, this.props.pollInterval)
        },
        render: function () {
          return (
            <div className='commentBox'>
              <h1>Comments</h1>
              <CommentList data={this.state.data}/>
              <CommentForm onCommentSubmit={this.handleCommentSubmit}/>
            </div>
          );
        }
      });
      /*
      var CommentBox = React.createClass({
        render: function () {
          return (
            React.DOM.div({
              className: 'commentBox',
              children: 'Hello, world! I am a CommentBox.!!!'
            })
          );
        }
      });
      */
      var CommentList = React.createClass({
        render: function () {
          var commentNodes = this.props.data.map(function (comment) {
            return <Comment author={comment.author}>{comment.text}</Comment>
          });
          return (
            <div className='commentList'>
              {commentNodes}
            </div>
          );
        }
      });

      var CommentForm = React.createClass({
        handleSubmit: function () {
          var author = this.refs.author.getDOMNode().value.trim();
          var text = this.refs.text.getDOMNode().value.trim();
          if (!text || !author) {
            return false;
          }
          this.props.onCommentSubmit({author: author, text: text});
          this.refs.author.getDOMNode().value= '';
          this.refs.text.getDOMNode().value = '';
          return false;
        },
        render: function () {
          return (
            <form className='commentForm' onSubmit={this.handleSubmit}>
              <input type='text' placeholder='Your name' ref='author' />
              <input type='text' placeholder='Say something' ref='text' />
              <input type='submit' value="Post" />
            </form>
          );
        }
      });

      var Comment = React.createClass({
        render: function () {
          var rawMarkup = converter.makeHtml(this.props.children.toString());
          return (
            <div className='comment'>
              <h3 className='commentAuthor'>
                {this.props.author}
              </h3>
              <span dangerouslySetInnerHTML={{__html: rawMarkup}} />
            </div>
          );
        }
      });


      React.renderComponent(
        <CommentBox url='comments.json' pollInterval={2000} />,
        document.getElementById('content')
      );
    </script>
  </body>
</html>

```

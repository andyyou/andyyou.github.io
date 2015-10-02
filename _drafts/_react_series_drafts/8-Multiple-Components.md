# 複合式元件
到目前為止我們看過了如何建立一個單一的元件去呈現資料以及回應使用者的操作。接下來讓我們來看看 React 的另一個重要的功能：可組成。

# 動機：關注點分離
模組化建立可重複使用的界面元件，透過使用 function 或類別，讓我們可以在開發時得到一些益處。具體來說可以針對應用程式的功能分離不同的關注點，不過還是請你在建立新元件的時候儘量保持單純。針對應用程式自行設計元件庫，你的 UI 也比較容易和你的應用整合在一起。

> 在使用 jQuery 和其他第三方套件時候，我們常常會因為作者設定的 HTML 結構(樣板)打亂了我們既有的編排習慣，在 React 中我們通常只要在需要的地方放入一個 `<div>` ，接著實際產出的 HTML 和功能會透過 Javascript 直接注入。我們只要知道怎麼用元件就好，而不需要去組織樣板。

# 範例
讓我們建立一個單純的 Avatar(頭像)元件，它使用 Facebook Graph API 來取得個人資料和大頭照然後顯示。
``` js
/** @jsx React.DOM */

var Avatar = React.createClass({
  render: function() {
    return (
      <div>
        <ProfilePic username={this.props.username} />
        <ProfileLink username={this.props.username} />
      </div>
    );
  }
});

var ProfilePic = React.createClass({
  render: function() {
    return (
      <img src={'http://graph.facebook.com/' + this.props.username + '/picture'} />
    );
  }
});

var ProfileLink = React.createClass({
  render: function() {
    return (
      <a href={'http://www.facebook.com/' + this.props.username}>
        {this.props.username}
      </a>
    );
  }
});

React.renderComponent(
  <Avatar username="pwh" />,
  document.getElementById('example')
);
```

在上面的範例中，`Avatar` 元件實例裡面俱有 `ProfilePic` 和 `ProfileLink` 兩個元件。在 React 裡面主元件就是最上層的元件應該要提供 `props` 所需的資料。更精准一點來說，如果有一個元件Ｘ被寫在元件Ｙ的 `render()` 裡面，這表示 Y 是擁有者-主元件也就是那個負責掌管狀態的元件。如同之前討論的一個元件不能更動自己的 `props`，而是透過上層元件去設定。通過屬性當接點我們可以保證 UI 的資料永遠是來自同一個地方以確保資料一致性。
有個重要的觀念是釐清這些被擁有者的關係和主從關係。被擁有者關係是 React 所俱有的，而元素地主從關係(父元素及子元素關係)就是單純的 DOM 結構。拿上面的範例說明：`Avatar` 擁有 `div`，`ProfilePic`，`ProfileLink` 物件。`div` 只是父元素，但他不是`ProfilePic` 和 `ProfileLink` 的擁有者(主元件)。

> 從程式的概念上來理解，一個 Avatar 是透過 `React.createClass()` 先建立類別，然後 `new` (`React.renderComponent(<Avatar />, [domTag])`)產生的實例物件來使用，每一個物件本身都有自己的 `this.props` 屬性。且資料通常是透過 `<Avatar username={data} />` 傳入的。元件的資料狀態通常避免從外部影響，帶入參數之後就讓元件自己內部去處理。看看編譯過的程式碼會比較好理解 `React.renderComponent(Avatar({username:'pwh'}), document.getElementById('example'));`，所以擁有者元件是 `Avatar` 而不是 `render()` 裡面的 `<div>`。當然所謂的控管資料和狀態就是 `Avatar` 。而 `<div>` 只不過是用來輸出 DOM 結構的父元素。

# 子元件
當你建立了一個 React 物件，你可以包含其他的 React 元件或 Javascript 表示式。
``` xml
<Parent><Child /></Parent>
```

讓我們根據上面的說明再提出一個範例

``` js
/** @jsx React.DOM */
var Avatar = React.createClass({
  render: function () {
    return (
      <div>
        <ProfilePic username={this.props.username} />
        <ProfileLink username={this.props.username} />
        {this.props.children /* 載入子元素 */}
      </div>
      /* 複習一下一個元件只能有一個根節點，你不能在這邊再加入一個 <div> */
    );
  }
});
var ProfilePic = React.createClass({
  render: function () {
    return (
      <img src={'http://graph.facebook.com/' + this.props.username + '/picture'} />
    );
  }
});
var ProfileLink = React.createClass({
  render: function () {
    return (
      <a href={'http://www.facebook.com/' + this.props.username}>
        {this.props.username}
      </a>
    );
  }
});
React.renderComponent(<Avatar username='andyyu0920'><ProfileLink username='phw' /></Avatar>, document.getElementById('example'));
```

父元件可以透過 `this.props.children` 讀取子元件。

# 子元件調和(Reconciliation)
調和的意思是 React 更新渲染 DOM 的處理過程。一般來說子元件會根據他們的順序重新被調整輸出。舉下面的例子來說

``` xml
// Render Pass 1
<Card>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</Card>
// Render Pass 2
<Card>
  <p>Paragraph 2</p>
</Card>
```
我們會直覺的認為 `<p>Paragraph 1</p>` 被移除，但實際上 React 會重新調和 DOM ，他會把第一個元素的內容換掉，接著刪除最後一個元素。

``` js
/** @jsx React.DOM */
var Card = React.createClass({
  render: function () {
    return (
      <div>
        {this.props.children}
      </div>
    )
  }
});
React.renderComponent(
  <Card>
    <p>Paragraph 1</p>
    <p>Paragraph 2</p>
  </Card>,
  document.getElementById('example')
);
React.renderComponent(
  <Card>
    <p>Paragraph 2</p>
  </Card>,
  document.getElementById('example')
);
```

為了更理解上面說的我們實作了另一個範例

``` js
/** @jsx React.DOM */
var Card = React.createClass({
  getInitialState: function () {
    return {children: [<input type='text' />, <p>Paragraph 1</p>, <p>{(new Date().toTimeString())}</p>]};
  },
  render: function () {
    return (
      <div>
        {this.state.children}
      </div>
    );
  }
});
var card = React.renderComponent(
      <Card />,
      document.getElementById('example')
    );
setInterval(function() {
  card.setState({children: [<input type='text' />, <p style={{display: 'none'}}>Paragraph 1</p>, <p>{(new Date().toTimeString())}</p>,<p>Paragraph 2</p>]});
}, 500);
```

直覺上我們會覺得 `<input>` 會被重新輸出，但是當我們在輸入框裡面留下資料的時候會發現他並沒有變成空白。總結來說 React 並不是單純直接把元件輸出，而是在內部經過比對處理後只更新異動的部分。

# 內嵌子元件狀態
對大多數元件來說，上面說的這種機制通常沒有什麼問題，然而對控管 `this.state` 的元件來說這可能會有問題。
一般情況下你可以透過隱藏元素來取代刪除他們。也就是說通常元件的結構定義完成之後我們通常不會去破壞任何一個節點。

``` xml
// Render Pass 1
<Card>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</Card>
// Render Pass 2
<Card>
  <p style={{display: 'none'}}>Paragraph 1</p>
  <p>Paragraph 2</p>
</Card>
```

# 動態的內嵌子元件
當情況變得更複雜，內嵌的子元件被重新排列，例如顯示搜尋結果，或者要加入一些新的元件。在這些情況下每個元件唯一的識別子或狀態必須維持在 `render()` 傳遞。
``` js
render: function() {
    var results = this.props.results;
    return (
      <ol>
        {this.results.map(function(result) {
          return <li key={result.id}>{result.text}</li>;
        })}
      </ol>
    );
  }
```
當 React 調和(重新調整)這些帶有 `key` 的子元件時，這可以確保任何有 `key` 的元件都將被重新載入(而不是被破壞)或破壞(而不是重複使用)。
我們來寫段範例驗證

``` js
/** @jsx React.DOM */
var data = [{id: 1, text:'A'}];
var List = React.createClass({
  render: function () {
    var results = this.props.results;
    return (
      <ol>
        {results.map(function (result) {
          return <input type='text' key={result.id}/>;
        })}
      </ol>
    );
  }
});
React.renderComponent(<List results={data} />, document.getElementById('example'));

setInterval(function() {
  data[0].id += 1;
  React.renderComponent(<List results={data} />, document.getElementById('example'));
}, 5000);
```
在 `<input>` 輸入值之後 5 秒後因為 key 變換了所以你的輸入的值就被清空了。

# 資料流
在 React ，資料是從主元件透過 `props` 傳遞就如同之前說過的。實際上這是一個單向的資料繫結。主元件負責把資料繫結到子元件的 `props` ，主元件可以基於 `props` 或 `state` 進行計算，由於這個流程會發生遞迴所以資料會自動映射至使用的地方。

# 關於效能
你也許會思考這樣的模式當 React 需要修改大量資料和節點的時候效能會不佳，好消息是 Javascript 本身是非常快速的，而且 `render()` 往往不會太複雜，因此大部份的應用程式速度都非常快。此外問題幾乎都在 DOM 的更動並不是 Javascript 而且 React 將會使用 `batching` 和 `change detection` 優化這些。
然而有些時候你真的想要調整這些效能的問題，此時你可以覆寫 `shouldComponentUpdate()` 方法，透過回傳 `false` 。React 將會略過這段處理。詳細參閱 [the React reference docs](http://facebook.github.io/react/docs/component-specs.html) 。

注意：當 `shouldComponentUpdate()` 傳回 `false` ，此時資料被改變了，React 就不能維持 UI 同步了。只有在有明顯效能問題的時候才使用它，且請確保你知道該如何使用。

# 呈現資料
在開發網頁的過程中，大部份的工作就是讓你的 UI 界面呈現資料。React 讓界面可以方便地去呈現資料並且當資料有異動的時候自動更新。

# 入門
讓我們看看這個簡單的範例。建立 `hello-react.html` 檔案並加入如下的程式碼：
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello React</title>
    <script src="http://fb.me/react-0.11.1.js"></script>
    <script src="http://fb.me/JSXTransformer-0.11.1.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/jsx">

      // ** 您的程式碼應該在此 **

    </script>
  </body>
</html>
```
針對檔案剩下的部分，我們只會專注在關於 `Javascript` 程式碼的部分，舉例來說你應該把下面的 Javascript 程式碼插入在像上面的 HTML 指示的位置上。
```js
/** @jsx React.DOM */

var HelloWorld = React.createClass({
  render: function() {
    return (
      <p>
        Hello, <input type="text" placeholder="Your name here" />!
        It is {this.props.date.toTimeString()}
      </p>
    );
  }
});

setInterval(function() {
  React.renderComponent(
    <HelloWorld date={new Date()} />,
    document.getElementById('example')
  );
}, 500);
```

# 及時更新
在瀏覽器打開 `hello-react.html` ，注意到 React 只會自動更新時間的部分。有 Javascript 經驗的開發者會注意到，這邊我們是透過 `setInterval()` 每 500 毫秒 `renderComponent()` 一次。但是如果你在 `input` 中輸入一些資料會發現它會維持不變，並不會被清空。React 會正確地為你處理這些。
從這邊我們可以得知 React 除非必要，平常不會直接操作 DOM 元素。它採用一種快速的方式-在內部模擬 DOM 來進行比較和計算如何有效率的操作 DOM。
一般來說我們會使用 `props` 把資料輸入元件，它是 `Properties`-屬性的縮寫 。`props` 是透過 JSX 標簽裡的屬性(attributes)來傳遞的。
你應該想到這些 `props` 在元件內部是靜態不變的，我們不會在元件內部變更 `this.props` 的值。

> 一般來說我們會把資料放在 `<Tag attribute={data here}>` 屬性，從外部把資料傳進去，接著在內部透過 `this.props` 取得資料或函式。

# 元件本身就像函數
React 元件非常單純，你可以把他們想成單純的函數，在取得 `props` 和 `state` (後續介紹) 後渲染產生 HTML，這讓整個元件的架構比較好理解。

注意：React 的限制，React 元件只有一個根節點，如果你想要傳回多個節點的結構就必須把它們全部包含在根節點裡面。

# JSX 語法
我們非常相信元件比樣板搭配呈現邏輯更可以正確的達到關注點分離的目的。我們認為標簽和程式碼應該緊密的聯結在一起。
此外呈現邏輯通常非常複雜，使用樣板語言去表示會變得很笨重。
我們發現，針對這個問題最佳的解決方式就是直接從 Javascript 把行為和標簽一起產生。如此才能完全發揮實際程式語言俱有的表述能力去建立 UI。為了讓這一切簡化，我們加入了 JSX 的功能，透過類似 HTML 的語法去產生標記物件。
JSX 讓你可以在 Javascript function 裡面使用類似 HTML 的語法，舉例來說在 React 裡面要做一個超連結如果不用 JSX 的話你得這樣寫 `React.DOM.a({href='http://facebook.github.io/react'}, 'Hello React!')` ，如果是透過 JSX 則變成這樣 `<a href="http://facebook.github.io/react/">Hello React!</a>`。我們覺得這樣做會讓建置設計 React 應用程式或元件變得簡單。但因為採用了 JSX 在部署專案或開發流程上會有一些需要額外加入處理的事情，基於每個開發者都有自己的一套工作流程和偏好，所以 JSX 不是必要的，它是附加可選的。
JSX 非常類似 HTML，但事實上它們並不是完全一樣。查閱[JSX gotchas ](http://facebook.github.io/react/docs/jsx-gotchas.html)可以知道哪些關鍵不同的地方。

最後，最簡單開始學習 JSX 的方式就是使用 `JSXTransformer` 。但是我們強烈建議不要在 Production 上面使用，你可以使用[react-tools](http://npmjs.org/package/react-tools)預先編譯。

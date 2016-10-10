---
layout: post
title: 'React 大解密'
date: 2014-09-20 00:43:00
categories: Program
tags: reactjs
---
## React 揭秘
關於這篇文章將會試著解釋關於 React 核心的概念。

<!--more-->

## 鳥瞰架構
在傳統的網頁應用程式中，我們如果要增加互動性時勢必廣泛的操作 DOM 元素，一般來說現在最普遍的技術是使用 `jQuery`:
![](http://i.imgur.com/qcNTzCE.png)
上圖我們故意讓 DOM 示意為紅色這是因為操作更新 DOM 是需要付出昂貴的代價，也意味著這很吃效能。
很多時候我們會使用 Model 來記錄關於 APP 狀態，不過通常我們最後目標是必須要將狀態呈現給使用者，所以我們必須自己實作這些細節。
這已經是我們很稀鬆平常的開發模式。

而 React 的主要目標就是提供一種不同且更有效率的方式去執行關於操作更新 DOM 這個部分，最終這個方式會取代我們直接操作 DOM 的方法。
React 使用的方式是透過建立一套虛擬 DOM 的機制，React 幫你處理關於操作 DOM 方面的事情。
![](http://i.imgur.com/Lrzy1e3.png)

為什麼多引進一層架構會讓效能增加? 如果在其架構之上多引入一層可以提升速度，這不是暗示瀏覽器並沒有實作最佳的 DOM 操作方式。
這也意味著虛擬 DOM 有著跟實際 DOM 不同的語義和行為。值得關注的是當我們改變虛擬 DOM 時並不能保證立即得到效果。
也因為這個機制導致 React 在實際接觸 DOM 之前必須要等待事件回圈結束。在同一時間它會去計算最小差異並盡可能的用最少的步驟去更新 DOM。

如此一來應用程式便能獨立執行批次更新，套用計算後的差異到實際 DOM 上，任何應用程式如果這麼做那麼都能夠像 React 一樣有效率。
但實際上自己編寫程式碼去做這些任務是很繁瑣且容易出錯，React 的精華之處就是幫你處理掉這些問題。

## 元件
就上面所提到的虛擬 DOM 的機制有著跟直接操作實際 DOM 不一樣的語義和行為，所以也會有明顯不同的 API。
所謂的元素即在 DOM 結構中的一個節點(node)，不過在虛擬 DOM 機制底下一個節點完全是不一樣的東西，我們稱這個節點為`元件`。

使用元件對 React 來說是一件非常重要的事情，因為元件的設計概念是要拿來做計算的，就是計算和實際 DOM 的差異。
比起計算整個結構的差異，React 透過虛擬 DOM 將使得實際執行的時間複雜度大幅下降。

為了理解為什麼? 我們必須深入探討元件的設計，就從 `Hello World` 範例:

~~~js
/** @jsx React.DOM */
var HelloMessage = React.createClass({
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }
});

React.renderComponent(<HelloMessage name="Andy" />, mountNode);
~~~

上面這段程式碼出現了一些可怕的東西，且在這個階段無法完全說明清楚。即使是這麼小的一段範例都包含著一個很強大的概念，所以在這邊我們將會花些時間慢慢一點一點說明。

這個範例建立了一個 React 元件的類別(class): `HelloMessage`，然後透過 `renderComponent()` 在虛擬的 DOM 的機制中建立一個元件(`<HelloMessage />`, 本質上它就是 HelloMessage 類別實例化的物件，同時也是一個虛擬的 DOM)
最後把這個物件裝到真實的 DOM 元素(mountNode)。

首先是需要注意的事情是 React 的虛擬 DOM 通常來自您在應用程式中客制的元件(在這個例子是 `<HelloMessage>`)。這是一個意義重大的新嘗試，從內建的 DOM 分離出來。
DOM 通常不帶有任何程式邏輯，就只是一個被動的資料結構，且讓我們能夠附加處理事件。換句話說 React 的虛擬 DOM 是透過特定程式中的元件所創造的，且能夠加入程式中的特定 API 及內部邏輯。
這樣的方式比起直接修改操作 DOM ，例如: 使用 jQuery 的方式，這種建置 View 的方法是一種全新的抽象化方式與框架。

值得一提的是: 如果您一直持續關注 HTML 你也許知道關於 HTML 也許很快的也能[自訂 DOM](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/)。
這將會帶給 DOM 類似的功能: 定義特定程式使用的 DOM 元素，不過 React 並不需要等到官方和瀏覽器完全實作這件事，因為虛擬 DOM 並不是真的 DOM。這讓 React 搶先在自訂元素與 [Shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)
這些功能實作普及之前您就能先用了。

回到我們的範例，我們已經建立了一個叫做 `<HelloMessage>` 的元件並且掛載 `mountNode` 裡面。
讓我們用圖片來說明初始化幾個部分的情形，首先，我們將虛擬 DOM 與實際 DOM 的關係視覺化，假設 `mountNode` 是網頁中的 `<body>` 標簽:

> 關於掛載(mount)一詞就理解為『對應』，把 a 掛載到 b 上 = 可以把 b  視為 a 。

![](http://i.imgur.com/F5KnIW7.png)

箭頭表示虛擬元件已經被掛載到原生 DOM 元素中。這段過程非常短，不過也讓我們來看看關於應用程式的視圖部分的邏輯:

![](http://i.imgur.com/MElJPrk.png)

這張圖片指的是整個網頁的內容是由我們客制的 `<HelloMessage/>` 來呈現，那麼關於 `<HelloMessage/>` 看起來到底長怎樣?

關於元件輸出渲染的部分是透過 `render()` 去定義欲呈現的元素。React 並沒有確切的說明關於何時或多頻繁的會去執行 `render()` 。
只有告訴我們當它注意到有效合法的改變時本身會去執行足夠次數的 `render()`，無論你回傳什麼樣的 DOM 結構。

在我們這個案例，`render()` 回傳了一個 `<div>` 裡面包含了一些內容。React 會執行這個 `render()` 取得 `<div>` 然後更新實際的 DOM ，使兩者一致。

![](http://i.imgur.com/w0w7JOe.png)

不僅僅是更新 DOM，還會幫你記住已經更新的東西。這也是我們待會會提到的關於 React 如何快速的判斷其中差異的方法。
關於 render() 如何回傳 DOM 節點，這邊我先簡略帶過。它是透過 JSX 去定義結構，這是一個非原生的 Javascript，注意雖然它看起來像是 XML。
最後 JSX 會被編譯回 Javascript，看看 JSX 的編譯結果有助我們理解這個架構:

~~~js
/** @jsx React.DOM */
var HelloMessage = React.createClass({displayName: 'HelloMessage',
  render: function() {
    return React.DOM.div(null, "Hello ", this.props.name);
  }
});

React.renderComponent(HelloMessage( {name:"John"} ), mountNode);
~~~

看到了吧！我們真的不是回傳一個 DOM 元素，而是一個等價于 DOM 元素的 React Shadow DOM。所以我們得知 React 回傳的並不是真的 DOM。
您可以理解為標記物件。

## 狀態與變化
到目前為止，我們忽略了故事中很重要的一段，關於元件可以被改變這件事。如果一個元件不允許被調整修改，那 React 跟 `static rendering framework`(靜態渲染框架) 也沒啥兩樣，功能就類似于 [Mustache](https://github.com/janl/mustache.js) 或者 [HandlebarsJS](http://handlebarsjs.com/)
這些樣板引擎，不過 React 的重點就是有效率的更新，要能夠更新元件勢必要允許我們修改一些狀態之類的東西。

React 使用元件的 `state` 屬性來表示其狀態的資料模型。
關於這點在官方文件的第二個範例就有舉例說明:

~~~js
/** @jsx React.DOM */
var Timer = React.createClass({
  getInitialState: function() {
    return {secondsElapsed: 0};
  },
  tick: function() {
    this.setState({secondsElapsed: this.state.secondsElapsed + 1});
  },
  componentDidMount: function() {
    this.interval = setInterval(this.tick, 1000);
  },
  componentWillUnmount: function() {
    clearInterval(this.interval);
  },
  render: function() {
    return (
      <div>Seconds Elapsed: {this.state.secondsElapsed}</div>
    );
  }
});

React.renderComponent(<Timer />, mountNode);
~~~

React 會在適當的時間點執行回呼函式 `getinitialState()`, `componentDidMount()` 以及 `componentWillUnmount()` ，根據到目前為止的解釋您應該可以清楚地理解這些函式名稱的其含義。
所以我們推測元件和狀態背地裡的行為:
1. `render()` 是 `state` 和 `props` 的一個 function，也就是當它們發現異動會執行 render。
2. `state` 只能透過 `setState()` 去改變。
3. `props` 不應該持續變動，只有當其父元素用新的屬性重新輸出時才改變。

(在這之前我們沒有明確地提到 props ，不過他們就是屬性 attributes。當元件要 render 時，它們就來自那些 JSX Tag 中的屬性)

稍早，我們曾經提到 React 會自己執行`足夠次數`的 render ，意思是除非有需要不然 React 不會執行 render。
那需要什麼？當你發動 setState() 或者父元件重新賦予新的 `props`，React 就會重新輸出。

現在我們將所有的事情放在一起，用一張圖來說明當程式更新了虛擬 DOM 的資料流(例如: 回應 AJAX 呼叫):

![](http://i.imgur.com/w5RMkO2.png)

> 1. 發動了 AJAX
> 2. React 內部需要呼叫 setState 用以改變內部狀態
> 3. 因內部狀態改變進而觸發 render
> 4. render 執行需要依照生命週期呼叫像 `componentWillMount` 這類的方法
> 5. 最後根據計算的最小差異更新 DOM

## 從 DOM 取得資料
截至目前為止，我們只有討論到關於收到狀態改變，到如何傳遞到實際 DOM，但實務上我們會需要從 DOM 取得資料，例如從 input 取得使用者輸入的資料。
為了觀察如何運作，我們取用了官方第三個範例:

~~~js
/** @jsx React.DOM */
var TodoList = React.createClass({
  render: function() {
    var createItem = function(itemText) {
      return <li>{itemText}</li>;
    };
    return <ul>{this.props.items.map(createItem)}</ul>;
  }
});
var TodoApp = React.createClass({
  getInitialState: function() {
    return {items: [], text: ''};
  },
  onChange: function(e) {
    this.setState({text: e.target.value});
  },
  handleSubmit: function(e) {
    e.preventDefault();
    var nextItems = this.state.items.concat([this.state.text]);
    var nextText = '';
    this.setState({items: nextItems, text: nextText});
  },
  render: function() {
    return (
      <div>
        >h3<TODO</h3>
        <TodoList items={this.state.items} />
        <form onSubmit={this.handleSubmit}>
          <input onChange={this.onChange} value={this.state.text} />
          <button>{'Add #' + (this.state.items.length + 1)}</button>
        </form>
      </div>
    );
  }
});
React.renderComponent(<TodoApp />, mountNode);
~~~

簡單的說，我們把我們要的行為綁定到 DOM 的事件(在這個範例就是 onChange)，接著你在這個事件中呼叫 setState 讓 React 幫您更新介面(實際的 DOM)。
如果您的程式有資料模型，您的事件大概就是透過 setState 更新那個資料模型，React 發現狀態異動就會去更新。
另外如果您曾經用過其他提供雙向資料繫結的框架，看起來可能會懷疑 React 本身在技術上退化了?

儘管這個範例看起來 React 並不是真的把事件加到 `<input>` 的 "onChange" ，取而代之的是加到文檔層級。讓事件透過汽泡傳遞的機制然後分派他們到正確的虛擬 DOM 元素。
這麼做的好處是包含提升速度(在 DOM 綁定太多處理事件會讓網站變慢)，跨瀏覽器實現一樣的行為(處理事件的屬性和其派送的行為並沒有統一的標準，意思是在不同瀏覽器可能有些許的差異)。

所以最後我們可以總結一張完整的圖片來說明關於資料流和事件處理機制:

![](http://i.imgur.com/qq0uWsY.png)

## 結論
* React 是一個處理 View 的函式庫: React 並不強迫您要在 Model 做些什麼設定或改變。一個 React 元件只是一個 View-Level 的概念，而元件的狀態就只是 UI 方面的狀態。您可以繫結任何類型的資料模型或者函式庫到 React(雖然某些資料模型的處理方式會更有效率，例如 Om)
* React 的元件抽象化在更新 DOM 的方面尤其優秀: 元件抽象化是一個原則，使得我們可以編寫組織良好的架構，同時又提供高效率的更新機制。
* React 元件從 DOM 的角度執行更新不太方便: 比起函式庫自動傳遞同步資料模型，撰寫事件處理給 React 帶來一種很低階的感覺。
* React 是[抽象漏洞](http://zh.wikipedia.org/wiki/%E6%8A%BD%E8%B1%A1%E6%BC%8F%E6%B4%9E%E5%AE%9A%E5%BE%8B): 意味著 React 有本質上的缺陷，但有提供避免問題發生的方向。大部份的時間你的程式只會和虛擬 DOM 打交道，但有時候你需要直接對 DOM 做些操作。此時您可以查閱手冊的[這部分](http://facebook.github.io/react/docs/working-with-the-browser.html)

---
layout: post
title: 'React 動畫'
date: 2014-09-13 23:02:00
categories: Reactjs
---

## 動畫
React 提供了一個 `ReactTransitionGroup` 的組件元件來作為一個底層的動畫 API，以及另一個 `ReactCSSTransitionGroup` 來方便實作基本的 CSS 動畫。

> 高度抽象化的 API(High-level API) 與 底層的 API(Low-level API) 是什麼？
  `High-level API` 允許開發者操作[WSDL](http://w3school.com.cn/wsdl/wsdl_documents.asp)，一般來說一個 WDSL 編譯器全部的 SOAP 界面來產生物件，而開發者需要做的就只是直接呼叫物件中的 `method`，簡單的來說你不需要直接操作 SOAP 的結構。而 `Low-level API` 則允許開發者直接去動 SOAP 結構，而不是呼叫物件中的方法
  [更多關於 High-level 與 Low-level 定義](http://en.wikipedia.org/wiki/High-_and_low-level)


## 高度抽象化的 API: ReactCSSTransitionGroup
`ReactCSSTransitionGroup` 建構在 `ReactTransitionGroup` 之上，當 React 元件掛載或卸載到 DOM 物件時，您可以透過 `ReactCSSTransitionGroup` 可以簡單加入 CSS 過場特效與動畫。
這個靈感來自于 `ng-animate` 。

### 入門
`ReactCSSTransitionGroup` 是一個連接到 `ReactTransitions` 的介面。這個單純的元素需要把你想要做動畫效果的元件全部包起來。下面的範例是我們要讓列表的項目淡入淡出:

{% highlight js %}
/** @jsx React.DOM */

var ReactCSSTransitionGroup = React.addons.CSSTransitionGroup;

var TodoList = React.createClass({
  getInitialState: function() {
    return {items: ['hello', 'world', 'click', 'me']};
  },
  handleAdd: function() {
    var newItems =
      this.state.items.concat([prompt('Enter some text')]);
    this.setState({items: newItems});
  },
  handleRemove: function(i) {
    var newItems = this.state.items;
    newItems.splice(i, 1);
    this.setState({items: newItems});
  },
  render: function() {
    var items = this.state.items.map(function(item, i) {
      return (
        <div key={item} onClick={this.handleRemove.bind(this, i)}>
          {item}
        </div>
      );
    }.bind(this));
    return (
      <div>
        <button onClick={this.handleAdd}>Add Item</button>
        <ReactCSSTransitionGroup transitionName="example">
          {items}
        </ReactCSSTransitionGroup>
      </div>
    );
  }
});
{% endhighlight %}

在這個元件中，當你把一個 `item` 加入 `ReactCSSTransitionGroup` ，它將會取得 `example-enter` 的 CSS 屬性，而當你再次點擊這個項目時會取得 `example-enter-active` 樣式。
而這個慣例是根據在 `transitionName` 的名稱所組成的。
現在，您可以使用這些 `class` 樣式去觸發 CSS 的動畫或者過場的效果。舉例來說讓我們加入下面的 CSS

{% highlight css %}
.example-enter {
  opacity: 0.01;
  transition: opacity .5s ease-in;
}

.example-enter.example-enter-active {
  opacity: 1;
}
{% endhighlight %}

你可能注意到了，當您要移除某個項目的時候， `ReactCSSTransitionGroup` 卻把它保留在 DOM 裏面。
同時如果你用的是未經壓縮的 `react-with-addons` 版本，就會看到 `console` 警告你，React 期望取得一個 CSS 效果。
`ReactCSSTransitionGroup` 會保留您的 DOM 元素直到動畫執行完畢

![](http://i.imgur.com/o1D2VRp.png)

試著加入下面這段 CSS

{% highlight css %}
example-leave {
  opacity: 1;
  transition: opacity .5s ease-in;
}

.example-leave.example-leave-active {
  opacity: 0.01;
}
{% endhighlight %}

### 動畫群組被需要被掛載才能執行
為了套用這些過場特效到子元素中，`ReactCSSTransitionGroup` 必須要先被掛載到 DOM 中。下面是一個錯誤的示範，因為
`ReactCSSTransitionGroup` 是隨著新的項目一起被掛載而無法生效，如果您把 `ReactCSSTransitionGroup` 包覆在子元素的外層則會出現奇怪的效果。
比對上面一開始的範例就會明白了。
> 這小節要說明的觀念是: ReactCSSTransitionGroup 運作的方式是先安裝到 DOM 在賦予子元件這些效果。

### 有無 `items` 的動畫行為
雖然上面的範例我們在 `ReactCSSTransitionGroup` 中渲染了列表，不過 `ReactCSSTransitionGroup` 的是允許沒有子元件的。
意味著您可以移除到完全沒有項目，即渲染的 `items` 是空的。
這個部分我們理解到 `ReactCSSTransitionGroup` 是針對單一元素在掛載或者卸載的時候套用動畫。同時也可以用在新元素取代舊元素的時候
其行為為兩個元素同時播放動畫，舊的元素會先保留 DOM 等其動畫播放完畢。
舉例來說，我們可以實作一個簡單圖片輪播的範例:

{% highlight js %}
/** @jsx React.DOM */

var ReactCSSTransitionGroup = React.addons.CSSTransitionGroup;

var ImageCarousel = React.createClass({
  propTypes: {
    imageSrc: React.PropTypes.string.isRequired
  },
  render: function() {
    return (
      <div>
        <ReactCSSTransitionGroup transitionName="carousel">
          <img src={this.props.imageSrc} key={this.props.imageSrc} />
        </ReactCSSTransitionGroup>
      </div>
    );
  }
});
{% endhighlight %}

> 注意: 您必須替每一個在 `ReactCSSTransitionGroup` 中的子元素設定 `key` 的屬性，即使只有一個項目。這是 React 偵測判斷子元素狀態的方式(entered, left, stayed)。

> 這邊提供一段展示影片，即使我們只透過 this.state 去更新圖片的 `src` ，ReactCSSTransitionGroup 還是會再產生一個節點。
[![](http://img.youtube.com/vi/r0KjKXnSbhQ/0.jpg)](http://www.youtube.com/watch?v=r0KjKXnSbhQ)

官方提供的範例只是概念上的說明，上面展示影片的範例程式碼如下:

{% highlight js %}
/** @jsx React.DOM */
var ReactCSSTransitionGroup = React.addons.CSSTransitionGroup;

var ImageCarousel = React.createClass({
  propTypes: {
    imageSrc: React.PropTypes.string.isRequired
  },
  getInitialState: function () {
    return {imageSrc: this.props.imageSrc}
  },
  handleChangeImage: function () {
    console.log('changed');
    this.setState({imageSrc: 'dist/images/1.jpg'});
  },
  render: function () {
    return (
      <div>
        <ReactCSSTransitionGroup transitionName="carousel">
          <img src={this.state.imageSrc} key={this.state.imageSrc} onClick={this.handleChangeImage}/>
        </ReactCSSTransitionGroup>
      </div>
    );
  }
});

React.renderComponent(
  <ImageCarousel imageSrc='dist/images/0.jpg' />,
  document.getElementById('example')
);
{% endhighlight %}

### 關閉動畫
如果您想，您也可以取消 `enter` 或 `leave` 的動畫。有時候你的需求是只要物件出現時的動畫，但是最一開始的範例有提到如果沒有實作移除的動畫， ReactCSSTransitionGroup 會卡著 DOM 物件。
您可以透過在 `ReactCSSTransitionGroup` 加入 `transitionEnter={false}` 或 `transitionLeave={false}` props 已關閉動畫效果。

{% highlight js %}
<ReactCSSTransitionGroup transitionName='example' transitionLeave={false}>
  {items}
</ReactCSSTransitionGroup>
{% endhighlight %}


> 注意: 使用 `ReactCSSTransitionGroup` 時，裡面的元件並沒辦法在過場或動畫結束後收到任何通知，也不能在動畫時期執行更多複雜的邏輯。
> 如果您需要控制更多細節，您可以使用底層的 API `ReactTransitionGroup API` ，它一樣是透過提供 hooks 的方式讓您可以自訂效果。

## 底層 API: ReactTransitionGroup
`ReactTransitionGroup` 是動畫的基礎。當子元素被加入或移除(如上面範例)，會在一個特有的生命週期去呼叫它的 hooks。

* componentWillEnter(callback)
  當元件被加入一個已存在的 `TransitionGroup` 這個事件被呼叫，而觸發時間跟 `componentDidMount()` 時一樣。
  它會中斷其他的動畫直到 `callback` 被執行。另外是他不會在初始化的時候被呼叫。

* componentDidEnter()
  當上面的 callback 執行完畢之後接著會執行這個事件。

* componentWillLeave(callback)
  當子元件已經被從 `ReactTransitionGroup` 移除的時候會執行這個事件，雖然子元件已經被移除了，但是 `ReactTransitionGroup` 還是會保留 DOM 直到 `callback` 被呼叫。

* componentDidLeave()
  一樣的當 `willLeave` `callback` 被呼叫(時間點跟 `componentWillUnmount` 一樣)。

### 渲染一個不同元件
預設 `ReactTransitionGroup` 會輸出一個 `span`

![](http://i.imgur.com/IsKFmU8.png)

不過您是可以使用 `component` 屬性修改這個預設的行為

{% highlight js %}
<ReactTransitionGroup component={React.DOM.ul}>
  ...
</ReactTransitionGroup>
{% endhighlight %}

所有的 DOM 元件都在 `React.DOM.` 底下，然後 `component` 並不需要一定是 DOM 元件。
這可以是任何您想要的 React 元件甚至是您開發的。

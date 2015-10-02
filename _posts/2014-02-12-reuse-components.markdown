---
layout: post
title: 'React 重複利用元件'
date: 2014-02-12 23:58:00
categories: Reactjs
---
# 重複利用元件
這裡的設計界面，指的是打破現有的設計元素(button, form, fields 等)組合出定義良好可重複使用的元件。這樣一來下次你需要建置一樣的界面的時候就可以少寫一些程式碼，同時也節省許多開發時間。

# Prop 驗證
當你的應用程式不斷增加，透過設定 `propTypes` 有助你確保你的元件被正確的使用。。`React.PropTypes` 會產生一系列的驗證使得你可以確保收到的資料是正確的。
當 `prop` 提供一個無效的資料時就會發出一個錯誤的例外。讓我們看看下面的使用範例：

{% highlight js %}
React.createClass({
  propTypes: {
    // 您可以替 prop 指定 Javascript 預設的型別
    // 這些都是可以選擇的
    optionalArray: React.PropTypes.array,
    optionalBool: React.PropTypes.bool,
    optionalFunc: React.PropTypes.func,
    optionalNumber: React.PropTypes.number,
    optionalObject: React.PropTypes.object,
    optionalString: React.PropTypes.string,

    // 可以明確的限制 prop 為列舉型別。
    optionalEnum: React.PropTypes.oneOf(['News','Photos']),

    // 限制某種 Class
    someClass: React.PropTypes.instanceOf(SomeClass),

    // 加上特上面說的任何一種型別加上必須的限制。
    requiredFunc: React.PropTypes.func.isRequired

    // 加上自訂的驗證
    customProp: function(props, propName, componentName) {
      if (!/matchme/.test(props[propName])) {
        throw new Error('Validation failed!')
      }
    }
  },
  /* ... */
});

{% endhighlight %}

下面提供一個範例，你可以試著把 `isShow` 改成非 boolean 的任何值。

{% highlight js %}
/** @jsx React.DOM */
var Test = React.createClass({
  propTypes: {
    isShow: React.PropTypes.bool.isRequired
  },
  render: function () {
    return (
      <p>{this.props.isShow ? "true" : "false"}</p>
    );
  }
});
React.renderComponent(<Test isShow={true} />, document.getElementById('example'))
{% endhighlight %}

出現錯誤：
![](http://i.imgur.com/CZMeCgx.png)

我們可以再多嘗試一些範例來更明確理解:

{% highlight js %}
/** @jsx React.DOM */
 var Car = function (wheel, brand) {
   this.wheel = wheel;
   this.brand = brand;
 };
 Car.prototype.run = function () {
   console.log('go!');

 var Bike = function (wheel) {
   this.wheel = wheel;
 }
 var car = new Car(4, 'Toyota');
 var bike = new Bike(2);
 console.log(car
 var Test = React.createClass({
   propTypes: {
     vihicle: React.PropTypes.instanceOf(Car)
   },
   render: function () {
     return (
       <p>I drive {this.props.vihicle.brand} car and its has {this.props.vihicle.wheel} </p>
     );
   }
 });
 React.renderComponent(<Test vihicle={car} />, document.getElementById('car'));
 // React.renderComponent(<Test vihicle={bike} />, document.getElementById('bike'));
{% endhighlight %}

# 預設 `Prop`
React 可以讓你定義 `props` 的預設值

{% highlight js %}
var ComponentWithDefaultProps = React.createClass({
  getDefaultProps: function() {
    return {
      value: 'default value'
    };
  }
  /* ... */
});
{% endhighlight %}

執行範例：

{% highlight js %}
/** @jsx React.DOM */
var ComponentWithDefault = React.createClass({
  getDefaultProps: function () {
    return {value: 'B'}
  },
  render: function () {
    return (
      <div>{this.props.value}</div>
    );
  }
});
React.renderComponent(<ComponentWithDefault value='A' />, document.getElementById('example'));
{% endhighlight %}

`getDefaultProps` 將會把預設值暫存起來，以確保 `this.props.value` 有值。以上面的實際範例來說如果你不帶值則值會是 B ，如果有值則預設值會被取代。這使得你可以放心使用 `props` ，而無需反覆撰寫脆弱的代碼來處理元件。

# 傳遞 Props 的捷徑
常見的 React 元件是用基本 HTML 組合的延伸。通常你會想要傳遞屬性給 HTML 元素，`React` 提供了 `transferPropsTo()` 方法可以把屬性帶入讓你少打一些字。

{% highlight js %}
/** @jsx React.DOM */

var CheckLink = React.createClass({
  render: function() {
    // transferPropsTo() will take any props passed to CheckLink
    // and copy them to <a>
    return this.transferPropsTo(<a>{'√ '}{this.props.children}</a>);
  }
});

React.renderComponent(
  <CheckLink href="javascript:alert('Hello, world!');">
    Click here!
  </CheckLink>,
  document.getElementById('example')
);
{% endhighlight %}

上面這段說明，其實我們應該先釐清一般來說我們會把 `React.renderComponent(<Component attribute='value'/>)`，當作呼叫函式或實例化物件，所以這邊的工作是把參數帶進去。接著因為 React 在 `render()` 的時候只能有一個根元素去包含其他元素。所以當你用了 `transferPropsTo()` 實例化產生物件那邊的屬性(attributes)會被整合到跟元素，接著你可以用 `this.props.children` 拿到在帶進來的子元素。
如果上面這段說明你還不是很明白，我們在比較一下沒有 `transferPropsTo()` 的寫法：

{% highlight js %}
/** @jsx React.DOM */

var CheckLink = React.createClass({
  render: function() {
    // transferPropsTo() will take any props passed to CheckLink
    // and copy them to <a>
    return (<a href={this.props.href}>{'√ '}{this.props.children}</a>);
  }
});

React.renderComponent(
  <CheckLink href="javascript:alert('Hello, world!');">
    Click here!
  </CheckLink>,
  document.getElementById('example')
);
{% endhighlight %}

好吧！上面這個例子你並沒有少打幾個字，但如果當屬性很多的時候就真的是了XD

# Mixin
在 React 中元件是是幫助你重複使用程式碼最好的方式，但是有些時候不同的元件可能會同樣的功能。有時候被稱作橫切關注點。React 提個 `mixin` 解決這個問題。
一個常見的情況是元件透過 `setInterval()` 更新，不過問題是當你不需要更新要取消 `setInterval()` 的時候。當你知道關於元件的[生命週期](http://facebook.github.io/react/docs/working-with-the-browser.html#component-lifecycle)，你就可以透過 `mixin` 讓有需要這個功能的元件共用。

{% highlight js %}
/** @jsx React.DOM */

var SetIntervalMixin = {
  componentWillMount: function() {
    this.intervals = [];
  },
  setInterval: function() {
    this.intervals.push(setInterval.apply(null, arguments));
  },
  componentWillUnmount: function() {
    this.intervals.map(clearInterval);
  }
};

var TickTock = React.createClass({
  mixins: [SetIntervalMixin], // Use the mixin
  getInitialState: function() {
    return {seconds: 0};
  },
  componentDidMount: function() {
    this.setInterval(this.tick, 1000); // Call a method on the mixin
  },
  tick: function() {
    this.setState({seconds: this.state.seconds + 1});
  },
  render: function() {
    return (
      <p>
        React has been running for {this.state.seconds} seconds.
      </p>
    );
  }
});

React.renderComponent(
  <TickTock />,
  document.getElementById('example')
);
{% endhighlight %}

注意當元件混入很多 mixin event 的時候，如果是生命週期的函式則保證被呼叫，但如果不是則會導致元件被破壞。

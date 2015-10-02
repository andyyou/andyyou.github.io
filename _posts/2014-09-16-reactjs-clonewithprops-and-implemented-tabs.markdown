---
layout: post
title: 'React cloneWithProps 與實作 Tabs'
date: 2014-09-16 16:16:00
categories: Reactjs
---

# 實作一個 Tabs 元件

## 複合式(組合)元件
在 React 中任何東西都是元件，就像樂高一樣，你可以用小片的積木組成大塊的，再組合出您想到的東西。
同樣的道理您也可以用許多的小元件(小功能模組)來組合出您的應用程式。所謂的複合式元件或稱作組合元件，
他其實就是由多個元件去組成一個多功能的大元件。

在這篇文章我們要來建立一個 `tabs` 標簽切換功能的元件，為了達成這個功能我們需要 4 個不同的元件:
`<Tabs />`, `<TabList />`, `<Tab />` 和 `<TabPanel />` 分別用來呈現整個 `tabs` ， 列出
 標簽列，標簽列的按鈕，以及顯示的內容。結構如下:


{% highlight html %}
 <Tabs>
  <TabList>
    <Tab>Iron man</Tab>
    <Tab>Superman</Tab>
    <Tab>Lucy</Tab>
  </TabList>
  <TabPanel>
    鋼鐵人介紹
  </TabPanel>

  <TabPanel>
    超人介紹
  </TabPanel>

  <TabPanel>
    鹿茸介紹
  </TabPanel>
 </Tabs>
{% endhighlight %}

首先是 `<Tabs/>` 的行為，它被用來當做一個容器，其角色有點像是一個 `controller` ，因為它必須要掌管所有 DOM 的事件(點擊 Tab 切換至該內容，被選取到的 index)
同時也需要管理 `state` 看看哪個 `<Tab/>` 目前正被選取到，所以我們會稱 `<Tabs />` 為擁有者元件(owner component)。

我們遭遇到的第一個挑戰是: 元件之間該如何溝通。每一個元件都有一個 `state` 。每當 `state` 發生變動，React 就會更新並重新渲染元件以使其跟 `state` 一致。
當我們選了某個索引後，`<Tabs/>` 元件就要去更新 `<Tab/>` 和 `<TabPanel/>` 的 `state`。

## [典範轉移](http://cguim99.blogspot.tw/2013/03/paradigm-shift_24.html)
首先我們為每一個元件建立一個 API，透過建立一個方法來變更 `state`。在 React 中一個元件可以透過 `this.props.children` 去存取子元件。
所以一開始我們理論上只要使用 `handleSelected` 去設定適當該顯示的子元件如下:

{% highlight js %}
var tabs = this.props.children[0].props.children,
    panels = this.props.children.slice(1),
    index = this.state.selectedIndex;
tabs[index].handleSelected(true);
panels[index].handleSelected(true);
{% endhighlight %}

這樣的做法在 v0.10.0 以前的版本是可以運作的，概略的實作如下:

<div data-height="268" data-theme-id="8540" data-slug-hash="oitzv" data-default-tab="js" data-user="AndyYou" class='codepen'><pre><code>/**
 * @jsx React.DOM
 */


var Tab = React.createClass({
  getInitialState: function () {
    return {selected: false}  
  },
  handleSelected: function (status) {
    this.setState({selected: status});
  },
  render: function () {
    var cx = React.addons.classSet;
    var classes = cx({
      &#x27;react-tab&#x27;: true,
      &#x27;active&#x27;: this.state.selected
    });
    return (
      &lt;li className={classes}&gt;
        &lt;a href=&#x27;#&#x27; data-index={this.props.index}&gt;{this.props.children}&lt;/a&gt;
      &lt;/li&gt;
    );
  }
});

var TabList = React.createClass({
  render: function () {
    return (
      &lt;ul className=&#x27;react-tab-list&#x27;&gt;
        {this.props.children}
      &lt;/ul&gt;
    );
  }
});

var Tabs = React.createClass({
  getInitialState: function () {
    return {selectedIndex: 0}  
  },
  componentDidMount: function () {
    var tabs = this.props.children[0].props.children,
        panels = this.props.children.slice(1),
        index = this.state.selectedIndex;
    for (i in tabs) {
      if (i == index) {
        tabs[i].handleSelected(true);
        panels[i].handleSelected(true);
      } else {
        tabs[i].handleSelected(false);
        panels[i].handleSelected(false);
      }
    }
  },
  handleClick: function (e) {
    var index = parseInt(e.target.getAttribute(&#x27;data-index&#x27;));
    var tabs = this.props.children[0].props.children,
        panels = this.props.children.slice(1);
    for (i in tabs) {
      if (i == index) {
        tabs[i].handleSelected(true);
        panels[i].handleSelected(true);
      } else {
        tabs[i].handleSelected(false);
        panels[i].handleSelected(false);
      }
    }

  },
  render: function () {
    return (
      &lt;div className=&#x27;react-tabs&#x27; onClick={this.handleClick}&gt;
        {this.props.children}
      &lt;/div&gt;
    );
  }
});

var TabPanel = React.createClass({
  getInitialState: function () {
    return {selected: false}
  },
  handleSelected: function (status) {
    console.log(this.props.name + &#x27; selected: &#x27; + status);
    this.setState({selected: status});
  },
  render: function () {
    var cx = React.addons.classSet;
    var classes = cx({
      &#x27;react-tab-panel&#x27;: true,
      &#x27;active&#x27;: this.state.selected
    });
    return (
      &lt;div className={classes}&gt;
        {this.props.children}
      &lt;/div&gt;
    )
  }
});

var App = React.createClass({
  render: function () {
    return (
      &lt;div className=&#x27;container&#x27;&gt;
        &lt;Tabs&gt;
          &lt;TabList&gt;
            &lt;Tab name=&#x27;ironman&#x27; index={0}&gt;Iron man&lt;/Tab&gt;
            &lt;Tab name=&#x27;superman&#x27; index={1}&gt;Superman&lt;/Tab&gt;
            &lt;Tab name=&#x27;lucy&#x27; index={2}&gt;Lucy&lt;/Tab&gt;
          &lt;/TabList&gt;

          &lt;TabPanel name=&#x27;panel-ironman&#x27;&gt;
          鋼鐵人
          &lt;/TabPanel&gt;

          &lt;TabPanel name=&#x27;panel-superman&#x27;&gt;
          超人再起
          &lt;/TabPanel&gt;

          &lt;TabPanel name=&#x27;panel-lucy&#x27;&gt;
          露西
          &lt;/TabPanel&gt;
        &lt;/Tabs&gt;
      &lt;/div&gt;
    );
  }
});

React.renderComponent(
  &lt;App /&gt;,
  document.getElementById(&#x27;example&#x27;)
);</code></pre>
<p>See the Pen <a href='http://codepen.io/AndyYou/pen/oitzv/'>oitzv</a> by AndyYou (<a href='http://codepen.io/AndyYou'>@AndyYou</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>


不過到了 React v0.10.0 版本的時候這樣做會出現警告:


{% highlight sh %}
Invalid access to component property "setSelected"
{% endhighlight %}

到了 v0.11.0 的時候更慘您已經無法直接存取子元件的方法，因為是新版的 React `this.props.children` 回傳的物件只是描述物件(discriptors)。不再是對應元件的參考物件，且官方建議您不應該直接存取子元件的實際物件。


## Component Refs
React 提供一種機制給你取得實際元件的物件，就是使用 `refs`


{% highlight js %}
var App = React.createClass({
  handleClick: function () {
    alert(this.refs.myInput.getDOMNode().value);
  },

  render: function () {
    return (
      <div>
          <input ref="myInput"/>
          <button
              onClick={this.handleClick}>
              Submit
          </button>
      </div>
    );
  }
});
{% endhighlight %}

透過 `refs` 屬性您就可以取得該子元件的參考

### 動態的子元件
典型的 `refs` 使用方式是父元件已經知道子元件的情況，所以可以直接在 `tag` 中指定 `ref` 如上面的範例。
不過這次我們希望我們的 Tabs 元件可以動態的放入 `<Tab/>` 和 `<TabPanel/>`
例如:

{% highlight js %}
var App = React.createClass({
  render: function () {
    return (
      <div className='container'>
        <Tabs>
          <TabList>
            <Tab name='ironman' >Iron man</Tab>
            <Tab name='superman' >Superman</Tab>
            <Tab name='lucy' >Lucy</Tab>
          </TabList>

          <TabPanel name='panel-ironman'>
          鋼鐵人
          </TabPanel>

          <TabPanel name='panel-superman'>
          超人再起
          </TabPanel>

          <TabPanel name='panel-lucy'>
          露西
          </TabPanel>
        </Tabs>
      </div>
    );
  }
});
{% endhighlight %}

而 Tabs 只是動態地把開發者加入的任意結構輸出

{% highlight js %}
var Tabs = React.createClass({
  render: function () {
    return (
      <div className='react-tabs'>
        {this.props.children}
      </div>
    );
  }
});
{% endhighlight %}

因此我們需要一些動態指定 `refs` 的方法，而這個方法就是透過 `cloneWithProps`

{% highlight js %}
var App = React.createClass({
  render: function () {
    var index = 0,
        children = React.Children.map(this.props.children, function (child) {
        return React.addons.cloneWithProps(child, {
            ref: 'child-' + (index++)
        });
    });

    return (
        <div>
            {children}
        </div>
    );
  }
});
{% endhighlight %}

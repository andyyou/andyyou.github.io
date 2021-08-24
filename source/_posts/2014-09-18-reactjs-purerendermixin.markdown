---
layout: post
title: 'React PureRenderMixin'
date: 2014-09-18 11:53:00
categories: Program
tags: [reactjs]
---

## PureRenderMixin
如果您的 React 元件的 `render` 函式非常單純(相對來說，就只是單純想把 props 和 state 輸出)，您就可以使用 `PureRenderMixin` 來提升效能:

<!--more-->

~~~js
var PureRenderMixin = require('react').addons.PureRenderMixin;
React.createClass({
  mixins: [PureRenderMixin],

  render: function() {
    return <div className={this.props.className}>foo</div>;
  }
});
~~~

內部的 mixin 幫您實作了 `shouldComponentUpdate` ，他會幫您比對 `props` 與 `state` ，只要一樣就回傳 `false`。
協助您快速提升一些效能:

~~~js
/**
 * @jsx React.DOM
 */
var PureRenderMixin = React.addons.PureRenderMixin;
var iPhone = React.createClass({
  mixins: [PureRenderMixin],
  getDefaultProps: function () {
    return {className: 'red'}
  },
  handleClick: function () {
    this.setProps({className: 'blue'});
    console.log('excute handle click');
  },
  render: function() {
    console.log('I am rendering now...');
    return <div className={this.props.className} onClick={this.handleClick}>foo</div>;
  }
});

React.renderComponent(
  <iPhone />,
  document.getElementById('example')
)
~~~

> 注意:
> 關於比對方面，React 只為您執行淺層的比對。意思是如果您的資料結構非常複雜可能 shouldComponentUpdate() 可能永遠判斷兩個物件為 true，因為他只做淺層比對，有可能是只比較參考。
  所以建議您使用單純屬性和狀態或使用強制更新 `forceUpdate()` 當你知道這個資料結構太複雜的時候。
  此外，`shouldComponentUpdate` 會略過子元件的更新，所以請確保所有的東西是真的很單純！

---
layout: post
title: 'React classSet 方法'
date: 2014-09-14 13:18:00
categories: Program
tags: reactjs
---
## 操作樣式名稱(Class)
`classSet()` 是一個協助您快速操作 DOM 裏 `class` 字串的工具。
這裡有一些常見的情況，當沒有使用 `classSet()` 的時候:

<!--more-->

~~~js
/**
 * @jsx React.DOM
 */
var ClassSet = React.createClass({
  render: function () {
    var classString = 'message';
    if (this.props.isImportant) {
      classString += ' message-important';
    }
    if (this.props.isRead) {
      classString += ' message-read';
    }
    return (
      <div className={classString}>Great, I'll be there</div>
    )
  }
});

React.renderComponent(
  <ClassSet isRead='1' isImportant='1' />,
  document.getElementById('example')
)
~~~

你可能因為一些 CSS Class 設定的問題而開始組合字串，但很顯然的這是個很無聊瑣碎的工作。
而且這種做法程式碼很難閱讀且易出錯。這個時候 `classSet()` 可以幫你解決這個問題。

~~~js
/**
 * @jsx React.DOM
 */
var ClassSet = React.createClass({
  render: function () {
    var cx = React.addons.classSet;
    var classes = cx({
      'message': true,
      'message-important': this.props.isImportant,
      'message-read': this.props.isRead
    })
    return (
      <div className={classes}>Great, I'll be there</div>
    )
  }
});

React.renderComponent(
  <ClassSet isRead='1' isImportant='1' />,
  document.getElementById('example')
)
~~~

> 其他補充事項:
> 只要看到程式中使用了 `React.addons.*`， React 記得使用 `react-with-addons` 的版本，另外有些屬性名稱因為跟程式關鍵字衝突會有變更，這裡的 class 就變成 className，另外一個常遇到的是 `for` 要改成 `htmlFor`。


~~~js
return (
  <div className={classes}>
    <label htmlFor='boy'>Boy</label>
    <input type='radio' id='boy' name='sex'/>
    <label htmlFor='girl'>Girl</label>
    <input type='radio' id='girl' name='sex'/>
  </div>
)
~~~

當我們使用 `classSet()` 時，我們傳入一個物件，其中的 `key` 屬性名稱就是 CSS class 樣式。當它設成 `Truthy values` 的時候就表示要套用，這個樣式名稱會自動加入串接的字串中。

> Truthy values 指的是只要 Javascript 判斷為真即可，並不是一定要 `true`，舉例來說 `1`, `string`, `Object` 都是 Truthy values 。

不要再使用串字串的方式了！

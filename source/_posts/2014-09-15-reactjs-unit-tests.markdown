---
layout: post
title: 'React 單元測試'
date: 2014-09-15 18:12:00
categories: Program
tags: reactjs
---

## 單元測試
`React.addons.TestUtils` 讓您可以在您的測試框架中更簡單的測試您的元件(官方使用 [`Jest`](http://facebook.github.io/jest/))。
簡單的來說它是一系列輔助的方法以協助您測試 React 元件。

<!--more-->

### Simulate (模擬方法)

~~~js
Simulate.{eventName}(DOMElement element, object eventData)
~~~

模擬一個 DOM 的事件調派，其中可以包含一個 `eventData`。
這可能是在 `ReactTestUtils` 最有用的一個工具。

使用方式的範例:

~~~js
var node = this.refs.input.getDOMNode();
React.addons.TestUtils.Simulate.click(node);
React.addons.TestUtils.Simulate.change(node);
React.addons.TestUtils.Simulate.keyDown(node, {key: "Enter"});
~~~

> 下面開始使用統一的格式來簡易說明每個方法的格式

~~~~~
回傳的物件型別 方法名稱(參數型別 參數名稱)
~~~~~

### renderIntoDocument

~~~js
ReactComponent renderIntoDocument(ReactComponent instance)
~~~

輸出一個元件到 `document` 獨立的 DOM 節點(div)去。如下面舉例的一小段測試範例:

~~~js
describe("Label Test",function(){
    beforeEach(function() {
        ReactTestUtils = React.addons.ReactTestUtils;
    });

    it("Check Text Assignment", function () {
        var label = <Label>Some Text We Need for Test</Label>;
        ReactTestUtils.renderIntoDocument(label);
        expect(label.refs.p).toBeDefined();
        expect(label.refs.p.props.children).toBe("Some Text We Need for Test")
    });
});
~~~

觀察原始碼:

~~~js
renderIntoDocument: function(instance) {
  var div = document.createElement('div');
  // 我們的測試並不需要附加到網頁中的某個 DOM 元素時使用:
  return React.renderComponent(instance, div);
}
~~~

### mockComponent

~~~js
object mockComponent(function componentClass, string? tagName)
~~~

模擬元件，顧名思義透過傳入一個模擬的元件模組到這個方法中以快速製作一個"假"元件，取代本來用 `render` 的方式。
讓我們來查看原始碼好理解官方簡短的說明:

~~~js
mockComponent: function(module, mockTagName) {
  var ConvenienceConstructor = React.createClass({
    render: function() {
      var mockTagName = mockTagName || module.mockTagName || "div";
      return ReactDOM[mockTagName](null, this.props.children);
    }
  });
  copyProperties(module, ConvenienceConstructor);
  module.mockImplementation(ConvenienceConstructor);
  return this;
}
~~~


### isDescriptorOfType

~~~js
boolean isDescriptorOfType(ReactDescriptor descriptor, function componentClass)
~~~

當 `discriptor` 物件符合該元件類別時回傳 `true`。
> *關於 descriptors*
> 我們提過 React 是透過一套虛擬 DOM 的機制，並不是直接操作 DOM 元素，在 v0.10 版之前 React 回傳給你的物件
> 都是這個 discriptor (描速物件) 它和 React 內部操作的元件是指向同一個參考。
  [更多關於 discriptor 機制](https://gist.github.com/sebmarkbage/d7bce729f38730399d28)


### isDOMComponent

~~~js
boolean isDOMComponent(ReactComponent instance)
~~~

如果物件是 DOM 元件 (例如: `<div>`, `<span>`)就回傳 true。

### isCompositeComponent

~~~js
boolean isCompositeComponent(ReactComponent instance)
~~~

判斷是否為復合式元件(即用 `React.createClass()`)。

### isCompositeComponentWithType

~~~js
boolean isCompositeComponentWithType(ReactComponent instance, function componentClass)
~~~

結合 `isComponentOfType()` 和 `isCompositeComponent()`。

### isTextComponent

~~~js
boolean isTextComponent(ReactComponent instance)
~~~

如果物件是一個純文字元件就回傳 `true`。

### findAllInRenderedTree

~~~js
array findAllInRenderedTree(ReactComponent tree, function test)
~~~

遍歷所有在結構中的元件，並使用傳入的 `test()`。當 `test(component)` 為 true 就把該元件加入陣列。
很少機會會單純使用這個方法，它通常用來對 React 的原生物件做其他的單元測試。

### scryRenderedDOMComponentsWithClass

~~~js
array scryRenderedDOMComponentsWithClass(ReactComponent tree, string className)
~~~

找出在已輸出的 DOM 元件中跟 `tagName` 一致的物件。

### findRenderedDOMComponentWithTag

~~~js
ReactComponent findRenderedDOMComponentWithTag(ReactComponent tree, string tagName)
~~~

類似 `scryRenderedDOMComponentsWithClass()` 不過一定要回傳一個結果，如果沒有或其他數量就例外。

### scryRenderedDOMComponentsWithTag

~~~js
array scryRenderedDOMComponentsWithTag(ReactComponent tree, string tagName)
~~~

找出所有在已輸出樹狀結構中符合 `tagName` 的元件，並傳回一個陣列。

### findRenderedDOMComponentWithTag

~~~js
ReactComponent findRenderedDOMComponentWithTag(ReactComponent tree, string tagName)
~~~

類似 `scryRenderedDOMComponentsWithTag()` ，不過一定要回傳一個結果，如果沒有或其他數量就例外。

### scryRenderedComponentsWithType

~~~js
array scryRenderedComponentsWithType(ReactComponent tree, function componentClass)
~~~

找出在樹狀結構中所有跟 `componentClass` 型別一致的元件。

### findRenderedComponentWithType

~~~js
ReactComponent findRenderedComponentWithType(ReactComponent tree, function componentClass)
~~~

跟 `scryRenderedComponentsWithType()` 一樣，不過一定要回傳一個結果，如果沒有或其他數量就例外。

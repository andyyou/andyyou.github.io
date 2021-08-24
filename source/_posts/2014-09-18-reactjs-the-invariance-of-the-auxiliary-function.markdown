---
layout: post
title: 'React 不變性的輔助函式'
date: 2014-09-18 10:34:00
categories: Program
tags: [reactjs]
---
## 不變性的輔助函式(Immutability Helpers)

在一開始我們必須先對這文鄒鄒的術語做個解釋: 所謂的 Immutability 英文的意思就是不能夠或不輕易受外界影響而改變。而在程式領域中我們舉的例子來說明: 即當資料或物件建立之後就不能或者輕易改變。
以 C# 來說明就是 Array 和 List 的關係，`Object-C` 的 NSArray 和 NSMutableArray 對應的關係。

<!--more-->

React 允許您使用任何方式來管理資料，包含可變性，或不變性的資料或物件。然而如果您能夠在一些影響效能關鍵的地方使用不變性的資料且判斷是否要執行更新，將會有助您提升效能。
通常使用 `shouldComponentUpdate()` 搭配靜態資料能有效的提升程式的執行效率。

> `shouldComponentUpdate()` 被調用在 props 或 state 收到新值後 `render()` 之前，如果回傳 true 就更新， false 就不動。

在原生 Javascript 中處理不變性資料比起其他中介語言更加困難例如: [Clojure](http://clojure.org/)。
好消息是官方提供了我們一個方便的輔助函式`update()`，它可以協助我們處理這類的資料。

在這邊我們補上一個實作範例:

~~~js
/**
 * @jsx React.DOM
 */
var Car = React.createClass({
  getInitialState: function () {
    return {
      wheel: 4
    }
  },
  getDefaultProps: function () {
    return {
      brand: {name: 'Toyota'}
    }
  },
  handleClick: function () {
    this.setProps({brand: {name: 'Audi'}});
    // this.setState({wheel: 1}); /* 只要改變 props 或 state 都會觸發 shouldComponentUpdate */
  },
  shouldComponentUpdate: function (nextProp) {
    console.log(this.props.brand);
    console.log(nextProp.brand);
    console.log(this.props.brand === nextProp.brand)
    return true;
  },
  render: function () {
    return (
      <div onClick={this.handleClick}>
        {this.props.brand.name}
      </div>
    )
  }
});

React.renderComponent(
  <Car />,
  document.getElementById('example')
);
~~~

## 主要的概念
舉例來說如果您的資料是這樣:

~~~js
myData.x.y.z = 7;
// 或者...
myData.a.b.push(9); // { a: { b: [ 9 ] } }
~~~

由於上一次的物件或資料已經被覆蓋了，您沒有辦法知道哪些資料發生變更。於是，您需要建立一個新的 `myData` 副本，只改需要改的地方。
實務上您應該常常會使用  `===` 在 `shouldComponentUpdate()` 去比對新舊物件/資料。下面我們繼續回到資料處理的部分即上面實作範例 `handleClick` 那邊


~~~js
var newData = deepCopy(myData);
newData.x.y.z = 7;
newData.a.b.push(9);
~~~

不幸的是，如果真的複製每一個環節那肯定多做很多不必要的操作，效能自然不好，而且有些狀況下不容易做到。
我們的確可以透過另一種方式: 只複製那些我們要改變的物件，保留那些沒變的物件。
不過這在今天的 Javascript 非常麻煩

~~~js
var newData = extend(myData, {
  x: extend(myData.x, {
    y: extend(myData.x.y, {z: 7}),
  }),
  a: extend(myData.a, {b: myData.a.b.concat(9)})
});
~~~

上面這段虛擬碼大略跟您使用 Underscore extend 方法意思相同: [http://underscorejs.org/#extend](http://underscorejs.org/#extend)
看起來上面這種做法的效能會很不錯(因為他只有淺層複製了 `log n` 個物件，其他的沿用)。不過這麼做其實很痛苦，因為格式看起來是不斷重複一樣的東西。
這樣不止很煩，而且很容易出 bug。

`update()` 提供了另一種簡單的語法糖衣將這個模式從新包裝，使其看起來清楚一些，我們的程式碼將會是如下:

~~~js
var newData = React.addons.update(myData, {
  x: {y: {z: {$set: 7}}},
  a: {b: {$push: [9]}}
});
~~~

雖然語法還是有一點點複雜，需要一點時間適應的(這種寫法的靈感來自 MongoDB 查詢語法)。這樣的做法沒有太多多餘重複的程式碼。
關於 `$` 前綴字這邊稱之為`指令`，而要改變的資料結構稱為`目標`。

## 指令列表

* `{$push: array}` `push()` 所有項目至陣列
* `{$unshift: array}` `unshift()` 就是插入所有項目到陣列
* `{$splice: array of arrays}` 使每個陣列呼叫 `splice()`
* `{$set: any}` 完整取代資料
* `{$merge: object}` 合併物件
* `{$apply: function}` 傳入目前值到函式並取得新值

## 範例

### 簡易的使用 push

~~~js
var initialArray = [1, 2, 3];
var newArray = update(initialArray, {$push: [4]}); // => [1, 2, 3, 4]
~~~

### 巢狀集合

~~~js
var collection = [1, 2, {a: [12, 17, 15]}];
var newCollection = update(collection, {2: {a: {$splice: [[1, 1, 13, 14]]}}});
// => [1, 2, {a: [12, 13, 14, 15]}]
~~~

存取集合中索引為 2 ，然後底下的 a 物件執行 `splice()` 函式，前兩個參數分別是起始索引，第二個參數是要取代幾個元素，後面則是要加入的元素 `13`, `14` 。


### 使用 function 來更新值

~~~js
var obj = {a: 5, b: 3};
var newObj = update(obj, {b: {$apply: function(x) {return x * 2;}}});
// => {a: 5, b: 6}
// This is equivalent, but gets verbose for deeply nested collections:
var newObj2 = update(obj, {b: {$set: obj.b * 2}});
```
### 淺層合併
```js
var obj = {a: 5, b: 3};
var newObj = update(obj, {$merge: {b: 6, c: 7}}); // => {a: 5, b: 6, c: 7}
~~~

## 補充- 關於淺層複製與深層複製(Shallow copies v.s. Deep copies)
淺層複製盡可能的只複製少量資訊，一個集合的淺層複製只複製結構，不複製元素，當您使用淺層複製意味著兩個物件共同參考到同一個元素記憶體位置的意思。
而深層複製，複製了所有東西，簡單說就是真的把所有東西複製一份。

![](http://i.imgur.com/fy0Kt8a.png)

---
layout: post
title: '深入 React 生命週期'
date: 2015-12-17 05:00:00
categories: Javascript
---

![](http://i.imgur.com/ZXhnb9I.png)

## 各種情況的生命週期流程

大致上元件執行生命週期方法的情形可分為四種

1. 初始化，第一次 render
  - getDefaultProps()
  - getInitialState()
  - componentWillMount()
  - render()
  - componentDidMount()
2. props 發生改變時
  - componentWillReceiveProps(nextProps)
  - shouldComponentUpdate(nextProps, nextState)
  - componentWillUpdate(nextProps, nextState)
  - render()
  - componentDidUpdate(prevProps, prevState)
3. state 發生改變時
  - shouldComponentUpdate(nextProps, nextState)
  - componentWillUpdate(nextProps, nextState)
  - render()
  - componentDidUpdate(prevProps, prevState)
4. 元件 unmount 卸載時
  - componentWillUnmount()

* 當遇到複合式元件的狀況時，子元件的生命週期從父元件 render 之後開始發動
* 一旦父元件發生改變，子元件的 componentWillReceiveProps 還是會觸發。也就是每次父元件更新，子元件都會重新渲染(Update流程)
* 當使用 `React.createClass` 建立一個 component 時， `render` 是生命週期方法中必須的。
* 當 render 被呼叫時會去檢查 this.props, this.state 並且只能回傳一個 component 意即其他元素都要包在這個元件底下。
* render return `null` 或者 `false` 則表示不渲染輸出，背地裡 React 則是輸出 `<noscript>` 標籤。
* 不可以在 render 裡面操作修改 state 和 props。
* getInitialState 一定要回傳物件或 null，在元件被 mounted 之前僅被呼叫一次。
* getDefaultProps 如果回傳一個`物件`，會和所有元件的實例物件共用，傳址不是傳值。

## 第一次初始化的流程

0. displayName 用來設定元件名稱
1. getDefaultProps() 當元件類別被建立時就會觸發，且和所有實例物件共享
2. getInitialState() 開始渲染元件時初始化
3. componentWillMount() 準備掛載 DOM 之前觸發
4. render() 執行輸出
  - 如果裡面有其他的子元件則依據上面的流程 getInitialState() -> componentWillMount() -> render() -> componentDidMount()
5. componentDidMount() DOM 完成之後立即觸發(會等到子元件都完成才觸發)

## 更新的流程

1. componentWillReceiveProps(nextProps) 這邊使用 setState 不會再觸發一次 render。初始化不會被觸發
2. shouldComponentUpdate(nextProps, nextState) 元件是否要更新
3. componentWillUpdate(nextProps, nextState) 將要更新之前，不可以用 setState
4. render()
5. componentDidUpdate(prevProps, prevState)

## 呼叫 setState 不會 re-render 的方法

* componentWillMount()
* componentWillReceiveProps()

## 存取 DOM 的適當時機

* componentDidMount()

## 不得使用 setState 的事件

* shouldComponentUpdate(nextProps, nextState)
* componentWillUpdate(nextProps, nextState)
* render()

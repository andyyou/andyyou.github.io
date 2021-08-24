---
layout: post
title: 'React 附加組件介紹'
date: 2014-09-13 10:15:00
categories: Program
tags: [reactjs]
---
## 附加組件(Add-ons)
官方會把一些實用的共用工具放置在 `React.addons`。這些工具應該暫時被視為還在實驗階段，但最後官方應該會將其整合進核心功能或者如下列共用的工具函式庫:

<!--more-->

* `TransitionGroup` 與 `CSSTransitionGroup` 是用來處理關於動畫和過場特效，不過它們通常不是這麼容易實作。
* `LinkedStateMixin` 簡化處理使用者表單欄位和元件狀態之間的一些狀況。
* `classSet` 讓操作 DOM 的 CSS 字串時的程式碼較簡潔。
* `TestUtils` 簡單的測試工具。
* `cloneWithProps` 複製 React 元件與修改 `props`。
* `update` Javascript 函式，簡化處理不可變動的資料的過程。

下面列出的項目只存在 React 開發版本中:

* `PureRenderMixin` 適用於特定情況下改善效能。
* `Perf` 用來測量效能並提供優化的建議。

當您需要使用這些附加組件時，請使用 `react-with-addons.js` ，而不是單純使用 `react.js`。

---
layout: post
title: 'React 與 DOM 的差異'
date: 2014-09-14 13:44:00
categories: Program
tags: reactjs
---
## DOM 的差異

React 為了跨瀏覽器和提升效能的因素，實作一套和瀏覽器本身無關的 events 以及模擬 DOM 的機制。我們可以借由這個機制處理一些關於原始 DOM 設計上一些不足的地方。

<!--more-->

* 所有的 DOM 屬性 `Properties` 和 `Attributes` (包含事件)都應該使用駝峰式命名 `camelCased` ，這和一般的 Javascrpt 程式碼風格一致。我們故意在這邊違背 html 規格 ，因此這和 html 規格是不同的。

* `style` 屬性透過 Javascript 物件和駝峰式的屬性來設定，而不是 CSS 字串。所以設定 CSS 的語法風格會和 DOM, Javascrit 屬性一致，外加這麼做可以防止 XSS 攻擊。

* 所有在事件符合 W3C 規範，且所有事件(包含 submit)傳遞都遵照 W3C 規範，查閱 [Event System](http://facebook.github.io/react/docs/events.html) 取得更多資訊。

* 關於 `onChange` 事件行為就跟你所期待的一樣，當一個表單欄位改變了，事件就會被觸發，而不是在 `onblur` 失去焦點的時候才觸發。 我們特意違背現有的瀏覽器行為，因為原始的 `onChange` 事件行為跟其名稱並不符合，React 需要正確的用到這個 Event ，當使用者輸入資料的同時 React 就會及時反應。查閱[Forms](http://facebook.github.io/react/docs/forms.html)得知更多資訊。

* 表單輸入的屬性例如 `value` `checked` 更多關於一些命名，用法，等請查閱 [Forms](http://facebook.github.io/react/docs/forms.html)

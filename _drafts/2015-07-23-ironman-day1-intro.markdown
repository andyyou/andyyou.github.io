---
layout: post
title: '第一次寫 Javascript ES 2015 就上手 No1 - 前言'
date: 2015-07-20 05:30:00
categories: Javascript
---

# 前言

學習 Javascript 的過程比起其他語言老實說對於那些有過其他語言經驗的來說，感覺是不好的，至少我是這麼覺得。
因為 Javascript 是一個持續進化快速的語言，或者說一開始就沒有辦法規劃清楚所有東西，也因為其歷史背景導致有挺多缺失的。
其過程就如同下面這張圖

![](http://curtis.lassam.net/comics/cube_drone/159.gif)

不管你是願不願意，是不是個積極地開發者，你都會被逼著往前走。然後就會問自己：我到底打到什麼時候是個頭啊？

現實是殘酷的，很不幸的身為 Web Developer 我們逃離不了這個宿命，於是乎回到主題。一眨眼又過了一年，原本在上一次的鐵人賽心中盤算要開始撰寫系列關於 Meteor.js 相關文章，但世事多變因為這陣子的開發工作完全沒機會玩 Meteor 幾乎都沈浸在 React 的世界中。
又因為在使用一些開放函式庫的過程中常常就遇到一些神奇的黑魔法(語法)，舉例來說 `document.querySelectorAll(".hey")::find("p")::html("hahaha");` 哇勒! 這到底是 C 還是 Ruby 啊？ 一些新功能還有點像 C# 或 Python 這真的是 Javascript 嗎？

而通常在一番 Google 之後就會發現原來這是 ES6(ES2015) 的新語法，甚至還有 ES7 的功能，此時我才驚覺大家都開始把這些新功能用在專案上了，是時候花些時間認真看看了。
最後小弟今年就打算從 Webpack, BabelJS, ES6, 7 這三個大軸心來紀錄一系列筆記。當然會選這些主題主要都是我還不熟的，除了參賽也希望藉此給自己一些壓力把欠的債給讀一讀。

我會先從封裝打包工具 Webpack 開始，因為這樣才能夠協助我們撰寫範例，搭配 BabelJS 來實作 ES6 或 ES7 的新功能。

# 大綱

* Bundler using Webpack
* Introduce Babel, Traceur, Broccoli and others
* Before start 
  - [參考](http://developer.telerik.com/featured/six-steps-for-approaching-the-next-javascript/?utm_source=javascriptweekly&utm_medium=email)
* Unicode
* Template Strings
  - [參考](http://www.sitepoint.com/understanding-ecmascript-6-template-strings/)
* Let + Const
* Arrows
* Classes
* Enhanced Object Literals
* Destructuring
* Default + Rest + Spread
* Iterators + For..Of 
* Generators
* Comprehensions
* Math + Number + String + Object APIs
* Modules
  - CommonJS, AMD
  - ES6 Module
* Module Loaders
* Map + Set + WeakMap + WeakSet
* Proxies
* Symbols
* Subclassable Built-ins
* Binary and Octal Literals
* Promises I [參考](http://liubin.github.io/promises-book/#chapter1-what-is-promise)
* Promises II
* Reflect API
* Tail Calls
* Bind (::) operator
* Decorator
  - [參考](http://efe.baidu.com/blog/introduction-to-es-decorator/)
* Conclusion

[列表](http://es6katas.org/?utm_source=javascriptweekly)

# 期許

老實說部分新功能已經常常被小弟我用在專案上了，但還有挺多是沒用過，這次希望能夠用 30 天持續的記錄。讓自己可以更精進同時也希望能夠幫助到其他正需要這方面資料的開發者。

# 資源

即便是第一天我相信還是有些人非常急性子想學些東西，這邊整理了我在這一系列文章所參考閱讀的文章，希望對您有所助益。

* [SitePoint JS 相關文章](http://www.sitepoint.com/javascript/raw-javascript/)
* [ES6 In Depth](https://hacks.mozilla.org/category/es6-in-depth/)
* [Babel JS 文件](https://babeljs.io/docs/learn-es2015/)
* [初探 ES6 Harmony 的黑歷史](http://www.codedata.com.tw/javascript/introducing-es6-1-harmony-history)
* [ECMAScript 6 入门](http://es6.ruanyifeng.com/#README)
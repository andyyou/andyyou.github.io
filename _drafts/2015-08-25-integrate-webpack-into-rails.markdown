---
title: '整合 webpack & ES6 到 Rails'
layout: post
date: 2015-08-25 12:00:00
categories: rails
---

您是否曾經:

1. 思考是否有更好的方式整合 Javascript Client Framework 到已存在的 Ruby on Rails 專案?
2. 在整合 Javascript 函式庫或範例的時候感到困惑, 不知道該怎麼處理 Javascript 模組比較恰當?
3. 遭遇過因為 Javascript 混雜在同一個全域而產生的問題?
4. 聽聞關於 ES6 想要在 Rails 專案中使用 ES6 的語法?

我們會在一個 Rails 專案中完成下面的目標:

1. UI 具有豐富的互動性, 當我們編輯 JS 和 CSS/Sass 在存檔的瞬間立即看到我們的修改且不是整個頁面重新載入
2. 透過 package.json 和 npm 指令, 直接就能在 JS 中使用 require/import 輕鬆的使用 Node 生態圈的函式庫和資源
3. 無縫整合 Node 的 JS 資源到 Rails Asset Pipeline, 因此並沒有完全捨棄 Asset Pipeline 而是並存並善用彼此的優點
4. 無縫將 Node 客戶端資源匯整到已存在的 Rails 專案
5. 可以使用任何 JS 工具, 例如 React JSX tranpiler, ES6, Babel 等等

這篇文章將會介紹如何在 Rails 專案中利用 webpack 協助我們開發並完成上面列出的目標

首先, 我想要分享一個簡短的故事, 這故事是關於我是怎麼必須要`實現讓 Rails 可以和 JS 生態圈資源整合的更好`


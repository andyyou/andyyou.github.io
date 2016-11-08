---
title: 模組系統, Commomjs 和 require 的運作機制
categories: Program
tags: [javascript, nodejs]
---

這篇筆記試圖說明 Node 中的模組系統，CommonJS 運作機制和底層 require 的機制。
<!--more-->

# CommonJS 的出現

在 ES2015 標準出現之前 Javascript 並沒有原生的方式來組織專案中的程式碼。Node 透過 CommonJS 模組標準來填補這個空缺。
在這篇文章我們會說明 Node 的模組系統是如何運作的，以及我們可以如何使用新的 ES 標準來組織模組。

# 何謂模組系統

模組是程式碼結構的基石。模組讓我們可以組織程式碼隱藏細節資訊只透過 `module.exports` 開放我們需要使用的部分（public interface）。每一次當我們使用 `require` 時，我們就可以載入模組（等同於將其他檔案的程式碼匯入使用）。

最簡單的 CommonJS 範例如下

```js
function add (a, b) {
  return a + b
}

module.exports = add
```

接著當要使用 `add` 模組的時候我們可以透過 `require` 來將其匯入。

```js
const add = require('./add')

console.log(add(4, 5)) // 9
```

在底層，`add.js` 會被 Node 透過下面這種方式包起來：

```js
(function (exports, require, module, __filename, __dirname) {
  function add (a, b) {
    return a + b
  }

  module.exports = add
})
```

這也是為什麼您可以類似於存取全域變數一般達到匯入與模組的功能。
同時這也確保您的變數在模組的存取範圍（scope）內而不是全域。

# require 的運作機制

模組在 Node 載入機制第一次載入時是會快取（cache）。這表示不管您呼叫 `require('awesome-module')` 多少次，都會取得相同的物件實例。

---
layout: post
title: 'requestAnimationFrame 筆記'
date: 2016-05-12 12:00:00
categories: Program
tags: [javascript]
---

在 Javascript 我們曾經只有一種方式來處理跟特定時間有關的循環事件: `setInterval()`。簡單說就是當你需要重複某些任務時(依據時間)。
您就會用到這個方法。對於動畫來說需要每秒 60 個 `frames` 人類才會覺得是順暢平滑的，因此我們可能會寫出像下面這樣的程式碼

<!--more-->

```js
setInterval(function () {

}, 1000/60)
```

除了這個方式，還有另外一種方式就是使用 `requestAnimationFrame` ，當然這個東西已經出現很久了，不過在這之前，大多的時候我都不需要自己造輪子，也不太有機會需要對於動畫有深入的處理。
因為自己開發一些輪子的需求所以這篇文章會了解一些關於 `requestAnimationFrame` 的基本用法。

簡單說使用 `requestAnimationFrame` 有下面幾點好處

* 瀏覽器可以優化，讓動畫更平滑順暢
* 該暫停的動畫不會繼續使用 CPU
* 省電

# 最簡單的範例

```js
function repeatOften () {
  requestAnimationFrame(repeatOften)
}

requestAnimationFrame(repeatOften)
```


# 啟動/停止

`requestAnimationFrame` 會回傳一個 `ID` 我們可以用這個 ID 來取消它，就類似 `setTimeout`, `setInterval`。
下面我們用 jQuery 簡單的展示一個範例

```js
var globalID;

function repeatOften () {
  $('<div />').appendTo('body');
  globalID = requestAnimationFrame(repeatOften);
}

$("#start").on("click", function() {
  globalID = requestAnimationFrame(repeatOften);
});

$("#stop").on("click", function() {
  cancelAnimationFrame(globalID);
});
```

# 參考

* [polyfill](https://gist.github.com/paulirish/1579671)
* [CSS-Tricks](https://css-tricks.com/using-requestanimationframe/)

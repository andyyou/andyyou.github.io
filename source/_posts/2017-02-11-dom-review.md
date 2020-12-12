---
title: 'DOM 基礎複習筆記'
date: 2017-02-11 15:25:13
categories: Program
tags:
  - html
  - javascript
---

## DOM

DOM = Document Object Model。DOM 詮釋了 HTML，XML，SVG 上元件的組成建構，操作，是一套程式介面。所有的 DOM 都提供了 API 可以對其操作。實務上指的就是 HTML Tag，文件樹的結構化表示法。

例如：我們使用 HTML 標籤來撰寫，然後瀏覽器會把這些 HTML 轉成 DOM Object 讓我們可以操作。

<!--more-->

## DOM Node（DOM Object） v.s DOM Element

Node 節點通常是對 DOM 結構中任何類型物件的泛稱。Node 可以是任何一種內建的 DOM 元素（Element）e.g. `document`，`document.body`。也可以指特定 HTML tag e.g.`<input>`，`<p>`。Node 即任何一種 DOM Object。

元素 Element 指的是特定類型的 Node（DOM Object）。

更精確的說：Node 的實踐是為了建構樹狀結構，它會具有 `firstChild` `lastChild` `childNodes` 等方法。有些 Node 同時也是 Element。因為 Element 繼承 Node。

```js
> document instanceof Node
true

> document instanceof Element
false

> document.firstChild instanceof Node
true

> document.firstChild instanceof Element
true
```

## Bubbling v.s. Capturing

DOM Element 可以嵌套在其他 DOM Element 之中，而且即便我們點擊的是子元素（嵌套在內部），父元素也會觸發自己對應的處理程序 Handler（Event）。

造成這個結果的原因就是 Event bubbling。

舉例來說下面的 div 的處理程序 i.e. 觸發的事件，就算我們點擊的是子元素 `em` 或 `code` 也會被執行。

```html
<div onclick="alert('div handler')">
  <em>Click here to triggers on nested <code>em</code>, not on <code>div</code></em>
</div>
```

這就是因為 Event bubbles 即事件上升傳遞

## Bubbling 上升傳遞

所謂 Bubbling 的原則或說執行流程就是：
當最底層（最內部）的元素觸發事件之後，接著會照著順序往上層／父／容器元素觸發對應事件。

舉例來說，下面有三層的 div 結構

```html
<div class="d1"><!— 最外層 —>
  <div class="d2">
    <div class="d3"><!— 最內層 —>
    </div>
  </div>
</div>
```
Bubbling 或稱上升傳遞機制，例如 click 事件，當我們點擊 d3 時：
1. 首先確保最內部的 div d3 (同時也稱作 target)的 `onclick` 先被觸發執行。
2. 接著觸發 div d2 的 `onclick`
3. 最後 div d3  的 `onclick`

這樣的行為稱為 Bubbling Order 也有人直翻冒泡。因為事件觸發由最內部往上層就像水中的氣泡一樣。

[0](https://javascript.info/tutorial/bubbling-and-capturing)
[1](https://www.kirupa.com/html5/event_capturing_bubbling_javascript.htm)

## this 和 event.target

當我們觸發最內部元素的事件時，這個最內部的元素又稱為 target 或觸發起始元素。
在 IE 有個 `srcElement` 屬性可以取得，其他遵循 w3c 規範的瀏覽器則使用 `event.target`。

在 Javascript 中 this 簡單來說就是指向調用 function 的物件（可以理解為其別名）

所以在這個 Bubbling 的流程下 this 和 target 分別為：

1. `event.target/srcElement` - 維持同樣的元素
2. `this` - 綁定該事件的元素，會指向現在正處理該事件的事件監聽器所註冊的 DOM 物件。

注意：target 在整個過程中是不變的，this 則會變動。

> 在 w3c 兼容的瀏覽器中 this 也等於 `event.currentTarget`。至於 IE 中的 attachEvent 方法則不會傳遞 `this` 和 `currentTarget`。

## 停止 Bubbling

因為冒泡的機制，當某個事件被觸發後就會一路往上傳直到 `<html>` 才會停止。但很多時候這會造成不預期的結果。因此我們需要停止傳遞的方法。

* w3c 兼容瀏覽器：event.stopPropagation()
* IE < 9 : event.cancelBubble = true

假如同一個事件綁定多個處理函式，其中一個函式停止了傳遞，其他綁定的函式仍然會執行。另外綁定多個函式瀏覽器並不能確保其執行的順序。

## Capturing 由上至下傳遞

在所有瀏覽器，除了 IE < 9。事件在處理執行時都有兩個階段。

![](https://javascript.info/files/tutorial/browser/events/event-order-w3c.gif)

根據 w3c de 規範觸發流程會先由上至下稱為 Capturing，接著由下至上稱為 Bubbling。

預設，所有 Capturing 階段的事件處理函式會被忽略。除非我們使用 `addEventListener` 最後的參數為 `true`，那麼就只會執行 Capturing 階段的事件，Bubbling 階段的觸發就會被忽略。

> 在實務上 Capturing 很少被使用。不過有些事件不會冒泡傳遞，例如：onfocus/onblur

## 總結

* 事件觸發的順序是先 Capturing 階段至最內部的元素，接著 Bubbling。除了 IE<9 只有 Bubbling
* 除非 addEventListener 最後的參數使用 true 否則 Capturing 階段的觸發都會被忽略。
* 可以使用 event.stopPropagation() 或 event.cancelBubble = true 來停止傳遞觸發。

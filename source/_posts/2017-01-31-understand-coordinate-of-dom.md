---
title: 理解 DOM 座標
date: 2017-01-31 19:38:10
categories: Program
tags:
  - javascript
---

## 座標系統

在瀏覽器中有兩種座標系統 & 滑鼠座標：
    1. 相對於 `document` - 座標 (0, 0) 在整個頁面的最左上角。
    2. 相對於 `window` - 座標 (0, 0) 在可視區 Viewport 的最左上角。
    3. 滑鼠座標 - 通常透過事件取得。

當頁面還沒捲動時 window 和 document 的原點 (0, 0) 與其他座標是相同的。

開始捲動後，document 的座標等於 window（viewport） 的座標加上 scroll 的位置，大部分的情況下我們會使用 document 座標系統，因為它即便 scroll 了，還是會保持一致。

<!--more-->

## 元素座標與 offsetParent

一個元素的座標位於該元素的左上角，不幸的是沒有屬性可以直接取得元素對應 document 的座標。不過可以透過計算 offsetTop/offsetLeft 和 offsetParent 來取得。

offsetParent 為元素的一個唯讀屬性，其值為上層最接近的有設定 position 的容器元素。更精確的說是 position 不為 static 的元素。當沒有任何 positioned 的元素時最接近的 table cell 或根元素為 offsetParent。HTML 兼容模式 body 為 offsetParent。`display: none` 時 offsetParent 為 null。

一個簡單的計算方式就是遍歷所有 offsetParent 計算 offsetTop/offsetLeft。簡單說就是不斷累加自己在父元素（positioned）的座標/距離，最終到達 document 則完成計算。

```js
function getOffset (el) {
  var top = 0, left = 0
  while (el) {
    top += parentInt(el.offsetTop)
    left += parentInt(el.offsetLeft)
    el = el.offsetParent
  }

  return {
    top: top,
    left: left
  }
}

```

這種方式有 2 個缺點：
1. 不同瀏覽器行為可能不同。因為 border 和 scroll 是否加入計算造成差異。
2. 慢！因為要遍歷所有 offsetParent。

## 正確的方式 el.getBoundingClientRect()

這個方法時 w3c 標準，並且幾乎所有新版的瀏覽器都支援。它會回傳封閉該元素的一個矩形(CSS border-boxes)，而這個矩形會透過物件的形式包含 top, left, right, bottom 等資料傳回。

再次強調 - 不幸的是沒有屬性可以直接取得元素對應 document 的座標。這個 getBoundingClientRect() 取得的資訊是相對於 window 不是 document。

根據 CSS 規範，任何內容都會被放置在 CSS Box 中。像是 div 這類的 Box 又稱 block box，這類的 Box 原本就是一個矩形。如果元素是 inline 的話會牽扯到 inline box，line box，containing box，contain area。就會有多個的矩形來組織。又稱 [anonymous block box](https://www.w3.org/TR/CSS21/visuren.html#anonymous-block-level)

{% asset_img line-box.jpg %}

使用 getBoundingClientRect()

```js
function getOffset (el) {
  // (1)
  var box = el.getBoundingClientRect()
  var body = document.body
  var docEl = document.documentElement

  // (2)
  var scrollTop = window.pageYOffset || docEl.scrollTop || body.scrollTop
  var scrollLeft = window.pageXOffset || docEl.scrollLeft || body.scrollLeft

  // (3)
  var clientTop = docEl.clientTop || body.clientTop || 0
  var clientLeft = docEl.clientLeft || body.clientLeft || 0

  // (4)
  var top = box.top + scrollTop - clientTop
  var left = box.top + scrollLeft - clientLeft

  return {
    top: Math.round(top),
    left: Math.round(left)
  }
}
```

1. 取得矩形與元素等相關座標資訊
2. 計算 page scroll。所有瀏覽器器除了 IE < 9 外都支援 `pageXOffset/pageYOffset`。在 IE 當宣告了 DTD i.e DOCYPTE 則 scroll 位置可以透過 documentElement（<html>）取得，否則就從 body 取。
3. 在 IE < 8 document 或 body 會具有特殊的偏移，所以我們需要取得偏移並扣掉。但要注意的是  clientTop/clientLeft 同時也表示 border 寬，一般來說我們不會替 document 或 body 設定 border，在這種情況下 IE 的document.documentElement.clientTop 不是 0 而是 `2px`。因此我們要減掉。
4. 加入捲動的距離，然後扣掉位移即是該元素相對於 document 的座標。

針對 Firefox 我們需要而外使用四捨五入 Math.round()

## scrollTop

取得已經捲動多少距離

* document.body.scrollTop（Undefined DTD，兼容所有瀏覽器）
* document.documentElement.scrollTop（DTD， Chrome, Safari 為 0）
* window.pageYOffset（FF，Chrome，IE9+，Opera）
* window.scrollY（不推薦使用）

## DTD

DTD（Document Type Definition）文件類型定義，定義 XML 的元素，結構，屬性。
使用 DTD 可以讓不同的使用者在交換資料時明白其資料格式的標準，而程式可以使用 DTD 來驗證 XML 是否正確。

在 HTML 中 `document.compatMode` 可以得知是否宣告了 DTD，值為 `BackCompat` 為未宣告，`CSS1Compat` 為宣告 DTD。

## MouseEvent 座標

 * screenX/Y - 相對於螢幕左上角的座標
 * pageX/Y - 相對於頁面左上角的座標（IE9 不支援）包含 scroll
 * clientX/Y - 相對於瀏覽器左上角的座標，不含 scroll
 * layerX/Y - 首先當元素的 position 不是 static 我們稱為定位元素（positioned）。觸發事件的元素相對於父容器定位元素或根元素的座標，Chrome 從 border 開始計算，Firefox 則從 padding 邊界，計算過程包含 scroll。
 * offsetX/Y - 滑鼠點擊點相對於目標元素 padding 邊界的座標，點擊到 border 區域時會是負值。
 * movementX/Y - 上個 mousemove 座標與當前的 mousemove 座標移動距離。currentEvent.screenX - previousEvent.screenX

## window 尺寸

* window.innerWidth
* window.innerHeight
* window.outerWidth
* window.outerHeight
* window.pageYOffset
* window.pageXOffset
* window.scrollX
* window.scrollY
* window.screenX
* window.screenY

## DOM 尺寸

* clientWidth / clientHeight
* scrollWidth / scrollHeight
* offsetWidth / offsetHeight
* scrollTop / scrollLeft
* offsetTop / offsetLeft
* clientTop / clientLeft
* offsetParent
* getBoundingClientRect()
* video.videoWidth
* video.videoHeight

## 小結

client 前綴的概略為相對於瀏覽器 viewport 或指內部 padding area。
offset 前綴的概略為相對於父定位元素。
scroll 前綴的概略為相對於完整頁面。
screen 相對於螢幕。

## 資源

* [Javascript coordinates](https://javascript.info/tutorial/coordinates)

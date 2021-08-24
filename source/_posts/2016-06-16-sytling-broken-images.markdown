---
layout: post
title: '[譯] 美化無法載入的圖片 <img>'
date: 2016-06-16 12:00:00
categories: Program
tags: [css]
---

當網頁中無法載入圖片(通常是圖片連結失效或錯誤)時，瀏覽器預設會幫我們顯示一個小小的圖示告知我們連結失效了。

![](http://localhost:3000/)

這些預設圖片每一個瀏覽器都不太一樣，通常也不太好看。這篇文章主要介紹的內容就是我們可以透過 css 來美化這個預設行為。

<!--more-->

# 關於 <img> 元素的兩件事

為了了解如何美化`壞掉的圖片`，有兩個關於 `<img>` 的行為我們必須先瞭解。

1. 能夠套用文字相關的樣式，這些樣式只會被套用在替代文字上。在圖片正常載入的情況下則沒有任何效果。
2. `<img>` 是一個[replaced element](https://developer.mozilla.org/en-US/docs/Web/CSS/Replaced_element)，例如 `<video>`, `<object>` 等等，大略的意思就是這個元素呈現的外型尺寸是透過外部資源來決定的，像是圖片就是由實際圖片來決定，想知道更詳細的說明可以參考[Sitepoint 提供了更好的說明](http://reference.sitepoint.com/css/replacedelements)。因為元素是透過外部資源來控制的，所以預設來說 `:before` 和 `:after` 這些偽元素(pseudo-elements) 應該是不能運作的，不過當圖片失效或是無法載入的時候這些偽元素就可以作用了。

因為上面這兩個原理所以我們能夠套用一些樣式只出現在 `<img>` 無法載入圖片的時候。

# 實作

假設我們現在有一個圖片標籤

```html
<img src="http://andyyou.github.io/broken.png" alt="andyyou">
```

### 提供有意義的資訊

第一個重點就是我們需要設定 `alt` 為的就是在圖片失效的時候提供給使用者一些關於這張圖片的描述資訊。接著為了處理圖片失效的問題我們也需要提供一些資訊告訴使用者這張圖片壞了。另外就是可以透過 `attr()` 取得相關的資訊。

```scss
img {
  font-family: 'Helvetica';
  font-weight: 300;
  line-height: 1.4em;
  text-align: center;
  width: 100%;
  height: auto;
  display: block;
  position: relative;

  &:before {
    content: "We are sorry, the image is broken :(";
    display: block;
    margin-bottom: 10px;
  }

  &:after {
    content: "(url: " attr(src) ")";
    display: block;
    font-size: 12px;
  }
}
```

### 取代預設的替代文字

透過偽元素我們可以取代預設的`替代文字 alt text`，原理就是把偽元素蓋在預設替代文字的上方。

```css
img:after {  
  content: "\f1c5" " " attr(alt);

  font-size: 16px;
  font-family: FontAwesome;
  color: rgb(100, 100, 100);

  display: block;
  position: absolute;
  z-index: 2;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #fff; /* 如果透明或不設定的話就露餡了 */
}
```

### 其他樣式

到了這邊其實我們知道有兩個偽元素可以操作，除了顯示資訊之外其實也可以用來單純作樣式的設計

![](http://i.imgur.com/gPvl8jf.png)

```scss
img {
  font-family: 'Helvetica';
  font-weight: 300;
  line-height: 2em;
  text-align: center;
  width: 100%;
  height: auto;
  display: block;
  position: relative;
  min-height: 50px;

  &:before {
    content: " ";
    display: block;
    position: absolute;
    top: -10px;
    left: 0;
    height: calc(100% + 10px);
    width: 100%;
    background-color: rgb(230, 230, 230);
    border: 1px dotted rgb(200, 200, 200);
    border-radius: 5px;
  }

  &:after {
    content: "\f127" " Broken Image of " attr(alt);
    display: block;
    font-size: 16px;
    font-style: normal;
    font-family: FontAwesome;
    color: rgb(100, 100, 100);
    position: absolute;
    top: 5px;
    left: 0;
    width: 100%;
    text-align: center;
  }
}
```

# 瀏覽器支援

很不幸的並不是所有瀏覽器都完整支援 `alt text`, `:before`, `:after` 套用樣式。所以在部分瀏覽器上這個作法可能不被支援。

# 來源

* [Styling broken images](https://bitsofco.de/styling-broken-images/)

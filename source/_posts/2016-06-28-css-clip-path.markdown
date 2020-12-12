---
layout: post
title: "[譯] 學習 CSS clip-path 屬性"
date: 2016-06-28 12:00:00
categories: Program
tags: [css]
---

整體來說網頁主要是由`矩形`所構成的，另一方面印刷品則具備相對多樣性。造成這樣差異的原因有很多，不過其中一個即是缺少合適的工具。
這篇文章主要會介紹 `clip-path` 這個 css 的樣式規則可以讓我們遮掉元素的局部，就是不顯示出來。達成這項功能背後的原理就是我們可以透過它去定義可視區，類似於遮色片的概念。我們將從基礎開始然後涵蓋語法，進一步理解進階的概念。

<!--more-->

# 基礎

clip 意思是剪裁，意味著修剪某物的外型。在網頁設計的情況下，是讓我們可以去決定某個元素完全隱藏或局部隱藏。而在這篇文章中另外兩個相關的概念是 `clipping path` 和 `clipping region`。

`clipping path` 剪裁路徑意思就是我們使用它去修剪一個元素，透過路徑組成 `clipping region` 剪裁範圍。這個範圍或稱區域可以是簡單的形狀或複雜的多邊形。剪裁範圍就是透過一個封閉的剪裁路徑所組成的，這與您使用 Illustrator 的鋼筆工具繪製形狀的概念是一樣的。

任何在形狀範圍外的東西都會被瀏覽器裁掉。不只包含背景其他像是內容，borders，text-shadow 效果都會被裁掉。此外瀏覽器甚至不會處理範圍外的事件像是 hover, click 等。

即便我們設定的元素不再是矩形，但周圍的元素排列方式仍然維持原本矩形的佈局。為了達成周圍的元素跟著裁切的形狀，我們可以使用 [shape-outside](https://www.sitepoint.com/css-shapes-breaking-rectangular-design/) 屬性。

同時記住不要把舊的樣式 [clip](https://www.w3.org/TR/css-masking-1/#clip-property) 和 `clip-path` 等搞混了。舊有的屬性只支援矩形的裁切。

# 使用

目前這個樣式規則的語法如下：

```css
clip-path: <clip-source> | [ <basic-shape> || <geometry-box> ] | none
```

上面語法表示的是：

* `clip-source` 會是一個 URL 參考到一個 SVG `<clipPath>` 元素，這個元素可以是在檔案內部的或外連的。
* `basic-shape` 基本形狀的 function ，您可以在 [CSS Shapes 規範](https://www.w3.org/TR/css-shapes-1/#typedef-basic-shape) 中找到並使用。
* `geometry-box` 可選的參數。搭配 `basic-shape` 函數，它的用途是設定整張畫布的基準點，定義裁切從哪範圍開始。如果我們自訂 `geometry-box` 那裁切路徑便會照著我們定義的形狀，包含任何圓角設定 `border-radius` 。這樣說有點模糊，稍後我們會詳細介紹。

現在，讓我們來思考下面這段 css 程式

```
img {
  clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%);
}
```

![](http://i.imgur.com/b1Y2MbL.png)

這段設定將會裁切所有圖片為一個菱形。不過為什麼圖片會變成菱形呢？為什麼不是梯形或平行四邊形？您應該猜到了，這是因為形狀取決為我們定義的頂點。下面圖片說明了建立裁切形狀時使用的慣例

每個座標的第一個參數指定了該座標的 x 軸，第二個參數則是 y 軸。所有的點依照順時鐘的方向繪製。依照這個規則與上圖我想您應該理解了。

> 如果您正在使用 CodePen 測試這些語法，請注意上面的程式範例僅在 Chrome 並且需要使用 `prefix` 才能正常運行，您可以打開 Auto Prefix 功能。

[CodePen](http://codepen.io/andyyou/pen/MepjaY)

### 使用 geometry-box 裁切元素

當我們要裁切 HTML 元素時，`geometry-box` 可以是下列值 `margin-box`, `border-box`, `padding-box`, `content-box`

```css
.el {
  clip-path: polygon(10% 20%, 20% 30%, 50% 80%) margin-box;
  margin: 15px;
}
```

上面的範例，`margin-box` 會決定我們得座標點從 `margin` 的範圍開始，而 `(10%, 10%)` 就是我們實際內容的左上角，因此 clip-path 也必須對應計算。

SVG 元素的狀況下，參數可能是 `fill-box`, `stroke-box`, `view-box` 。其中 `view-box` 會最接近整個 svg 的可視範圍。

# clip-path 的使用方式

關於這個屬性有很多有趣的使用方式。第一個就是它可以美化文字內容。您可以觀察一下下圖，我們透過背景色與裁切所營造出的感覺不再受限與單純的方形。

![](http://i.imgur.com/mp1esMs.png)

簡單說您可以簡單的使用背景色，漸層等等您已經熟悉的屬性然後搭配 clip-path ，例如上面粉紅色背景類似泡泡對話框的效果，從前我們可以要透過 border 組成三角形在透過 transform 和調整位置等屬性去完成，現在換成 clip-path 只需要一行，我們可以輕鬆完成任何不規則的形狀。

```css
.msg {
  clip-path: polygon(0% 0%, 100% 0%, 100% 75%, 75% 85%, 75% 100%, 50% 80%, 0% 75%);
}
```

當然您可以用來裁切圖片使其變成各種不同的形狀，您的相片圖庫的頁面不再只能用矩形呈現。下面是一個範例建議您使用 Chrome 玩玩

[CodePen](http://codepen.io/SitePoint/pen/YqbdOq)

# 動畫

這個屬性也能夠被用在動畫上。唯一要注意的是初始時和最後結果的座標數量要一致。

```css
@keyframes polygons {
  25% {
    clip-path: polygon(20% 0%, 100% 38%, 70% 90%, 0% 100%);
  }
  50% {
    clip-path: polygon(0 46%, 100% 15%, 55% 74%, 0 100%);
  }
  70% {
    clip-path: polygon(100% 38%, 100% 38%, 66% 100%, 0 53%);
  }
}
```

為了維持動畫的順暢，基本上座標數量最好保持一致，至於形狀的變化您可以透過重疊座標來實現。

# 奇技淫巧

另外我們還可以透過 clip-path 來隱藏元素，不過這個效果類似於 `visibility` 和 `opacity` 該位置空間仍會被佔據。

```css
.hide {
  clip-path: polygon(0px 0px,0px 0px,0px 0px,0px 0px);
}
```

# 關於瀏覽器支援度

這個屬性暫時在 IE, Edge 是不能使用的，Firefox 也還沒完全支援只支援部份語法，在 v47 還得要套過 `layout.css.clip-path-shapes.enabled` 來開啟。Chrome 的話要加上 `-webkit-` prefix。

# 總結

總結來說關於 clip-path 的重點大略為

* 設定的座標點(clip-path)需要能夠封閉，圍成一個形狀(clip-range)，這個形狀就是顯示的區域。
* 周圍的元素仍需要靠 `shape-outside` 來修正。
* geometry-box 用來設定座標軸範圍，用在 HTML 和 SVG 參數是不同的。
* 動畫的部分座標點數量需維持一致。
* 目前大多數的瀏覽器支援度還不夠。
* [clippy](http://bennettfeely.com/clippy/) 和 [座標產生器](http://cssplant.com/clip-path-generator)可以協助我們取得座標。

# 原文參考

* [Introducing the CSS clip-path Property](https://www.sitepoint.com/introducing-css-clip-path-property/)

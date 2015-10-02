---
layout: post
title: 'viewport 與 media query'
date: 2013-08-28 14:11:00
categories: CSS
---

# 簡易觀念

整個 `Responsive Web Design` 最主要的就是靠 css 中的 media query 搭配 `<meta name="viewport" content="width=device-width, initial-scale=1.0">` 這兩個部分來達成。

簡言之就是透過

{% highlight css %}
@media screen and (max-width:420px){  // css here }
{% endhighlight %}  

來針對各種 size 的 width 做對應的設計，而裝置瀏覽器的解析度又可以使用 meta 來設定，就可以使用百分比來處理設計。
原理的概念大致就如上敘述。

隨著智慧型手機上網越來越多的情況下 apple 率先在 safari 定義了一個 meta `viewport` 用來設定瀏覽器的解析度。
預設來說，如果都不設定的情況 safari 會用 width = `980px` 去解析網頁，Android 2.x 是 `800px` ，Android 3.0+ 為 `980px`。

# 注意

螢幕解析度不一定等於 `device-width`，Nexus One 是螢幕解析度(800)不等於device-width(320)的典型例子，iPhone4也是 – 螢幕解析度(960)，但 `device-width` 傳回值為(320)。

# 原理

這邊讓我們先回到影像處理的基本單位 pixel，一張 `800*600` pixel 的數位影像 可以想成是 `800*600` 個"單色格子"組出這張圖片，這就是單純圖片像素的定義。
 
接著螢幕的呈像：例如設定 `800*600` 解析度，表示在你的螢幕上總共要顯示只要顯示這麼多格子，所以如果你螢幕很大。但是解析度設的很小（`640*480`） 就會看到影像變大了，因為一個"物理格子"變大了。換句話說 長跟寬各由 1024 及 768 個像素組合而成的長方形，面積共786432像素（長x寬）的顯像螢幕；此時，若有一個1024x768的圖檔，則它剛好可以填滿整個螢幕。

# 關於 viewport

{% highlight html %}
				<meta name="viewport" content="initial-scale=1.0, width:device-width" />
{% endhighlight %}

`viewport` 的作用是告訴瀏覽器，讀取這個 html 時可視區域有多大，於是瀏覽器就會根據 viewport 的設定去調整。
其中最重要的是 `width＝device-width` 等於是告訴瀏覽器，`寬有幾個 pixel ` 他會覆寫本來預設是 980px 的預設值  所以會發現網站的圖片變大了。
例如：本來設計的手機版網頁寬是 590px 就可以在 viewport 設定 width=590。如此一來整個網頁就會完整呈現，但視覺上會變小。
延伸自適應網站設計的用法則是讓 width=device-width，再透過 CSS 去設計各種手機尺寸的版型 如 320 * 480。

# 屬性和值

* width:[數字] 或 device-width
* height:[數字] 或 device-height
* initial-scale:最小0.25，最大5
* minimum-scale:最小0.25，最大5
* maximum-scale:最小0.25，最大5
* user-scalable:1 或 0 (yes 或 no)

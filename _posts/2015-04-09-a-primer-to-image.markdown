---
layout: post
title: 'CSS 背景圖片排版'
date: 2015-04-09 16:30:00
categories: CSS
---

使用 CSS 套用背景圖片到元素中可能是您前端設計過程中最常用到的功能。
`background` 有非常豐富的屬性，讓我們可以針對不同的需求做設定。

同時一個元素可以具有多張背景圖片。如果您想要套用多張圖片您可以直接在 `background-image` 屬性值設定，不同圖片路徑之間只要用 `,` 分開即可。`background-position` 屬性則是用來指定背景圖片的位置，而這個屬性非常值得深入研究，不同的設定會造成不同的結果，有些效果對您來說可能是沒看過的。

為了確保本文的程式碼和概念清楚單純，我們將只用一張圖片的範例來驗證瞭解每個屬性，不過這些概念同樣適用於多張圖片的情況下。

一個背景圖片被放在一個我們稱之為 `background position area` 的地方，這個`背景座標區域`功能就如同其名稱一樣，其中有一個座標系統讓我們可以處理背景圖的定位。

在我們進一步開始探討`放置`圖片的概念之前，讓我們先快速的在看過一次 CSS box model 以及看看它如何影響元素中背景圖片的位置。

# CSS Box Model 區域
根據定義在 CSS 中一個元素通常有 3 個區塊，這幾個區塊我們統稱`區塊模型`，如果您常閱讀英文的文件那麼這邊就是 `box` ，這幾個區塊分別是 `border 區塊`, `padding 區塊`, `content 區塊`。首先是 `border 區塊` 這個區塊在元素的最外層包含整個內容加上邊框即 `border` 本身。

`padding 區塊` 則是扣除掉 `border` 後元素內容和內容外圍的空間，沒錯就是您透過 `padding` 屬性設定的間隙。

最後 `content 區塊` 就是把元素扣掉 border 和 padding ，實際內容部分。

![]({{ site.url }}/assets/images/tutorials/css_box_1.png)

圖示元素的區塊模型，圖片來源：參考 Codrops CSS 

如果您已經知道如何使用 CSS 排版的樣式規則那您可能會問那 `margin` 呢？`margin` 在元素的最外圍，包含整個元素，它被用來設定各個元素之間的距離，定義上它不被算在元素`內部`。

好！當我們給元素設定了 `background` - 可以是圖片或顏色。背景預設會從 `padding 區塊` 開始畫，這個行為是可以被改變的，透過使用 `background-origin` 這個屬性就是用來設定背景從哪邊開始，預設是 `padding-box`。

為了準確的把背景圖片放到指定的`區塊`中，這個區塊需要一個座標系統，用來計算座標。接著讓我們更深入的看看座標系統。

>注意：如果 `background-attachment` 屬性設定為 `fixed` 那麼 `background-origin` 就會無效。

{% highlight css %}
background-origin: padding-box|border-box|content-box|initial|inherit;
{% endhighlight %}

# 元素的座標系統
預設情況下，CSS 區塊模型的特性是每一個元素都會透過自己的 `height` 和 `width` 來建立座標系統。這個座標系統被用來設定元素和其他元素之間的位置關係，同時影響內部子元素與自己的位置關係。

所以在 CSS 中一個 HTML 元素有一個座標系統，但如果是 svg 元素則沒有。這是因為 svg 本身並沒有區塊模型的概念。

在 CSS 裏通常一個元素的起始點會在元素的最左上角，而`背景座標區域`也同樣屬於區塊模型概念，並且使用這種觀念來擺放圖片，就是從圖片的左上角的點開始。

OK! 因為預設的座標區塊是 `padding 區塊`，因此我們的背景就會從 padding 區塊的最左上角開始。

意味著當您套用一張背景圖，瀏覽器會先從 padding 最左上角的點開始把圖放進去，然後重複擺放該圖。

例如：您套用了一張背景到元素中且設定了`no-repeat`，這樣圖片就只會顯示一次，如果您沒設定就會一直重複。

後面我們會繼續討論如何使用 `background-position` 屬性來調整預設最左上角這個行為。

不過在討論 `background-position` 之前我們先來驗證 `background-origin` 

# 使用 background-origin 調整背景擺放的區域與座標系統

`background-origin` 屬性主要是用來改變座標系統的原點藉此調整背景排版的位置，我們有三種值可以用 `padding-box`, `content-box`, `border-box` 直接來看看[範例](http://codepen.io/SaraSoueidan/pen/61c15fc3845d5b5879735b75007899b3/)，要小心的是 border-box 其實是有變的不過呢如果您的 `border` 有設定顏色，那麼會把圖片蓋掉，您可以試著把 border 的 color 弄成半透明驗證。

接著我們就可以使用 `background-position` 指定背景的位置，我們剩下所有的範例都會採用 `padding-box` 的設定。

# 使用 `background-position` 來設定圖片位置
我們已經看過上一節示範預設的圖片位置的起始點和排版的區域是如何運作的。也恩賜預設的 `position` 的值是 `0% 0%`。

預設的位置值是採用百分比當作單位。您可以直接設定不同的百分比或者根據 position area 範圍和距離來設定`長度`

![]({{ site.url}}/assets/tutorials/position_6.png)

上面圖表示的是設定和邊緣的距離，除了設定百分比和絕對長度之外還有 `top`, `right`, `left`, `bottom`, 和 `center` 五個關鍵字能用。就是區分成關鍵字和長度(px)和百分比這三大類。

一個位置的設定方式可以是一個偏移量值(關鍵字,百分比,長度), 兩個偏移量(可以是上面三種值混搭，不過要注意關鍵字的部分前面是橫向 x 軸的所以是 [left | center | right])，這邊要提醒您的是說 `background-position: top 25%;` 是不正確的，要 `background-position: right 25%;`，以及四個偏移量，四個偏移量的話就是關鍵字搭配長度。
如果兩個值的部分都是關鍵字那順序可以不照語法指定，意思是您可以先放垂直 y 軸的屬性再放 x i.e `background: bottom left;`
下面是語法和一些範例

{% highlight css %}
// 語法
background-position: 30% 15%, 40% 80%, 10px 10px /* 多張影像用逗號分開*/
background-position: length length
background-position: percentage percentage
background-position: percentage
background-position: bottom length right length
background-position: left top

// 範例
background-position: top left;
background-position: 50px 30%;
background-position: right: 10px bottom 100px;
background-position: center center
background-position: 10px 20px;
background-position: 5em 2em;
background-position: 75% 50%;
{% endhighlight %}

如果您提供一個值的時候那麼第二個值(偏移量)

{% highlight css %}
background-position: 10% 50%; /* 10% 是從容器左邊 10% 和圖片從左邊的10%重疊處 */
background-position: top; /* 等於 `top center` */
background-position: 50px; /* 等於 `50px center` */
{% endhighlight %}

[理解百分比的運作](https://css-tricks.com/i-like-how-percentage-background-position-works/)

設定上您可以混合任何合法的參數值，例如混搭百分比和長度或者關鍵字，不過要注意的是關鍵字的部分，除非全部都用關鍵字否則順序很重要，注意第一個參數對應水平的偏移量，第二個對應垂直的。

實際上關鍵字是百分比的縮寫，例如 `top` 等於 `0%`，`bottom` 等於 `100%` 以此類推。

好！接著讓我們開始看看每一種可能的值是如何運作的，這應該是這篇最重要部分，不同的參數類型會產生不同的效果。

# 絕對位置/長度如何運作
當我們指定一個絕對位置的屬性單位(通常是 px)時，您執行的事讓背景圖片根據左上角原點來偏移，換句話說圖片就是根據左上角原點搭配您設定的值來偏移。

看下面的圖片比較容易理解

![]({{ site.url}}/assets/tutorials/position_7.png)

當然絕對位置的值可以是負數，負數的話就會往反方向移動

![]({{ site.url}}/assets/tutorials/position_8.png)

# 百分比是如何運作
不同於絕對位置根據原點偏移的概念，百分比會對齊圖片的 X% 的點和容器 X% 的位置。
舉例來說 `0% 0%` 會對其圖片的 `0% 0%` 位置和元素背景區域的 `0% 0%` 位置。 一樣我們透過下面圖片來理解

![]({{ site.url}}/assets/tutorials/position_9.png)

而百分比一樣也可以設定負值。

# 搭配使用關鍵字指定邊緣
在上面的範例我們都是從左邊和上面來計算偏移，這是預設的行為。
不過我們可以結合關鍵字來指定邊緣
例如：

{% highlight css %}
background-position: top 1em right 3em;
{% endhighlight %}

注意關鍵字要在前面，另外如果指給定三個值 `background-position: top 25px right ;` 最後一個會變成 `0` 而設定了無效的值那麼就會使用預設值 `0% 0%` 例如 `background-position: top 25%` 

# 尺寸，重複，裁切等等
記住，您可以使用多張圖片，如果要對多張圖片設定位置只要使用逗號分隔每一組設定即可。其他如果要控制大小則用 background-size，總而言之有 9 個關於如何控制背景圖的屬性可以使用，一旦您能掌握 position 和 origin 剩下的觀念就相對單純許多。

最後補上一個比較新的屬性的教學 [background-blend-mode](http://sarasoueidan.com/blog/compositing-and-blending-in-css/)


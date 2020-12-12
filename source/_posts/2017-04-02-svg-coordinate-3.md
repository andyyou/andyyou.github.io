---
title: '[譯] 理解 SVG 座標系統與 Transformation - 3 建立 viewpor'
tags:
  - svg javascript
categories: Program
date: 2017-04-02 17:15:34
---

在 SVG 繪製的任何一個時間點，我們都可以透過內嵌 `<svg>` 或者使用像是 `<symbol>` 這類的元素來建立新的 viewport 和用戶座標。
本篇文章我們將要探討該怎麼作？以及如何利用這種方式協助我們控制 SVG 元素使其更具彈性。

本篇是探討 SVG 座標系與變形系列文章的第三篇也是最後一篇，在第一篇我們討論了 SVG 座標系的基本觀念即 `viewport` `viewBox` `preserveAspectRatio` 這三個東西。第二篇主要討論了關於變形的用法和座標系之間關係的觀念。系列文章的連結條例於下：

* {% post_link svg-coordinate-1 [譯] 理解 SVG 座標系統與 Transformation - 1 viewport、viewBox、和 preserveAspectRatio %}
* {% post_link svg-coordinate-2 [譯] 理解 SVG 座標系統與 Transformation - 2 transform 屬性 %}
* {% post_link svg-coordinate-3 [譯] 理解 SVG 座標系統與 Transformation - 3 建立 viewport %}

在這篇文章中，我假設您至少已經讀完第一篇文章，並且能夠掌握 `SVG viewport` 的觀念和 `viewBox` `preserveAspectRatio` 兩個屬性。本篇文章不需要具備第二篇討論的觀念，您可以略過無妨。

<!--more-->

# 嵌入 `<svg>` 

在第一篇文章中我們聊到了關於 `<svg>` 可以為我們的 SVG canvas 即`繪圖的那一層`建立一個 `viewport` 的觀念，就像視窗與網頁內容之間的關係。而這邊要說的就是在 SVG 繪製的任一時間點，其中`繪製`的意義講的更具體一點就是我們加上的 SVG 元素的任意時間點，我們還可以在建立新的 viewport。
作法就是在 `<svg>` 內部加上另一個 `<svg>` 就可以建立新的 viewport。在我們建立新的 viewport 同時我們也建立了一個新的 viewport 座標系和新的用戶座標系。

舉例來說，假設我們的 `<svg>` 如下

```html
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
  <!-- 圖案 -->
  <svg>
    <!-- 其他在嵌入 svg 內部的圖案 -->
  </svg>
</svg>
```

第一件要注意的是內部的 `<svg>` 元素不需要設定命名空間 `xmlns`，因為它會和外層的 `<svg>` 使用同樣的命名空間。當然如果我們使用的是 `HTML 5` 那麽外層的 `<svg>` 也不需要加上命名空間。

> XML 命名空間用來解決 tag 名稱衝突的問題，例如在 HTML 5 以前我們使用 XHTML 應加上 `<html xmlns="http://www.w3.org/1999/xhtml">`。

我們可以用 SVG 元素來將一些元素圈在一個群組，然後集體操作它們。我們知道如果只是要群組的功能我們可以使用 `<g>` ，例如要移動元素，我們可以在 `<g>` 套用 `transform` 效果來達成位移效果，底下所有的元素都會受到影響。這邊要使用 `<svg>` 肯定是因為它具備其他功能。
舉例來說直接設定 `x` 和 `y` 座標在大部分情況下比起使用 transform 要方便多了。還有，`<svg>` 可以設定寬高，而 `<g>` 不行。
我們並沒有要讓 `<svg>` 取代 `<g>` 的意思，兩者在用途上有些差異，我們並不是都需要 `<svg>` 的功能，因為它會建立另外的 viewport 座標和用戶座標，有些時候這並不是我們要的。

透過在 `<svg>` 上設定寬高（這邊指的是內嵌的 `<svg>`），我們可以限制內容呈現的邊界，如同第一篇說的我們使用 SVG 的 `width` 和 `height` 會定義出視窗的範圍，超出範圍的內容都會被裁切掉。另外，這邊要在加上兩個屬性 `x` 和 `y`。

如果不設定 `x` 和 `y` 預設是 `0`，不設定 `width`，`height` 的話預設是最外層 SVG 的 `100%`。看看這個[範例](https://jsfiddle.net/u512m6cy/)稍微體會一下上面提到的裁切效果。

此外，當我們改變這個 `<svg>` 的用戶座標系時，該 `<svg>` 內的內容都會受到影響。

`<svg>` 內的元素如果使用百分比值會根據父層的 `<svg>` 來計算，而不是最外層的 `<svg>`。在內層 `<svg>` 元素上如果使用百分比值則是依據外層的 `<svg>` 來計算。下面範例的結果裡面的 `<svg>` 會是 400 單位，而內部 `<svg>` 裡的方形會是 200 單位。

```
<svg width="800" height="600">
  <svg width="50%">
    <rect width="50%"></rect>
  </svg>
</svg>
```

假定這個範例被放在一個 HTML 文件中，例如最外層的 `<svg>` 被設為 100%，讓它隨著瀏覽器視窗自動縮放。於是裡面的 `<svg>` 就會動態的跟著外層的寬高變動縮放。

> 最外層的 `<svg>` 元素可以使用 CSS 設定背景色和邊框和一般 HTML 元素很接近，但內部的 `<svg>` 則無法。[範例](https://jsfiddle.net/u512m6cy/2/)。另外內嵌 SVG 使用的單位會根據外層設定的結果，這點需要特別注意。修改這個[範例](https://jsfiddle.net/mwmqrkem/1/)內部 `<svg>` 的 width，height，viewBox 觀察一下。

善用內嵌 SVG 可以增加操作 SVG 元素的靈活性。是什麼靈活性呢？假設我們在最外層 `<svg>` 的寬設成 100%，讓我們的 SVG 可以隨著容器（或瀏覽器視窗）縮放，到這一步就可以像網頁那樣使 SVG 也具備自適應的效果。如果我們有其他需求我們可以接著透過 `viewBox` 和 `preserveAspectRatio` 讓內容依照我們想要的方式在自適應的 viewport （最外層的 `<svg>`）中呈現。原作者在 [CSSConf 中的 slide](https://docs.google.com/presentation/d/1Iuvf3saPCJepVJBDNNDSmSsA0_rwtRYehSmmSSLYFVQ/pub?start=false&loop=false&delayms=3000#slide=id.p)有介紹過關於 Responsive SVG 的作法。您可以在[查閱](https://docs.google.com/presentation/d/1Iuvf3saPCJepVJBDNNDSmSsA0_rwtRYehSmmSSLYFVQ/pub?start=false&loop=false&delayms=3000#slide=id.g180585666_036)。

然而，當我們像上面那樣在 SVG 實作自適應的效果時，整個畫布裡的元素都會同時產生對應的變化。有時，我們並不想要這樣，我們可能只想要某個元素彈性變動，其他元素固定位置/尺寸。這個時候內嵌 `<svg>` 就能派上用場。

一個 SVG 元素能夠擁有自己的座標系，自己的 `viewBox` 和 `preserveAspectRatio` 屬性，我們可以依照需求定義其位置，尺寸，顯示的範圍。

所以為了讓一個元素保留更多彈性，我們可以將它用 `<svg>` 包起來，如果給 SVG 百分比的寬那麽元素就會依照最外層 `<svg>` 的寬來調整，想要填滿容器的話可以加上 `preserveAspectRatio="none"`。注意，SVG 沒有限制嵌入的階層，這邊為了讓介紹單純一點，文章只使用一層。

為了展嵌入 SVG 如何發揮作用，下面讓我們來看個範例

# 範例

假設我們有下面一張 SVG

![](https://sarasoueidan.com/images/svg-nesting-example-1.png)

這張自適應的 SVG 當我們改變視窗的大小，整張 SVG 就會根據需要產生改變。下面的截圖顯示了當頁面變窄時的情形，圖片變小了。注意到 SVG 的內容如何變形，它仍然維持原來的座標與比例等等，只是座標系依據 viewport 使單位產生變化。

！[](https://sarasoueidan.com/images/svg-nesting-example-1-resized.png)

使用內嵌 SVG 我們可以改變這點，我們可以替 SVG 中的元素設定在 viewport 中的位置，也就是當最外層 SVG viewport 尺寸改變時，每個元素各自有自己對應變形的方式。

> 注意在這邊您需要熟悉 SVG viewport，`viewBox`，`preserveAspectRatio` 的使用方式。

下面我們要建立的效果是：當視窗改變尺寸時，蛋的上半部會往上移動，後續躲在`後層`的小雞就會出現。如下圖

![](https://sarasoueidan.com/images/svg-nesting-example-1-new.png)

為了完成這個效果，蛋上半部的部分必須要使用 `<svg>` 來和其他元素分開，我們給這個 `<svg>` 設定 ID 為 `upper-shell`。

然後，我們要讓新的 `svg#upper-shell` 和最外層的 `<svg>` 有一樣的寬高，這我們可以透過在 `<svg>` 上設定 `width=100%` `height=100%` 或者全部都不設定保持預設值 100% 都能完成。

> 假如內嵌的 svg 沒用設定寬高，其會自動展延填滿外層的 svg 即寬高為 100%

然後最後，為了讓上半部蛋殼升起，我們將要使用 `preserveAspectRatio` 來使 viewBox 移動到 viewport 的上方 - `xMinYMin`。

```html
<svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
  <!-- ... -->
  <svg viewBox="0 0 315 385" preserveAspectRatio="xMidYMid meet">
    <!-- the chicken illustration -->
    <g id="chicken">
      <!-- ... -->
    </g>
    <!-- path forming the lower shell -->
    <path id="lower-shell" fill="url(#gradient)" stroke="#000000" stroke-width="1.5003" d="..."/>
  </svg>

  <svg id="upper-shell" viewBox="0 0 315 385" preserveAspectRatio="xMidYMin meet">
    <!-- path forming the upper shell -->
    <path id="the-upper-shell" fill="url(#gradient)" stroke="#000000" stroke-width="1.5003" d="..."/>
  </svg>
</svg>
```

> 這裡為了示意觀念，把上面程式嗎一些跟圖有關的部分移除了

在這，注意到 `svg#upper-shell` 的 `viewBox` 值和最外層的 svg 是一樣的（上面程式已經被我們移除）。這麼做的原因是在視窗較大時我們希望圖案跟原來一樣。

所以，事情是這樣的；一開始我們的 SVG 是一顆有裂痕的蛋，圖層下方還有一隻小雞被蛋殼蓋住。然後我們利用內嵌 svg 的方式 - 建立另一個圖層把蛋殼的上半部移到這一層。這個內嵌的 svg 和外面其他的 svg 寬高和 viewBox 設定都一樣（蛋殼下半部 + 小雞）。最後上半部蛋殼 svg 的 viewBox 設定往上對齊 `xMin` 軸。當最外層的 svg 縮小時， viewport 隨著 `width` `height` 產生變化，此時 viewBox 定義的區塊比例和 viewport 不同了，上半部蛋殼就會往上移動。我們來看看圖片。

> 譯者註：原文 `outer svg` 並非最外層的 svg 在對照範例時可能會造成誤會，如果 viewBox 的比例和最外層一樣的話，將不會有效果。因為外層的用戶座標將會和內嵌的 svg 一樣，比例一樣的狀況下 preserveAspectRatio 不會有效果。

！[](https://sarasoueidan.com/images/svg-nesting-example-1-layered.png)

一但視窗尺寸縮小，SVG 變得瘦長，上半部蛋殼 svg 的 viewBox 就會因為 `preserveAspectRatio="xMidYMin meet"` 往上對齊

![](https://sarasoueidan.com/images/svg-nesting-example-1-viewbox.png)

> 灰色的部分顯示出內嵌 svg 的 viewport。淺紫色的部分顯示出上半蛋殼的內嵌 svg viewBox，因為 `preserveAspectRatio="xMidYMin"` 所以對齊上方。

點擊連結查看實際的[範例](https://sarasoueidan.com/images/svg-nesting-chick.svg)

內嵌 svg 或者我們說`增加 svg 圖層`可以讓我們設定不同的對齊裁切方式。換句話說等於是多了一層 viewport，多一次機會設定 preserveAspectRatio。如此一來我們就可以設計變化自適應效果。

如果我們想要整隻小雞顯示，我們可以單獨再把下半部的蛋殼獨立一層 svg，只是這次要換成 `preserveAspectRatio="xMidYMax meet"`

```html
<svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
  <svg id="chick" viewBox="0 0 315 385" preserveAspectRatio="xMidYMid meet">
    <!-- the chicken illustration -->
    <g id="chick">
      <!-- ... -->
    </g>
  </svg>

  <svg id="upper-shell" viewBox="0 0 315 385" preserveAspectRatio="xMidYMid meet">
    <!-- path forming the upper shell -->
    <path id="the-upper-shell" fill="url(#gradient)" stroke="#000000" stroke-width="1.5003" d="..."/>
  </svg>

  <svg id="lower-shell" viewBox="0 0 315 385" preserveAspectRatio="xMidYMax meet">
    <!-- path forming the lower shell -->
    <path id="the-lower-shell" fill="url(#gradient)" stroke="#000000" stroke-width="1.5003" d="..."/>
  </svg>
</svg>
```

每個內嵌的 svg viewport 都跟最外層的 svg 一樣，意味著 `width` 和 `height` 都是 `100%`。所以基本上我們得到了 3 個 viewport 座標系的副本。
每一層各自負責內部的元素。3 個 viewBox 定義一樣，只有 `preserveAspectRatio` 不同。

！[](https://sarasoueidan.com/images/svg-nesting-example-1-2.png)

> 我們可以把之前那個線上的範例，使用 inspect 把 viewBox 都移除觀察原始的樣子，搭配第一篇學習到的觀念驗證一下。在這邊呢，3 個 svg 寬高都是 100% 也就是 viewport 跟最外層一致，viewBox 則是剛剛好圈出圖案的部分，最後我們使用 preserveAspectRatio 來調整 viewBox。

當然，在這個範例效果是讓小雞躲在蛋殼後面，隨著視窗縮小讓小雞出現。您也可以設計一些不一樣的效果。例如我們先在小一點的視窗下建立圖案，當視窗變大時才讓某些圖案顯示出來。

我們可以發揮更多的創意，根據不同的視窗大小顯示/隱藏元素，加上 `media query` 把新元素放到特定位置等等。唯一的限制就是您的想像力。

同時也注意到嵌入的 svg 寬高並不一定要跟外層的 svg 一樣。您完全可以自己設定寬高來決定`顯示的範圍`一切取決於我們想要的效果。

# 使用嵌入 svg 使元素自動調整尺寸（Fluid)

除了在固定比例的情況下調整位置外，我們也能夠使用嵌入 svg 的方式讓特定元素自動調整尺寸或稱為流動式的元素。要完成這個效果我們只要內嵌的 svg 不要保持元素的比例即可。

舉例來說，如果我們只想要 SVG 中的某個元素可以自動調整尺寸，我們可以使用 svg 將它包起來，然後在 svg 設定 `preserveAspectRatio="none"`，如此一來 svg 內部的元素就會自動填滿容器的寬，其他元素則保持一樣的行為。

```html
<svg>
  <!-- ... -->
  <svg viewBox=".." preserveAspectRatio="none">
    <!-- this content will be fluid -->
  </svg>
  <svg viewBox=".." preserveAspectRatio="..">
    <!-- content positioned somewhere in the viewport -->
  </svg>
  <!-- ... -->
</svg>
```

[Jake Archibald](https://jakearchibald.com/) 使用用內嵌 svg 的方式建立過一個簡單的範例。該範例是一個簡單的操作介面，裡面包含了幾個固定比例的元素分別被放在 svg 的四周，然後中間的部分則會跟著 svg 的寬調整尺寸。[範例](https://jsbin.com/loceqo/1)在此。您可以使用瀏覽器的開發者工具觀察席下這個範例的原始碼，看看裡面 viewBox 和 svg 的設定。

# 其他建立 viewport 的方式

svg 元素並不是唯一可以在 SVG 中建立 viewport 的元素。下面的段落我們將要大略來來看看其他可以建立 viewport 的元素。

# 使用 `<use>` 和 `<symbol>` 建立 viewport

`symbol` 元素可以是用定義圖形樣板的，有點類似在 OOP 中 `class` 的感覺。既然有 `class` 那麼就會有負責實例化的 `new`。在 SVG 中 `use` 就類似 new 的功能。

每當我們使用 `use` 實例化 `symbol` 的時候好會建立一個新的 viewport。

透過 `use` 的 `xlink:href` 指定 `symbol` 就可以將實例化：

```html
<svg>
  <symbol id="my-symbol" viewBox="0 0 300 200">
    <!-- contents of the symbol -->
    <!-- this content is only rendered when `use`d -->
  </symbol>
  <use xlink:href="#my-symbol" x="?" y="?" width="?" height="?">
</svg>
```

上面程式碼的問號表示這幾個屬性可以設定或不設，如果沒設定預設是 `0`。

這裡我們觀察到當使用 `use` 搭配 `symbol` 時，我們用瀏覽器的開發者工具並不會看到 `use` 標籤中有 `symbol` 的內容。原因是因為 `use` 的內容被渲染在 shadow tree，您可以開啟檢視 shadow DOM 的功能就可以看到了。

![](http://i.imgur.com/Ijj0gAS.png)

使用 `symbol` 時，它的內容會被完整複製到 shadow tree 中，唯一的例外是 `symbol` 本身會被 `svg` 替換。這個產生的 `svg` 永遠都會有明確的寬高屬性。
假如 `use` 中有設定 `width` 或 `height` 那麼這些屬性會被轉移到產生的 svg 上。如果沒有設定寬高，那麼 svg 的寬高會是 `100%`。

> 當 `use` 參考引用 `symbol` 時，`symbol` 的內容會完整的複製並在 shadow tree 中生成，symbol 則會換成 svg。

因為麼們在 DOM 中使用 svg，然後這個 svg 實際上是透過 use 被包在另一個 svg 中的，這種情況跟的嵌入 svg 並沒有什麼不同 - 嵌入的 svg 一樣會建立新的 viewport。

不過有一點要注意的地方是 `viewBox` 屬性要設在 `<symbol>` 元素上。 `<symbol>` 可以使用 `viewBox` 和 `preserveAspectRatio` 屬性，但 `width` 和 `height` 是下在 `<use>` 上。更多關於 svg 的元素與用法可以參考 [理解 SVG 的建構，群組，引用功能 - <g>，<use>，<defs>，<symbol>](https://sarasoueidan.com/blog/structuring-grouping-referencing-in-svg/)。

所以現在我們有了新的 viewport ，這個 viewport 的座標和定位可以用 `<use>` 的 `x`，`y`，`width`，`height` 設定。
`viewBox`，`preserveAspectRatio` 則設在 `<symbol>` 上。

[Dirk Weber](http://eleqtriq.com/) 也曾使用 `symbol` 搭配嵌入 svg 的方式寫過一個模仿 CSS border 行為的[範例](http://w3.eleqtriq.com/2014/02/the-4-slice-scaling-technique-for-svg/)

# 引用 `<image>` 建立新的 viewport

`image` 元素的用途是將一個完整的檔案渲染到目前用戶座標系中給定的一個矩形中。一個 `<image>` 可以代表一個圖檔像是 png ， jpeg 或 MIME 類型為 `image/svg+xml` 的檔案。

當 `image` 元素引用的是一個 SVG 檔案時會因為檔案中的 svg 元素而建立一個暫時的 viewport。

```html
<image xlink:href="graphic.svg" x="?" y="?" width="?" height="?" preserveAspectRatio="?" />
```

`<image>` 元素也具有許多屬性可以使用，其中跟本文介紹有關的就是 `x`，`y`，`width`，`height` 和 `preserveAspectRatio`。

一般來說，SVG 檔案都有一個 `<svg>` 根節點；這個元素可能有設定寬高，座標，`viewBox` 或 `preserveAspectRatio`。

當 `image` 參考 SVG 圖檔時根 `<svg>` 上的 `x`，`y`，`width`，`height` 會被忽略。<del>除非我們在 `image` 上使用 `preserveAspectRatio` 搭配 `defer`</del>。

> [SVG 2 preserveAspectRatio 移除支援 defer](https://svgwg.org/svg2-draft/coords.html#PreserveAspectRatioAttribute)

總結來說 SVG 檔案的根 <svg> 元素 x，y，width，height，preserveAspectRatio 會由 `image` 上的屬性來設定。

* [範例](https://jsfiddle.net/tLL48dyx/)

至於 viewBox 一樣維持從檔案的設定來，如果檔案沒有 viewBox 則會參考使用 image 的 width，height。觀察這個[範例](https://jsfiddle.net/tLL48dyx/2/)（`viewBox="0 0 image-width image-height"`）。同時在沒有 `viewBox` 時，preserveAspectRatio 會被忽略。此時如果要對齊位置的效果只能用 image 的 x y。

假如我們有一個 `<image>` 引用 png 或 jpeg 然後 `preserveAspectRatio="xMinYMin meet"`，這時圖片的比例會固定並確保圖片儘可能完整的填滿 viewport。
這個 viewport 是靠 `<image>` 的 `x`，`y`，`width`，`height` 來定義的。
如果 `preserveAspectRatio="none"` 那麽圖片就會變形並完整填滿整個 viewport。

# 使用 `<iframe>` 建立 viewport

使用 `iframe` 引用 SVG 檔案類似於使用 `image`，一樣可以使用 `x`，`y`，`width`，`height` 和 `preserveAspectRatio`。
使用方式可以參考[範例](http://bl.ocks.org/GerHobbelt/2623079)。

# 使用 `<foreignObject>` 建立 viewport

`foreignObject` 元素可以建立新的 viewport 並在裡面渲染元素。`foreignObject` 主要是讓我們可以在 SVG 中加入非 SVG 的內容。通常 foreignObject 的內容會使用不同的命名空間。例如我們可以放入 HTML 到 SVG 裡。

同樣的 `foreignObject` 也可以使用 `x`，`y`，`width`，`height`。關於更詳細的資訊您可以參考下列連結

* [MDN](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/foreignObject)

# 總結

使用我們上面介紹的方式建立新的 viewport 和座標系主要是讓我們可以獨立控制 SVG 中的某個區塊，提供不同的設定。這篇文章的核心是讓我們了解關於嵌入 svg 如何提供更彈性的方式去協助我們創造更有意思的 SVG ，就如同上面提到的自適應 SVG ，自動調整尺寸，模仿 CSS border 等等。我們也可以應用在專案之中，甚至想出更多點子。

到此結束了理解 SVG 座標系統與 Transformation 系列文章，希望您能在這有些收穫。

# 重點

* 內嵌 svg 的 width, height, viewBox 等等的單位會依據外層 svg。


# 參考資源

* [Understanding SVG Coordinate Systems and Transformations (Part 3) — Establishing New Viewports](https://sarasoueidan.com/blog/nesting-svgs/)
* [理解SVG坐标系统和变换： 建立新视窗 - 簡體中文](http://www.w3cplus.com/html5/nesting-svgs.html)
* [SVG 優化工具](http://petercollingridge.appspot.com/svg-editor)
* [SVG 優化工具 - svgo](https://github.com/svg/svgo)


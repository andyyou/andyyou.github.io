---
title: '[譯] 理解 SVG 座標系統與 Transformation - 2 transform 屬性'
tags:
  - svg javascript
categories: Program
date: 2017-04-02 17:15:33
---


SVG 的元素可以執行縮放、移動、傾斜、旋轉等形變效果就像 HTML 元素可以用 CSS transform 一樣。然而當變形的任務牽扯到座標時，勢必也會影響結果。
在這篇文章我們要討論的是 SVG 的 `transform` 屬性與 CSS 屬性，內容涵蓋如何操作 SVG 變形以及座標系轉換過程您應該知道的事。

這是系列文章的第二篇，我們將開始探討 SVG 座標系與變形之間的關係。在第一篇時我們介紹了關於 SVG 座標系的基礎概念，更具體的說就是 viewBox， preserveAspectRatio 屬性和 viewport 的概念。

* {% post_link svg-coordinate-1 [譯] 理解 SVG 座標系統與 Transformation - 1 viewport、viewBox、和 preserveAspectRatio %}
* {% post_link svg-coordinate-2 [譯] 理解 SVG 座標系統與 Transformation - 2 transform 屬性 %}
* {% post_link svg-coordinate-3 [譯] 理解 SVG 座標系統與 Transformation - 3 建立 viewport %}

這裡我假設您已經閱讀並理解第一篇的部分，如果您還沒參透那麼這邊建議您先讀完第一篇。

<!--more-->

# trasfrom 屬性

`transform` 屬性可以設定一或多個關於元素變形的參數。它會將`參數`放到一個 `<transform-list>` 列表並將這些變形效果按照順序套用。各個的變形效果參數使用空白字元或逗號(,)分開。舉例來說要使一個元素變形的程式碼看起來如下：

```
transform="translate(20, 20) rotate(20)"
```

SVG 可以在 transform 中使用的變形包含：旋轉、縮放、位移、傾斜變形。使用方式類似於 CSS transform 但是它們的參數有些差異。

下面我們就來逐一介紹這些參數，更細的說來這些參數屬於一種函數，因為它們各自還需要各自的參數

# Matrix

使用 `matrix()` 您可以套用一到多種的變形效果到元素上，該函式的語法如下

```
martix(<a> <b> <c> <d> <e>)
```

上面是 matrix 的使用格式，它需要 6 個參數是依據矩陣來計算變形結果。對於不擅長數學的人來說，可能也沒辦法使用這個函數。不過您可以參考[W3C 的說明](http://www.w3.org/TR/SVG/coords.html#TransformMatrixDefined)或著[這篇不錯的文章](http://www.oxxostudio.tw/articles/201409/svg-20-transform-matrix.html)。在多數情況下這個函數很少被直接使用，因為牽扯到複雜的數學計算，所以在這邊我們會直接跳過這個函數，有興趣的讀者可以參考上面的連結。

# 位移

要移動 SVG 的元素，我們可以使用 `translate()` 函數。語法如下

```
translate(<tx> [<ty>])
```

關於 `translate()` 函數它需要 1 到 2 個參數，分別設定水平 X 軸和垂直 Y 軸的偏移值。`tx` 為設定 X 軸偏移，`ty` 則是 Y 軸偏移。
`ty` 是選擇性的參數，如果被忽略，其預設值為 0。`tx` 和 `ty` 之間可以用空白字元或 `,` 隔開。不需要設定任何單位，它們會跟著目前用戶座標系的單位。

下面範例會往右移動元素 100 個用戶單位，往下 300 個單位

```
<circle cx="0" cy="0" r="100" transform="translate(100, 300)">
```

[範例](https://jsfiddle.net/91rszv5a/)

# 縮放

我們也可以改變 SVG 元素的大小，這時就要使用 `scale()` 函數

```
scale(<sx> [<sy>])
```

`scale()` 函數需要 1 到 2 個參數，分別設定水平和垂直的縮放的倍率。sx 為 x 軸的倍率用來放大或縮小 x 軸的單位。sy 則是 y 軸的縮放倍率。
`sy` 同樣是選擇性的參數，如果忽略預設會等於 sx 的值。一樣兩個參數可以用 `空白字元` 或 `,` 隔開，也一樣不需要單位。

```
<rect width="150" height="100" transform="scale(2)" x="0" y="0">
```

下面的例子會水平擴展 2 倍的寬，縮小高為原本的一半。

```
<rect width="150" height="100" transform="scale(2 0.5)" x="0" y="0">
```

小數的 0 可以省略，空白字元或逗點可以擇一使用，上面的例子也可以寫成 `scale(2, .5)`。這裡有個重點需要注意當 SVG 元素被縮放時，當前的座標也會被縮放，結果就是元素的位置可能會偏移。不過現在先別擔心我們後續會討論到細節的部分。

# 傾斜

SVG 元素也可以被傾斜/扭曲(Skew)變形。我們可以透過 `skewX` 和 `skewY` 來完成這個效果。

```
skewX(<skew-angle>)

skewY(<skew-angle>)
```

`skewX` 函數會隨著 x 軸傾斜變形，`skewY` 則是跟著 y 軸變形。您可以先試試下面的範例來理解這個效果

[範例](https://jsfiddle.net/91rszv5a/1/)

skew 角度不需要設定單位，預設是`角度`。

注意到傾斜的時候元素可能會被改變定位，在範例中試試 `skewX(0)` 和 `skewX(60)` 可以觀察到結果。一樣我們先別擔心這個問題，待會會說明。

# 旋轉

使用 `rotate()` 可以旋轉 SVG 元素

```
rotate(<rotate-angle> [<cx> <cy>])
```

`rotate()` 函數基於一個中心點和旋轉的度數來產生旋轉效果。跟 CSS 不同的是：這邊不能設定單位，預設就是使用角度，不是徑度(弧度)。

選擇性參數 `cx`，`cy` 可設定旋轉的中心點，同樣`不`設定單位。如果 cx 和 cy 沒有設定時預設的旋轉的中心是目前用戶座標的原點。

設定旋轉中心點的語法就像 CSS 中 `transform: rotate()` 搭配 `transform-origin` 的簡寫。因為預設旋轉的中心點是 svg 用戶座標的左上角，所以跟您預想的旋轉效果可能不同。此時我們可以設定一個新的中心點。如果您知道該元素在 SVG 的座標，您就可以輕易的把元素的中心設為旋轉的中心點。

下面範例示範旋轉一個元素群組 `<g>` 並指定旋轉中心點為用戶座標系中的 `(50, 50)`

```
<g id="parrot" transform="rotate(45 50 50)" x="0" y="0">
  <!-- 鸚鵡圖案 -->
</g>
```

然而如果您想要依據元素中心點來旋轉元素，您可能想要像 `50% 50%` 這樣的用法，不幸的是 svg 的 `rotate()` 沒有這個功能。您需要使用絕對位置， 不過我們可以透過搭配 CSS 的 `transform-origin` 和 `transform` 來完成您想要的效果。本篇後續我們會更詳細的探討這個問題。

# 座標系的轉換

現在我們已經介紹了 SVG 所以的變形函數，我們將深入探討關於套用變形效果至 SVG 元素時️視覺效果呈現的部分。這是控制 SVG 變形中最重要的部分。以及我們稱其為座標系轉換，而不只是元素變形。

在[規範](http://www.w3.org/TR/SVG/coords.html)中，`transform` 屬性被定義為可以建立`新`用戶座標系的兩個屬性之一，`viewBox` 是另一個。但具體來說這個定義到底在說什麼？

> 使用 `transform` 屬性的元素會建立一個`新的用戶座標`即目前使用的座標系。

這個行為很像在 HTML 元素中套用 CSS 變形的屬性 - 即元素的座標系統改變了(影響子元素)，通常在執行一系列變形效果時尤其明顯。儘管很多地方 SVG 變形機制和 HTML 類似，但還是有些不同的地方。

最主要的差異就是座標系。HTML 元素的座標系會建立在元素自己身上。在 SVG 中，元素的座標一開始用的是當前的用戶座標，或稱用戶空間。

當您在 SVG 元素套用 `transform` 屬性時，元素會取得一個用戶座標`副本`來使用。我們可以想成為這個`變形的元素`建立了一個新的圖層，這個新圖層複製了一份自己的座標系(從 viewBox 複製)。接著元素的座標系會根據 transform 屬性的設定轉換，這個結果就會造成元素變形。

為了理解 SVG 變形是如何套用運作，讓我們試著從視覺化的範例來理解。下圖是我們要用來操作的 SVG

![](https://sarasoueidan.com/images/svg-transforms-canvas.png)

鸚鵡和小狗的圖片各自使用 `<g>` 群組起來，我們將試著對它們套用變形的效果

```
<svg width="800" height="600" viewBox="0 0 800 600">
  <g id="parrot">
    <!-- 鸚鵡圖案 -->
  </g>
  <g id="dog">
    <!-- 小狗圖案 -->
  </g>
</svg>
```

灰色的座標時 viewBox 初始化時建立的，為了單純起見我們先不改變初始化的座標，即 viewBox 和 viewport 相同尺寸。

> 當我們套用 `transform` 屬性到 SVG 元素時，會取得一個用戶座標副本來使用。

現在我們已經建立了 canvas 並初始化了用戶座標，接著我們要來讓元素變形。一開始讓我們來移動鸚鵡向右 150 個單位，向下 200 個單位。
這隻鸚鵡我們是透過 `path` 和基礎形狀畫出來的並用 `<g>` 包起來讓它成為一個物件。後續我們可以對 g 套用 `transform` 它就會將效果套用到 g 底下所以的形狀，於是鸚鵡就會像是單一物件一樣移動。

```
<svg width="800" height="800" viewBox="0 0 800 600">
  <g id="parrot" transform="translate(150 200)">
    <!-- 鸚鵡圖案 -->
  </g>
  <!-- ... -->
</svg>
```

下圖就是位移的結果，半透明的圖案是鸚鵡一開始的位置。

![](https://sarasoueidan.com/images/svg-transformations-translate.png)

關於 SVG 偏移的變形效果看起來非常簡單直覺，跟 CSS 的用法一樣。不過上面我們提到 `transform` 屬性套用到元素時會建立一個新的座標系。下圖表示建立在圖案上的用戶座標副本，現在鸚鵡的座標被移動了

![](https://sarasoueidan.com/images/svg-transformations-translate-system.png)

值得注意的是新的用戶座標是建立在元素上的，它是初始化時的用戶座標的副本(複製一份用戶座標)，裡面也保留了元素原本的座標。這意味著雖然我們說副本座標建立在元素上，但它並非依據元素的邊界或尺寸來建立，換句話說，元素本身不會限制座標的大小。這就是 SVG 座標系和 HTML 最大的差異。

> 新的用戶座標建立在`形變`的元素上，但不會被限制在元素的邊界或尺寸之下。

如果我們移動右下角的小狗效果會更明顯，假設我們移動小狗向右下各 50 個單位。下圖為套用的效果。我們注意到新的座標並不是依據小狗圖案本身的邊界，位移也不是從小狗圖案的左上角開始。從下圖我們理解到位移是先複製一份畫布的座標，新增了一個圖層然後依據副本的座標來變形。

![](https://sarasoueidan.com/images/svg-transformations-translate-dog.png)

現在讓我們來試試其他的效果，讓我們來將鸚鵡放大 2 倍

```
<svg width="800" height="800" viewBox="0 0 800 600">
  <g id="parrot" transform="scale(2)">
    <!-- 鸚鵡圖案 -->
  </g>
  <!-- ... -->
</svg>
```
 
 SVG 放大的效果和 HTML 放大的效果有些不同，我們注意到放大的 SVG 元素的位置和 viewport 1:1 時不一樣了。下圖顯示出不止圖案被變大連位置座標也被放大了

 ![](https://sarasoueidan.com/images/svg-transformations-scale.png)

我們稍早提到這是因為座標系變形，座標系的單位也被放大了。也就是原本的 x, y 在和 viewport 對應轉換後也變大 2 倍了。
而這隻鸚鵡會被畫在新的座標系上，所以在這個範例中的效果類似於 `viewBox="0 0 400 300"` 像是對畫布 zoom in。

如果我們來看看視覺化的結果，就會像下圖

![](https://sarasoueidan.com/images/svg-transformations-scale-system.png)

這隻鸚鵡的新座標系被放大了，注意到鸚鵡在這個新座標中位置並沒有改變，座標不變只是因為單位變大了所以渲染到 viewport 時看起來像是位移了。

讓我們試試使用不同的比例縮放鸚鵡的寬高，假如我們使用 `transform="scale(2 0.5)"` 那等於讓寬變為 2 倍，高只剩一半，這個效果類似於 `viewBox="0 0 400 1200"`

![](https://sarasoueidan.com/images/svg-transformations-scale-2.png)

注意到鸚鵡在的紅色的新座標中位置和原來灰色座標 - x 和 y 是一樣的。

也因此在 SVG 傾斜扭曲元素也會造成元素被`移動`的效果，因為目前的用戶座標被扭曲了。例如我們套用 `skewX` 效果到小狗的 x 軸上

```
<svg width="800" height="600" viewBox="0 0 800 600">
  <!-- ... -->
  <g id="dog" transform="skewX(25)">
    <!-- 小狗圖案 -->
  </g>
</svg>
```

下圖是套用傾斜效果到小狗圖案的結果

![](https://sarasoueidan.com/images/svg-transformations-skew-system.png)

因為座標系被扭曲了所以小狗的位置和原來不同了，但在新座標的位置不變。

接著我們來看看使用 `skewY()` 的效果

![](https://sarasoueidan.com/images/svg-transformations-skew-system-2.png)

最後我們來試著旋轉鸚鵡。預設的旋轉中心點是目前用戶座標的左上角。新的用戶座標會建立在旋轉的元素上。下面的範例我們要旋轉鸚鵡 45 度。正值會順時針旋轉。

```
<svg width="800" height="600" viewBox="0 0 800 600">
  <g id="parrot" transform="rotate(45)">
    <!-- 鸚鵡圖案 -->
  </g>
  <!-- ... -->
</svg>
```
![](https://sarasoueidan.com/images/svg-transformations-rotate.png)

您可能會想要讓元素依照圖案的中心點旋轉，而不是座標系的原點。使用 `rotate()` 時我們可以設定旋轉的中心點。假設我們想要在這個範例根據鸚鵡的中心來旋轉鸚鵡。
依照鸚鵡的寬高和位置，我們可以知道它的中心點大約在 `(150, 170)`

```
<svg width="800" height="600" viewBox="0 0 800 600">
  <g id="parrot" transform="rotate(45 150 170)">
    <!-- 鸚鵡圖案 -->
  </g>
  <!-- ... -->
</svg>
```

![](https://sarasoueidan.com/images/svg-transformations-rotate-center.png)

我們說所謂的變形效果是`套用在座標系`上，因此元素最終也會受到影響而變形。那麼究竟要如何改變座標系的原點 `(0, 0)` 呢？

當我們改變中心點或旋轉時，座標系會被移動，然後旋轉設定的角度，最後在依據設定的中心點位移回去。舉例來說：

```
<g id="parrot" transform="rotate(45 150 170)">
```

瀏覽器會執行下面一系列的操作

```
<g id="parrot" transform="translate(150 170) rotate(45) translate(-150 -170)">
```

目前的用戶座標(副本)會被位移到我們設定的中心點，然後旋轉設定的角度，最後透過位移負值

![](https://sarasoueidan.com/images/svg-transformations-rotate-center-system.png)

在我們繼續探討關於巢狀與連續套用變形效果之前，我想請大家注意關於所謂的目前的用戶座標，它是各自被建立在變形元素上的，每個變形元素的座標是各自獨立的，不會��影響。
下圖顯示了小狗和鸚鵡的座標系。

![](https://sarasoueidan.com/images/svg-transformations-multiple.png)

另外要注意的是每一個元素的用戶座標仍然存在於 canvas (svg viewBox 屬性) 所建立的`主`用戶座標系中。任何 viewBox 產生的效果會影響 canvas 中的元素，無論元素本身是否有自己的座標系。

舉例來說，下圖我們把 viewBox 屬性從 `viewBox="0 0 800 600"` 換成 `viewBox="0 0 600 450"`。於是整個 canvas 不只會被放大，還會保留每個元素各自套用的變形效果。

![](https://sarasoueidan.com/images/svg-transformations-multiple-2.png)

# 巢狀(Nested)與鏈接組合變形

很多時候我們會需要套用多個效果到元素上。這種套用多個變形效果我們可以當成是在`組合`變形效果。原文為 `chaining transformations` 也可稱作鏈接變形。
當我們要組合效果的時候，最重要的是要清楚，其行為會跟 HTML 元素變形一樣，每一個效果是會堆疊的，意味著每一個效果會套用在上一個效果處理後的座標系上。

例如：我們要旋轉一個元素後位移，此時位移會根據新的座標系並不是一開始沒有旋轉的那個。

下面的範例我們先套用了旋轉的效果然後向右位移 200 個單位 `transform="rotate(45 150 170) translate(200)"`

![](https://sarasoueidan.com/images/svg-transformations-rotate-translate.png)

這種組合效果的座標行為一旦牽扯到縮放可能又更不直覺，讓我們直接看看另一個[範例](https://jsfiddle.net/paLtbLvc/)

```html
<svg width="600" height="300">
  <rect id="a" x="20" y="20" width="10" height="100" fill="red"
    transform="translate(10 10)"
  ></rect>
  <rect id="b" x="20" y="20" width="10" height="100" fill="pink"
    transform="translate(10 10) scale(2)"
  ></rect>
  <rect id="a" x="20" y="20" width="10" height="100" fill="blue"
    transform="translate(10 10) scale(2) translate(-10 -10)"
  ></rect>
</svg>
```

根據我們要的效果來鏈接變形效果的函數，堆疊的最後座標(位置)和變形就是我們的結果。永遠記住座標的變化，尤其是那些造成單位發生改變的效果。例如一個座標 `(10, 10)` 經過 `scale(2)` 後，因為其新的用戶座標單位被變大了，座標依然是 `(10, 10)` 只是最後要渲染到 viewport 時因為元素的用戶單位是 viewport 單位的 2 倍才會被畫在 viewport `(20, 20)` 的地方。

另外當我們傾斜元素時，座標系統將不再是 90 度角而是根據`傾斜後的座標系統`來計算。

巢狀變形或稱嵌套變形(Nestd transformations)指的是當一個變形元素的子元素也具有變形效果。這種變形會堆疊父元素和自己的效果。

所以實際上巢狀變形跟組合變形是很類似的，唯一的差異是巢狀變形不是套用一系列的變形效果到元素上，它是直接取得父層的最終效果，接著補上自己的變形效果。

這在處理某 A 元素已和 B 元素有對應關係時尤其實用。什麼意思，假如我們想要實作小狗搖尾巴的動畫，這個尾巴會是小狗元素裡面的子元素，即在小狗的群組裡。

```
<svg width="800" height="800" viewBox="0 0 800 600">
  <!-- ... -->
  <g id="dog" transform="translate(..)">
    <g id="head">
      <!-- 頭 -->
    </g>
    <g id="body" transform="rotate(.. .. ..)">
      <path id="tail" d="..." transform="rotate(..)">
      </path>
      <g id="legs">
      </g>
    </g>
  </g>
</svg>
```

如果我們已經移動了小狗，又將身體旋轉了幾度。我們的搖尾巴動畫就必須繼承已產生的效果。當我們旋轉了尾巴的部分時，我們就繼承了其父層 `#body` 已轉換的座標系。同樣的 `#body` 也會繼承 `#dog` 的效果。結果跟上面提到的組合效果是不是很類似。

# 使用 CSS 屬性使 SVG 變形

由於 CSS 3 變形規範整合了 `SVG Transforms`，`CSS 2D Transforms`，和 `CSS 3D Transforms` 的規範。並且將一些功能引入 SVG 例如 `transform-origin` 和 3D 的部分。也就是我們可以用 `transform attribute` (在 SVG 標籤上) 和 `transform property` (CSS 樣式) 這`兩種`方式來讓  SVG 元素變形。但要注意的是兩者的參數設定是不同的。

> 後續我們稱 transfrom attribute 是指 SVG 元素上的屬性，transform property 則是 CSS 的用法。

由於 CSS Transforms 規範中定義 CSS 的 transform 屬性也可以被套用在 SVG 元素上。但 `transform` 屬性的效果函數必須要遵循 CSS 的語法，函數的參數必須要使用逗號 `,` 分開，不可以使用空白字元來區隔參數。但在設定字串中加入空白是沒問題的，只是沒有區隔參數的作用。然後 `rotate()` 函數沒有 `cx` `cy` 的用法，要設定旋轉中心點則使用 `transform-origin` 屬性，預設是 `50% 50% 0` 即該元素的中心點。另外 CSS 屬性是允許使用單位的，以角度來說我們也可以使用弧度(徑度)，座標部分可以使用 `px`，`em` 等等。

兩者用法的差異是這邊需要注意的。

下面的範例是我們使用 CSS 來旋轉 SVG 元素

```css
#parrot {
  transform-origin: 50% 50%;
  transform: rotate(45deg);
}
```

同時 SVG 元素也可以透過 CSS 3D Transforms 達成 3D 的效果。不過，仍然要注意用戶座標的影響和 HTML 座標系之間是不同的，試著使用下面 3D 旋轉的程式碼我們可以看出它並不是以元素的中心點旋轉的。

```
#svg-el {
  transform: perspective(800px) rotate3d(0, 0, 1, 45deg);
}
```

利用 CSS 變形效果套用在 SVG 元素上大致上跟套在 HTML 元素上一樣，只要注意座標系的觀念即可，因此在這篇文章我們將跳過 CSS 的部分。
在撰寫本篇文章的同時還是有些瀏覽器支援的功能並不完整，因此我強烈建議您實作看看到底瀏覽器支援的程度到哪來決定是否要在您的專案中使用該屬性。

# 動畫

SVG 變形也可以拿來作動畫，就像 CSS 變形效果一樣。我們使用 CSS `transform` 屬性來使 SVG 產生變形，當然也可以使用 CSS 動畫和轉場效果套在 SVG 元素上。

SVG 的 `transform attribute` 可以透過 SVG `<animateTransfrom>` 元素來產生動畫。
`<animateTransform>` 元素是三種可以讓 SVG 產生動畫的元素之一。詳細 `<animateTransform>` 的用法已超出本文範疇。我們概略看看範例即可。

[範例](https://jsfiddle.net/paLtbLvc/4/)


# 進階 - 底層的原理 - `矩陣`

對於那些想更加深入理解變形的人，這一小段落為您起個頭。詳細的解釋與算法您可以閱讀[Transform Matrix](http://www.oxxostudio.tw/articles/201409/svg-20-transform-matrix.html)這篇文章。這裡只是單純的歸納出 - 最終畫在 viewport 上的圖形都會依據 transform 所產生的 CTM (Current Transform Matrix) 來畫出變形的結果。每每套用一個效果等於是 `當前矩陣` * `變形矩陣`。

不管是 `viewBox` 造成的效果或是 `transform` 就是不斷的相乘得到最終的 CTM。您可以使用 `document.querySelector('el').getCTM()` 來觀察。

# 最後

學習 SVG 一開始可能會因為對座標系的轉換不是很清楚而感到非常困惑，尤其是您具備 CSS 的知識，自然會希望 SVG 元素和 HTML 元素的變形行為一致。
然而一但我們掌握了 SVG 轉換的機制我們就能夠輕易的控制 SVG。

最後一篇我們將更深入的探討巢狀 SVG 與如何建立新的 viewport 與 viewbox 的部分。

# 資源

* [SVG Matrix](http://www.oxxostudio.tw/articles/201409/svg-20-transform-matrix.html)
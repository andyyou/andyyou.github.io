---
title: '[譯] 理解 SVG 座標系統與 Transformations - 1'
tags:
  - svg javascript
categories: Program
date: 2017-03-17 14:20:44
---


SVG 元素並不像其他 HTML 元素受到 CSS 盒模型的管轄。乍看之下元素的定位、變形、移動、旋轉等形變的行為並不是那麼直覺好控制。
然而只要我們理解關於 SVG 的座標系與圖案變形(transformation)是如何運作，控制 SVG 就會變得非常容易。

在這篇文章我們要討論 SVG 中最重要的 1 個觀念和 2 個屬性 ，它們控制了 SVG 座標系統：`viewport`、`viewBox`、`preserveAspectRatio`。

我們將探討 SVG 的座標系統和 Transformations ，這系列文章將會包含下列 3 篇

* {% post_link svg-coordinate-1 [譯] 理解 SVG 座標系統與 Transformation - 1 viewport、viewBox、和 preserveAspectRatio %}
* {% post_link svg-coordinate-2 [譯] 理解 SVG 座標系統與 Transformation - 2 transform 屬性 %}
* {% post_link svg-coordinate-3 [譯] 理解 SVG 座標系統與 Transformation - 3 建立 viewport %}


<!--more-->

為了感受以及將要探討的概念，這裡提供了一個可互動的範例，讓我們概略理解一下關於 viewBox 和 preserveAspectRatio 這兩個屬性

[範例](https://sarasoueidan.com/demos/interactive-svg-coordinate-system/index.html)

先不需要花太多時間研究上面這個範例，待我們說明之後您可以使用這個互動的範例去驗證自己是否理解了 SVG 的一些行為。

# SVG canvas

第一個我們要先介紹的是 SVG canvas - 指的是一個空間或區域，SVG 會在這一層畫下圖案、內容的地方(SVG 作畫區域/畫布)。概念上 SVG canvas 這張畫布的 xy 軸都是無限長的。
因此 SVG 可以是任何尺寸。然而螢幕本身並不是無限長，於是 SVG 的渲染輸出是必須對應有限尺寸的 viewport（可視區域）就是該如何計算畫在螢幕上。SVG 的內容只要超過 viewport 的邊界就會被裁切，並且被隱藏起來。到這我們明白了在 SVG 中有一層畫布，圖案具體是跟這一層有相對關係，然後有一層可視區域 viewport 的概念，讓我們接著介紹。

# Viewport

viewport 或稱可視區指的是 SVG 的顯示範圍，我們可以把 viewport 想成是電腦中的視窗，改變這個視窗的尺寸會影響我們看到的內容，通過視窗我們可能看到的是整個內容或者只有局部。更具體的比喻 - SVG 的 viewport 類似於瀏覽器視窗，我們透過視窗看到裡面的內容。網頁內容的寬是可以比視窗寬的，一般情形下網頁內容的高的確會超過視窗的高，但我們一次只能從視窗中看到局部的內容。這就是 viewport 的概念。

透過 svg 標籤上的 `width` 和 `height` 屬性我們可以設定 viewport 的尺寸，同時指的是 svg 元素的尺寸，也是 canvas 的尺寸。

```html
<!-- viewport 會是 800px * 600px -->
<svg width="800" height="600">
  <!-- SVG 內容 即 SVG canvas -->
</svg>
```

> 在 3D 的世界中`世界座標系(World Coordinate Syste)`定義了一個空間的絕對位置，然後我們可以有多個用戶座標系(User coordinate system) - 取決於要從哪個點開始，往那個方向看去的相對座標。我們可以在世界座標系中任意`動態`的定義另一個用戶座標系。這段概略的介紹是要讓我們對於`用戶座標系`這個生硬的詞有個簡單的概念。

在 SVG 中，值不一定要具有單位。沒有單位的值，就會在`用戶空間 (User space)/用戶座標系`中使用`用戶單位`，如果值使用了`用戶單位`，那這個值預設就會使用 `px` 為其單位。
這表示上面的範例將會渲染一個 800px * 600px 的 viewport。當然，您也可以使用其他單位。SVG 支援的單位包含 em、ex、px、pt、pc、cm、mm、in 和百分比。

回到上面寬高的設定，這意味著一但最外層 SVG 的寬高被設定了，瀏覽器就會初始化 `viewport 座標系統` 和 `用戶座標系統`。

# 初始化座標系統

初始化的 `viewport 座標系` 建立 viewport 上，這個座標系從 viewport 的最左上角為 (0, 0) 開始計算。 X 軸的右側為正向，Y 軸的下方為正向。我們可以理解成 svg 元素的左上那個點就是 `(0, 0)`。
同時在初始化時 viewport 中的一個單位等同於 `1px`，這個座標系統跟 HTML 和 CSS 盒模型的座標系統很類似。

初始化的 `用戶座標系統` 則建立在 SVG canvas 上，這個座標系一開始會和 viewport 座標系相同。原點都是在 viewport 的左上角。
如果使用 `viewBox` 屬性的話這個`用戶座標系統`就會被調整修改，也就是不再跟 viewport 座標系統一致即實際的單位大小和原點可能不同。我們會在後續討論關於修改的部分。

現在，我們先不設定 viewBox ，讓 SVG canvas 的用戶座標和 viewport 座標相同。

在下面的圖片中，灰色的尺規標的是 viewport 座標 ，青藍色的則是用戶座標（viewBox）。因為它們目前是一樣的所以兩個尺規會重疊。

![](https://sarasoueidan.com/images/initial-coordinate-systems.jpg)

總結上面的說明就是兩個座標系統分別屬於在 viewport 和 SVG canvas。

圖片上的鸚鵡顯示寬為 200，高為 300 在這邊單位是 px，這隻鸚鵡依據初始化的座標被畫在 SVG canvas 層。而 viewport 就像瀏覽器視窗一樣，它就是我們渲染到瀏覽器上最終具體尺寸的依據。

# viewBox

到這邊我們先把它們整理一下 畫布(SVG canvas) 使用的是用戶座標，也是用來畫圖的座標，viewport 的是另一個座標系通常只用來作最後渲染的參考換算。
viewBox 可以調整用戶座標，`所以我偏好直接把 viewBox 直接當作是最終使用的座標系`，畢竟最終呈現就是用它。這個座標系統或者我們說這個空間是可以大或小於 viewport，要完成顯示於 viewport 或局部顯示取決於我們的需求。

`用戶座標系統` 其實就是 viewBox 參數產生的座標系，目前跟 viewport 座標一致。因為我們現在沒有設定，所以 2 個座標預設是一樣的，這就是為什麼目前畫上去的物件看起來像是依據 viewport 座標定義一樣，但實際上依據的是`用戶座標系`。

一般的流程是：我們透過 `<svg>` 的 width, height 定義 viewport 座標，一但 viewport 座標系統初始化，瀏覽器預設會建立一個`用戶座標系統`其所有定義跟 viewport 座標一樣。

後續我們可以使用 viewBox 調整`用戶座標系統`，如果`用戶座標系統`的寬高比和 viewport 一致，就會自動擴展以填滿 viewport 的空間。
不過，如果您的`用戶座標系統`和 viewport 比例不同，我們可能就需要使用 `preserveAspectRatio` 屬性來調整在 viewport 中顯示的方式，我們可以設定其在 viewport 的位置(垂直置中或水平置中等等)。在下一節我們會透過大量的範例來說明。現在我們將固定 viewBox 的比例跟 viewport 一樣，所以 preserveAspectRatio 在此沒有作用。

> 白話小結：在 svg 的世界裡，底下有一張不限寬高的畫布(canvas)，預設用 px 當單位，我們把圖畫在這裡。接著上面有一層視窗層(viewport)，概念像透過瀏覽器視窗看網頁一樣就是那個可視區域。初始化後兩個東西的寬高一樣，通常圖案也是根據此時的座標和單位配置，但後續 canvas 可以使用 viewBox 調整，概念上有點像 Sketch Slice 或 Illustrator 切片定義的那個範圍。在 canvas 上切個範圍，接著它會盡可能的塞滿 viewport，於是 canvas 上圖案的單位產生變化，我們還可以用 preserveAspectRatio 來設定`塞滿`的規則像是對齊等等。

# viewBox 語法

關於 `viewBox` 屬性需要 4 個參數分別為 `<min-x>, <min-y>, width, height`

```html
<svg viewBox="<min-x> <min-y> width height"></svg>
```

`<min-x>` 和 `<min-y>` 的值決定 viewBox 的左上座標，width 和 height 則決定 viewBox 的寬高。注意 viewBox 的寬高並不需要跟 svg 的寬高一致，
並且`負值是不合規範的`。只要寬或高其一設為 0 時則停止渲染該元素。

> 注意 viewport 的寬是可以透過 CSS 修改的，高不行。例如：設定 width: 100% 會使 SVG viewport 填滿 doucment 的寬。接著無論 viewBox 的值是什麼都會對應填滿 viewport 並轉換計算出對應的單位值。

```html
<!-- 在這個例子 viewBox 等於 viewport，但它們可以通過設定而不同 -->
<svg width="800" height="600" viewbox="0 0 800 600">
  <!-- 內容 -->
</svg>
```

如果您曾經閱讀 viewBox 的規範，您也許看過規範說您可以使用 viewBox 屬性來使 SVG 產生形變，例如：放大或位移。沒錯，後續我們甚至會用其特性裁切 SVG。

要理解 viewBox 和 viewport 之間的差異最好的方式便是直接觀察視覺化的結果，如果上面的說明已經讓您混亂，那就讓我們來看看一些範例，透過系列範例希望能讓您掌握 viewport 和 viewBox。讓我們先從簡單的開始，所以一開始我們讓 viewBox 和 viewport 的比例是一樣的，還有現階段我們先不探討 preserveAspectRatio 避免混亂。

# 與 viewport 寬高比相同的 viewBox 

範例中的 viewBox 寬高是 viewport 的一半，並且這次我們不改變 viewBox 的原點，`<min-x>` 和 `<min-y>` 都設為 0。

```html
<svg width="800" height="600" viewBox="0 0 400 300">
</svg>
```

所以 `viewBox="0 0 400 300"` 做了什麼？

* 先在 SVG canvas 中從 (0, 0) 到 (400, 300) 畫一個特殊的區塊
* 接著，依據此區塊裁切 SVG
* 將該區塊放大到塞滿整個 viewport
* 對應用戶座標與 viewport 座標轉換單位，在這個範例下，一個`用戶單位`具體的大小會等於 2 倍的 viewport 單位

下圖說明了將上面 `viewBox` 的屬性加到 `<svg>` 時的狀況。灰色代表 viewport 座標，青藍色表示 viewBox 座標即用戶座標系統。

![](https://sarasoueidan.com/images/viewbox-400-300-crop.jpg)

任何您畫在 SVG canvas 上的東西都會跟這個新產生的用戶座標有對應的關係，或者說坐落在這個座標系統上。

> 視覺化 SVG canvas 搭配 viewBox 的行為類似於 Google Map。你可以縮放特定區域，當放大時只有在 viewport 範圍的東西可以被呈現，雖然我們知道剩下的東西都還在地圖上，但它們不會顯示因為超出 viewport 邊界的內容會被裁切掉。所以具體來說裁切的動作是 `viewport` 執行的，viewBox 只是畫好區塊。

現在讓我們修改 `<min-x>` 和 `<min-y>` 的值。值可以是任何數字，但這邊我們將兩個值設成 100。寬高維持一樣的比例。

```html
<svg width="800" height="600" viewbox="100 100 200 150">
  <!-- 內容 -->
</svg>
```

套用 `viewBox="100 100 200 150"` 的效果同樣會像上一個範例一樣產生裁切的效果。

![](https://sarasoueidan.com/images/viewbox-200-150-crop.jpg)

同樣的，用戶座標會對應 viewport 座標。即 200 的用戶單位會對應 800  的 viewport 座標單位，等於實際長度是 4 倍的 viewport 單位。這個結果就像是 zoom-in 的效果。

同時注意到在這個時候，設定的 `<min-x>` 和 `<min-y>` 也對圖片產生了形變的效果，更具體的說就是 SVG canvas 被位移了 100 個單位 `transform="translate(-100, -100)"`。

的確如同 SVG 規範所說的 `viewBox 屬性的效果是自動計算變形矩陣並對應到特定的空間，通常這個空間是 viewport`。

這行為看似很複雜，但就如同我們之前說的，就是先裁切圖片，然後塞滿 viewport。接著規範加上了註解：在某些情況下瀏覽器處理 SVG 的部分(User Agent)除了要提供 scale 縮放的資訊外還得提供 translate 位移的資訊。當 viewBox 的 `<min-x>` 和 `<min-y>` 不是 0 的時候最外層的 svg 元素就會需要 translate 的相關資訊，簡單說就是為了要填滿 viewport 而移動了畫布。

為了示範圖形的位移(translate transformation)，讓我們套用負值 -100 到  `<min-x>` 和 `<min-y>`。從左上角 (-100, -100) 裁切之後把該區塊`填滿` viewport 的行為，其效果具體看起來是像是 `transform="translate(100, 100)"`。 意味著圖片在裁切和放大之後會往右下搬。讓我們來看看下面的範例：

```html
<svg width="800" height="600" viewbox="-100 -100 400 300">
  <!-- 內容 -->
</svg>
```

套用上面 viewBox 屬性的結果會如下圖

![](https://sarasoueidan.com/images/viewbox-400-300-crop-translate.jpg)

注意！不同於 `transform` 屬性，這種由 viewBox 造成自動添加的 transformation (變形，位移效果)並`不會影響設定 viewBox 元素本身的 x, y, width, height 屬性`。因此在上面這個範例中的 svg 元素仍然保有原本的 width, height, viewBox。
width 和 height 表示的是 viewBox 變更用戶座標前的值，然後看看灰色的尺規(表示 viewport 座標系統)在 viewBox 屬性產生效果之後 svg 仍然維持原本的屬性值。

其他方面則跟 `transform` 屬性一樣，它會建立一個新的座標系給其他子元素使用。您可以觀察到上面的範例用戶座標是新的，跟原本初始化時跟 viewport 座標相同的那個已經不一樣了。並且在 `<svg>` 下的元素都依據這個新的座標系重新定位，改變尺寸。

最後一個關於 viewBox 的範例類似於上一個，不過這次我們不是要裁切，而是要擴展，來看看在 viewport 下圖片會有什麼變化。
我們將 viewBox 的寬高設的大於 viewport 的寬高，但我們仍保持一樣的比例。後續我們在討論關於不同比例的部分。

下面的範例我們讓 viewBox 設為 viewport 的 1.5 倍。

```html
<svg width="800" height="600" viewBox="0 0 1200 900">
  <!-- 內容 -->
</svg>
```

其結果就是現在用戶座標擴大到 1200x900。然後一樣對應 viewport 的座標系轉換具體渲染輸出的單位，也就是填滿 viewport 這個行為。所以現在每一個用戶座標 x 軸的單位需要乘上 `viewport.width /  viewBox.width` 來轉換，用戶座標 y 軸的單位也是要乘上 `viewport.height / viewBox.height` 轉換。在這種情況下 viewBox 上每一個 x 座標的單位會等於 viewport 單位乘上 0.66 換算，以此類推 y 軸單位。

當然最好的理解方式就是直接看圖。viewBox 會先被擴大，再填滿 viewport，最終的結果就如下圖。因為圖是基於新的用戶座標畫在 SVG canvas 上而不是 viewport 座標，從新座標系每個單位變成 0.66 倍，我們可以預見圖會變小。

![](https://sarasoueidan.com/images/viewbox-1200-900.jpg)

截止目前為止，所有的範例都跟 viewport 的寬高維持一樣的比例。那如果 viewBox 的寬高和 viewport 的不一樣時會如何？

假定 viewBox 的寬高為 1000x500，如此 viewBox 的比例就不再跟 viewport 一樣了。
看看下面的範例

![](https://sarasoueidan.com/images/viewbox-1000-500.jpg)

用戶座標和圖都要被放置在 viewport 中，也因此預設行為如下：

* 整個 viewBox 即 viewBox 切出來的區域要盡可能的執行`填滿` viewport 的動作
* viewBox 的寬高比會被保留，所以 viewBox 不會擴展完整覆蓋 viewport
* 在 viewport 區塊中，viewBox 會被水平、垂直置中

上面描述的是預設行為。那是什麼控制這個行為呢？如果我們想要改變 viewBox 在 viewport 中的位置那該怎麼做？
這兩個問題就是 `preserveAspectRatio` 存在的意義。

# preserveAspectRatio 屬性

`preserveAspectRatio` 屬性的目的是為了在縮放時強制固定圖片的比例。

假設我們定義了一個和 viewport 座標不同寬高比的用戶座標系，接著瀏覽器會將 viewBox 填滿 viewport。但因為比例不同的關係，所以導致圖片在 viewBox 展延時變形了。照上面 1000x500 的設定圖片如果不設定比例將會變成下圖這樣：

![](https://sarasoueidan.com/images/viewbox-1000-500-stretched.jpg)

當 viewBox 使用 `0 0 200 300` 時扭曲變形也是顯而易見的，使用 200x300 是因為這樣 viewBox 會剛好涵蓋整隻鸚鵡，然後 viewBox 會展延填滿 viewport，我們就觀察到圖片產出變形了

![](https://sarasoueidan.com/images/viewbox-200-300-stretched.jpg)

`preserveAspectRatio` 屬性讓我們可以強統鎖定 viewBox 和圖案的比例來進行縮放，並且可以設定 viewBox 在 viewport 中的位置，這是因為固定比例縮放有以導致 viewBox 不會跟 viewport 完全吻合，預設(在不提供任何設定時)會將 viewBox 保持比例縮放，並水平與垂直置中於 viewport 中。

# preserveAspectRatio 語法

關於 preserveAspectRatio 語法如下：

```
preserveAspectRatio = defer? <align> <meetOrSlice>?
```

preserveAspectRatio 可用在任何會建立 viewport 的元素上(下一篇我們會深入討論)。除了 `<image>` 外，對於支援此屬性的元素，preserveAspectRatio 只在元素設定 `viewBox` 時產生作用。

`defer` 是選擇性的參數，通常只有當我們要套用 `preserveAspectRatio` 到 `<image>` 上會使用，套用到其他元素時此屬性會被忽略。由於 `<image>` 已經超過本文要討論的範圍，所以這邊我們會略過 `defer`。

`align` 參數本身除了如同字面的意義`對齊`之外還可以用來設定是否固定比例，當 viewBox 和其元素的比例和 viewport 不同時產生效果。

如果 `align` 設為 `none`

```
preserveAspectRatio="none"
```

圖片將會填滿 viewport 不管 viewBox 的比例，也就是上面變形的 2 個例子。

除了 `none` 外其他的設定值將會固定 viewBox 的比例，然後依照設定在 viewport 中對齊。

最後的參數 `meetOrSlice` 也是選填的，預設為 `meet`。這個參數設定了 viewBox 是否要完整呈現於 viewport 中。
`align` 和 `meetOrSlice` 兩個參數一起使用的寫法如下，中間需要插入一個空白：


``
preserveAspectRatio="xMinYMin slice"
``

一開始看到這些設定值也許會覺得很怪，不過我們大致上可以把 `meetOrSlice` 想成是 CSS 中 `background-size` 的 `contain` 和 `cover`，`meet` 的行為很像 `contain` ，而 `slice` 則像 `cover` 。下面我們先來討論關於 `meetOrSlice` 的參數

# meet (預設值)

儘可能的讓圖片填滿 viewport 同時遵循下列規則

* 固定 viewBox 比例
* 整個 viewBox 需完整呈現在 viewport 中，原理概略為取得寬高的比例 viewport.width / viewBox.width 和 viewport.height / viewBox.height。小的一邊儘可能填滿 viewport 對應的邊。

在這種情況下，因為比例不符合 viewport，viewport 的邊界會大於 viewBox 即 viewBox 繪製的區域小於 viewport 的區域。這就是 `meet` 參數的作用 viewBox 會完整呈現於 viewport 中。

> meet 類似於 background-size: contain 。背景圖會儘可能的放大同時固定比例，最後確保整張圖可以被完整呈現在背景容器裡。當圖片與元素容器比例不同時，圖片就無法完整覆蓋元素。

# slice

縮放並固定圖片比例讓 viewBox 覆蓋整個 viewport。viewBox 會被縮放到剛剛好可以覆蓋 viewport，通常會有一邊超出 viewport，超出的部分就會被 viewport 裁掉。這個參數的行為類似於 `background-size: cover`，在背景圖的情況，圖片保持寬高比，然後縮放到寬高可以完全覆蓋元素的背景定位區的最小尺寸。

所以 `meetOrSlice` 可以用來設定 viewBox 是否要完全呈現在 viewport 中或者是盡可能縮放覆蓋整個 viewport。

舉例來說：如果我們同樣使用 200x300 的 viewBox align 都維持置中，`meet` 和 `slice` 的效果會分別如下：

![](https://sarasoueidan.com/images/viewbox-200-300-meet-vs-slice.jpg)

`align` 的參數有 9 + 1 種，除了 `none` 其他的值都會啟用固定比例和設定對齊的方式。align 的行為類似於 `background-position` 使用百分比的用法。例如 `background-position: 50% 50%;` 就是把圖片 X 座標 50% 的位置對齊元素 X 座標 50% 的位置。
把 viewBox 想成是一張背景圖，不過和 `background-position` 不同是我們不是使用百分比來取座標點而是直接指定 viewBox 和 viewport 對齊的 xy 軸。

為了理解 `align` 我們要先介紹`軸`的部分。

記得 viewBox 的 `<min-x>` 和 `<min-y>`？我們要使用的就是 viewBox 的 `min-x` 軸和 `min-y` 軸，另外 `max-x` 和 `max-y`
分別是透過 `<min-x> + width` 和 `<min-y> + height` 計算而得的。最後還有 `mid-x` `mid-y` 計算方式即 `<min-x> + (width/2)` 和 `<min-y> + (height/2)`。

上面的說明越看越亂？好吧讓我們透過下圖，看看這些軸各自在哪。在圖片中 `<min-x>` 和 `<min-y>` 預設是 0 因為 `viewBox=“0 0 300 300”`

![](https://sarasoueidan.com/images/viewbox-x-y-axes.jpg)

灰色的虛線是 viewport 的 `mid-x` 和 `mid-y` 軸。我們將要設定一些值來讓 viewBox 對齊 viewport 的軸。

接著我們來一一看看所有設定值：

## none

不執行固定比例縮放。換句話說就是 viewBox 直接填滿 viewport 不管 viewBox 的比例，圖片可能因此變形。

> 注意：align 設為 none 時 meetOrSlice 直接無效

## xMinYMin

* 固定比例縮放
* viewBox 的 `min-x` 和 viewport x 的最小值對齊
* viewBox 的 `min-y` 和 viewport y 軸的最小值對齊
* 行為類似於 background-position: 0% 0%;

## xMinYMid

* 固定比例縮放
* viewBox 的 `min-x` 和 viewport x 的最小值對齊
* viewBox y 軸的中間點和 viewport y 軸的中間點對齊
* 行為類似於 background-position: 0% 50%;

## xMinYMax

* 固定比例縮放
* viewBox 的 `min-x` 和 viewport x 軸的最小值對齊
* viewBox 的 `min-y + height` 和 viewport y 軸的最大值對齊
* 行為類似於 background-position: 0 100%;

## xMidYMin

* 固定比例縮放
* viewBox x 軸的中間點和 viewport x 軸的中間點對齊
* viewBox 的 `min-y` 和 viewport y 軸的最小值對齊
* 行為類似於 background-position: 50% 0%;

## xMidYMid

* 固定比例縮放
* viewBox x 軸的中間點和 viewport x 軸的中間點對齊
* viewBox y 軸的中間點和 viewport y 軸的中間點對齊
* 行為類似於 background-position: 50% 50%;

## xMidYMax

* 固定比例縮放
* viewBox x 軸的中間點和 viewport x 軸的中間點對齊
* viewBox 的 `min-y + height` 和 viewport y 軸的最大值對齊
* 行為類似於 background-position: 50% 100%;

## xMaxYMin

* 固定比例縮放
* viewBox 的 `min-x + width` 和 viewport x 軸的最大值對齊
* viewBox 的 `min-y` 和 viewport y 軸的最小值對齊
* 行為類似於 background-position: 100% 0%;

## xMaxYMid

* 固定比例縮放
* viewBox 的 `min-x + width` 和 viewport x 軸的最大值對齊
* viewBox y 軸的中間點和 viewport y 軸的中間點對齊
* 行為類似於 background-position: 100% 50%;

## xMaxYMax

* 固定比例縮放
* viewBox 的 `min-x + width` 和 viewport x 軸的最大值對齊
* viewBox 的 `min-y + height` 和 viewport y 軸的最大值對齊
* 行為類似於 background-position: 100% 100%;

所以使用 `preserveAspectRatio` 的 `align` 和 `meetOrSlice` 我們就可以設定 viewBox 縮放與定位的行為。我們可以選擇是要全部顯示或只顯示局部。

有時結果會取決於 viewBox 的尺寸，有些設定雖然不同但結果會一樣。例如：之前的 `viewBox="0 0 200 300"` 在 `meetOrSlice` 為 `meet` 的情況下，`align` 就算不同也會達成一樣的結果

![](https://sarasoueidan.com/images/viewbox-meet-align-same.jpg)

但如果我們把 `meetOrSlice` 換成 `slice` 那麽結果就完全不同了。因為 `slice` 設定下 viewBox 展延覆蓋 viewport 的行為 - x 軸的 200 單位被換算並覆蓋了 viewport 的 800 單位，同時為了固定比例 viewBox 的 y 軸勢必會大於 viewport 的 y 軸而造成被 viewport 裁切。
觀察下圖

![](https://sarasoueidan.com/images/viewbox-slice-align-same.jpg)

下面我們不再示範更多例子，而是回到一開始那個互動的範例讓您親自調整 viewBox 和 preserveAspectRatio 的參數去驗證觀察。

不過在我們繼續往下走之前，我想提醒一下關於 `mid-x` `mid-y` `max-x` `max-y` 的值是會隨著 `min-x` 和 `min-y` 而改變的。

> viewBox 並不會真的裁切隱藏畫在 SVG canvas 上的圖，當比例不同時 viewBox 所定義的矩形區塊會作為縮放(單位實際值轉換)和對齊的依據。

下圖是 `viewBox="100 0 200 300"` 的效果，`meetOrSlice` 維持預設 `meet` 觀察 mid-x 和 max-x 是如何變化，是否跟您所預期的效果一致。

![](https://sarasoueidan.com/images/viewbox-axes-changed-min-x-min-y.jpg)

# 互動範例

要理解 viewport、viewBox 和 preserveAspectRatio 之間的關係最好的方式就是直接看視覺化的結果。
為了這個目的文章提供了這個簡單的互動範例，您可以透過修改參數的值直接觀察結果。

![](https://sarasoueidan.com/images/viewbox-demo-screenshot.jpg)

[查閱範例](https://sarasoueidan.com/demos/interactive-svg-coordinate-system/index.html)

希望您能從這篇文章更加理解 SVG 中 viewport、viewBox 和 preserveAspectRatio 的概念。如果您想要更深入的理解關於 SVG 座標系統，例如巢狀座標，如何在 SVG 中建立新的座標系與變形的議題，您可以繼續閱讀後續的系列文章。

# 資源參考

* [原文](https://sarasoueidan.com/blog/svg-coordinate-systems/)
* [MDN SVG](https://developer.mozilla.org/en-US/docs/Web/SVG)
* [w3c](https://www.w3.org/TR/SVG/coords.html)
* [理解SVG viewport,viewBox,preserveAspectRatio缩放](http://www.zhangxinxu.com/wordpress/2014/08/svg-viewport-viewbox-preserveaspectratio/)
* [簡體中譯](https://www.w3cplus.com/html5/svg-coordinate-systems.html)

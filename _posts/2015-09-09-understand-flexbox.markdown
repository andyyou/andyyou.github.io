---
layout: post
title: '第一次用 CSS - Flexbox 就上手'
date: 2015-09-09 05:30:00
categories: Javascript
---

Flexbox 是一種新的 CSS 3 佈局模式, 在這之前 CSS 有四種佈局模式

* block
* inline
* table
* positioned

在 Flexbox 和 Grid 出現之前, 開發者大量濫用 `float` 來達成一些複雜佈局的需求, 但本質上 float 並不是用來協助我們佈局的, 它只是一個舊有的對齊屬性.
雖然 float 可以達成我們的目的, 但它伴隨著一些限制和問題, 您應該有過使用 float 搭配 clearfix 的經驗吧.

現在 CSS 3 引進了兩種新的佈局方式用來取代被我們大量濫用的 float 和 table mode

* `Grid` 將佈局區分成 rows 和 columns, 有點類似 table 但比其更強大. 不過關於 Grid 的規格仍然在開發中, 還不能使用.
* `Flexbox` 根據單一的行或列分配空間, 類似 float 調配控制元素的位置尺寸概念, 但更好用! Flexbox 規格已經完成而且今時今日多數的主流瀏覽器都支援


# Flexbox 簡史
明白創造這種佈局模式的動機是幫助我們學習這東西一個非常好的開始. 

> Tab Atkins Jr : 大約在 2000 - 2009 後期, Mozilla 試著要讓 XUL Layout model 正名為 Flexbox, 我第一次得知這份草稿是在 2009 年, 但我想這應該可以追朔到 2007 年. 但這過程並沒有什麼結果; 只有 WebKit 實作一部分標準, 而且即使是 Firefox 也不符合該標準, 因為大部份的事情根本還沒被確認.
> 接著在 2010 年左右我加入了 WG 其中的第一個任務就是替 Firefox 整理這份草稿(整份重寫), 大概在一兩年後 Microsoft 提交了第一份 Grid 標準的草稿, 我一樣是使盡全力處理(重寫)
> 我這麼做的目標就是要取代那些因為 `float`, `table`, `inline-block` 等等而延伸的複雜佈局技巧, 身為一個網頁開發者我不得不這麼做.
> 那一堆人們為了佈局而產生的奇技淫巧(hack)通常不是怎麼高明, 很難記憶, 而且往往有一堆惱人的限制, 所以我希望讓佈局模型可以更好用一點, 在處理同樣的問題時可以更簡單, 方便並且相對完整.

簡而言之: Flexbox 和 Grid 被創造出來明顯的是為了取代 float, table 等等問題而產生的 hack.

# 共有 3 種 Flexbox 規格?
你也許聽過有 3 個 Flexbox 的規格書, 沒錯! 共有三個版本的規格書, 不過只有一個是我們要關注的.

* 2009 年的版本: `display: box` 現在已經不再跟 Flexbox 有任何關係
* 2011 過渡期版本: `display: flexbox` 只是草稿, 只被 IE10 實作, 如果可能的話應該避免使用
* 2012 最終版: `display: flex`

注意: 每一個規格在 display 屬性上使用不同的關鍵字, 如果您正在閱讀其他關於 Flexbox 的文章可以很輕易的判斷該篇文章在討論的是哪個版本.
如果你看到的文章不是 `display: flex` 那麼你可以不看了

# Flexbox 佈局具備單一方向性
如同上面提到, Flexbox 會依據單一 row(橫向) 或 column(垂直) 方向放置內容項目, 讓我們來看看這句話的意思是:

![](https://d262ilb51hltx0.cloudfront.net/max/800/1*5rqfzizVze5BEstfXNKnfQ.png)

一個 Flexbox 佈局由一個 `flex container` 組成, 裡面包含著 `flex items` 這個 flex container 可以設置水平或垂直, 我們可以簡稱這個特性為主軸 `main axis` 

![](https://d262ilb51hltx0.cloudfront.net/max/800/1*xo0yJQCryqinH2PhvVMwkw.png)


`flex container` 內部第一層的子元素會遵循 `main axis` 的方向排列, 這些子元素可以`彈性`調整他們的大小, 自動擴展去使用 container 內部那些未被使用的空間, 或者自動縮小以避免超出範圍.

透過巢狀內嵌多個不同方向的 `flex container` 可以完成複雜的佈局.

# Flexbox 屬性

關於 Flexbox 有非常多的屬性, 所以我會建議你如果有時間參考一下[CSS Tricks Flexbox Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)這邊文章非常詳細的介紹了所有的屬性

不要被嚇到了. 雖然 Flexbox 有非常多的屬性可以設定, 但是也有一系列的預設值, 所以我們可以透過很少的程式碼來建立複雜的佈局.

{% highlight css %}
.flex-container {
  display: flex;
}
{% endhighlight %}

單單一行 CSS 程式碼我們完成了下面這些功能

* 套用 `.flex-container` 的元素成為一個 `flex container` 
* 內部第一層的子元素會變成 `flex item`
* 預設 Flex item 會依照水平的方式排列
* Flex item 會依照程式碼的順序排列
* Flex item 會在 container 內從左至右排列
* Flex item 的尺寸會依照正規的 width 屬性或內容自身的寬, block-box(div etc) 不會自動長滿 100%
* 如果沒有足夠的空間, 那麼 Flex item 會自動依照寬度 width 的比例去縮小, 直到空間完全不足 scrollbar 才會出現
* 如果需要收縮, 則每一個 Flex item 都會收縮相同的值
* Flex item 會自動展延高度使其與其他 Flex item 中最高的那個相等

對於這只有一行的 CSS 的確是包含蠻多的邏輯! 你還可以修改其中任何一個行為, 不過我希望能夠讓讀者明白關於 Flexbox 的強大

# 瀏覽器的支援

![](https://d262ilb51hltx0.cloudfront.net/max/800/1*0GiTml6PceOb6fjvXYiYFw.png)

注意 IE 10 支援 Flexbox 不過是使用過渡時期的版本 `display: flexbox`

# 那麼對於 IE 10 以前的版本呢

如果你不在乎不支援 Flexbox 的瀏覽器破版, 有點的不同, 如果是這樣你就不需要在做其他處理. 這種行為有個術語叫做優雅降級(graceful degradation)
當瀏覽器看到它不懂的東西 - 例如: `display: flex` 他就會整個忽略, 對我們來說這是件好事, 因為這表示所有 Flexbox 的屬性都會被忽略.
您設定成 Flex container 的元素會用原本的行為運作通常應該是 `display: block` 就結果來說整個佈局中 block 會垂直排列

# 搭配 Modernizr 的優雅降級
如果你不能不管那些古董瀏覽器, 您可以透過 [Modernizr](http://modernizr.com/) 提供一個舊版使用 `float-base` 的佈局
Modernizr 是一個 Javascript 函式庫, 它會自動偵測瀏覽器的功能然後在 html 或 body 元素加上一系列的樣式, 讓我們可以根據是否支援來撰寫替代性(舊版 float-base)的樣式

以 Flexbox 的例子來說如果瀏覽器不支援 Flexbox 那麼就會有一個 `.no-flexbox` class 被加在 html tag 上面, 如此一來我們就可以用下面的樣式來試著用其他的樣式取代

{% highlight css %}
.parent { display: flex; }
.child { flex: 1; }
.no-flexbox {
  .child { float: left; width: 50%; }
  .parent::after {
    @include clearfix();
  }
}
{% endhighlight %}

好處是您可以透過舊有的方式來針對 `.no-flexbox` 的部分做 CSS 的撰寫, 如果未來不再需要支援這些老舊的瀏覽器只要把 .no-flexbox 部份的樣式移除即可

不過還是有一點要注意, 當你使用 Modernizr 2.8.3 時 IE 10 並不會出現 `.no-flexbox` 你可以改用IE專屬的條件式註解語法 `<!--[if IE 10]><![endif]-->`, `autoprefixer`, 或者直接更新使用 Modernizr 3+ 新版已經排除掉這個問題.

# Flexbox 教學 - 背景
關於 `Flexbox 佈局` 模組目前處在 W3C 最後的草稿階段, 其目標如同上面提到的希望提供一種更有效率的方式處理元素的佈局, 對齊, 在一個容器內分散排列的方式, 甚至是不知道該元素的尺寸或者說動態調整, 這也是為什麼會叫 `Flexible Box`

在 Flex 佈局背後的核心觀念是賦予一個 container 容器具有能力去修改子項目的尺寸(width/height)以及排列順序, 好讓我們可以填滿容器裡的空間更具體的說好讓我們可以去適應各種尺寸裝置的螢幕. 一個 Flex container 可以擴展其子項目填滿容器裡的空間或者是收縮以防止超出容器.

更重要的是 `direction-agnostic` 這是 [W3C 定義的術語](http://www.w3.org/TR/2012/WD-css3-flexbox-20120322/)

> To make it easier to talk about flexbox layout in a general way, we will define several direction-agnostic terms here to make the rest of the spec easier to read and understand. 

指的是那一系列處理方向的術語, 這跟原本的 block, inline 等佈局方式相反, 意思是指原本的佈局形式, 項目本身會具方向性例如: block 會垂直堆疊, inline 會水平排列, 而 Flex item 本身不需要知道該怎麼處理方向和排列, 是由 container 控制. 

原本的方式類似在做平面設計排版, 缺少彈性去支援大型複雜的專案尤其是當裝置螢幕切換方向, 改變尺寸, 伸縮那些狀況.

備註: Flexbox 佈局相對適合用於應用程式中的元件, 小型的佈局. 對於複雜佈局來說 Grid 會相對適合.

# 基礎知識和術語

由於 Flexbox 代表的是一整個模組, 是一系列排版的邏輯而不只是單一的 CSS 屬性, 意思是它牽扯到很多東西和屬性. 
其中有些屬性是要用在 Flex container 的, 另一些則是用在子項目 Flex item 

如果說一般的佈局架構在 block 和 inline 的方向性上, 那麼 Flexbox 佈局則基於 `flex-flow` 方向, 參考下圖, 這邊就是解釋了上面提到的 w3c 定義的那些術語, 透過圖片我們很清楚的可以明白關於 Flexbox 的核心概念.

在這一小節重點是跟以往下 CSS 屬性有些不同, Flexbox 佈局的使用和樣式設定上區分成

* Flex container - 設定為 `display: flex` 的元素
* Flex item - 在 Flex container 內部第一層直接與 container 接觸到的標籤, 可以是文字或其他標籤 e.g div span

container 的設定會影響 item, 位於內部的 item 也有些屬性可以覆寫 container 的設定, 為了方便說明下面開始我們會用 `container` 和 `item` 分別表示這兩者

![](https://cdn.css-tricks.com/wp-content/uploads/2011/08/flexbox.png) 

基本上 item 會依照 `main axis` 主軸排列, 從 `main-start` 到 `main-end` 和 `cross axis`  橫軸從 `cross-start` 到 `cross-end` 排列


* `main axis` - 即 Flex container 的主軸, 任何在內部第一層直接和 container 連接的項目稱為 Flex item 都會遵循這個主軸排列, 要注意的是這個主軸不一定要是水平的, 根據 `flex-direction` 屬性的設定可以改變方向性
* `main-start`|`main-end` - 在 container 內部的 item 會根據主軸從 `main-start` 開始向 `main-end` 排列放置
* `main-size` - 因為 `flex-direction` 屬性的設定可以改變方向性, 依據主軸方向該 item 的寬或高就是 `main-size`, 其屬性值來自於 item 的 `width` 或 `height`
* `cross axis` - 側軸(橫軸)指的是垂直於主軸的另外一個維度向性, 他的方向性會根據主軸而改變, 相依於主軸.
* `cross-start`|`cross-end` - item 會依照 `Flex Line` 排列放置, 類似於主軸的概念, 只不過這是另外一個維度
  ![](https://s3.amazonaws.com/static.bocoup.com/blog/flexbox%20diagram%201.jpg)
* `cross size` - 概念與 main-size 相同, 不過換成側軸上 item 的寬或者高

# Flex Container 屬性

![](https://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-container.svg)

### display

這個屬性用來定義產生一個 flex container; 一旦設定 `flex` 其第一層子項目就會變成 flex item.
屬性值有 `flex`, `inline-flex`. 另外注意 CSS `columns` 在 flex container 中是沒有作用的.

> flex container 並不是一般的 block, 也因此我們常常在用的一些控制位置, 對齊的屬性在這個容器中並不適用也就是說在 item 設定 `column-*`, `float`, `clear`, `vertical-align` 都是沒有作用的. 但是!! 對容器本身是有用的.

> flex 和 inline-flex 對於內部的 item 使用起來並沒有差異, 差別是容器本身的行為, inline-flex 容器本身就會類似 inline 一樣不會預設佔滿一 row

{% highlight css %}
.container {
  display: flex; /* or inline-flex */
}
{% endhighlight %}

### flex-direction

![](https://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-direction1.svg)

此屬性用來設定主軸的方向, 基本上是單向的佈局概念, item 可以水平或垂直排列, 一次只能設定一種方向

{% highlight css %}
.container {
  flex-direction: row | row-reverse | column | column-reverse;
}
{% endhighlight %}

* row - 預設的屬性值, 在文字方向設定 `direction: ltr;` 時 row 就是由左至右, 反之 `rtl` 則由右至左
* row-reverse - 反向的 row 
* column - 跟 row 的概念一樣不過這是垂直的
* column-reverse - 跟 row-reverse 概念相同, 但是是垂直的

### flex-wrap

![](https://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-wrap.svg)

預設 flex item 只會排成一排, 你可以修改這個屬性當 item 塞不下的時候換行

{% highlight css %}
.container{
  flex-wrap: nowrap | wrap | wrap-reverse;
}
{% endhighlight %}

* nowrap - 預設值, 只會有一行
* wrap - 多行, 塞不下就換行
* wrap-reverse - 多行, 反向排列

![](http://i.imgur.com/LvgBxq3.png)

### flex-flow

這是 `flex-direction` 和 `flex-wrap` 的縮寫版預設值是 `row` `nowrap`

{% highlight css %}
.container{
  flex-flow: <‘flex-direction’> || <‘flex-wrap’>
}
{% endhighlight %}

### justify-content

![](https://cdn.css-tricks.com/wp-content/uploads/2013/04/justify-content.svg)

設定主軸的對齊方式, 這個屬性可以協助我們分配容器中扣除 item 的空間, 其行為是當所有可伸縮的長度及邊距都完成計算後剩下的空間才輪到 justify-content 來分配

{% highlight css %}
.container {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
{% endhighlight %}

* flex-start - 預設值, item 對齊主軸的起始點(邊界), 後續的 item 一個接著一個
* flex-end - item 從主軸的終點邊界開始往回排列, 注意到我們在 Flexbox 裡 item 的順序是獨立出來的, 不要把順序和排列該對齊的位置搞在一起.

![](http://i.imgur.com/Mr4Gmrl.png)

* center - item 置中, 換個角度看就是把剩下個空白平均分給兩邊

![](http://i.imgur.com/C6D8OhN.png)

* space-between - item 被平均分配到主軸上, 也就是說剩下的空間平均分到到 item 之間的間隔, 注意頭尾是貼齊邊界. 也就是說如果沒有多餘的空間其效果跟 flex-start 一樣

![](http://i.imgur.com/UQ0ezNi.png)

* space-around - 把剩餘的空間平均分配到 item 的兩邊, 頭尾並不會貼齊邊緣, 類似下了左右的 margin.

![](http://i.imgur.com/9cwAsJ8.png)

### align-items

![](https://cdn.css-tricks.com/wp-content/uploads/2014/05/align-items.svg)

這個屬性用來設定 item 該如何沿著側軸(cross axis)對齊排列, 要注意主軸跟側軸的關係, 因為不見得主軸就是橫向的.
根據大部份的情況, 把它想成用來處理垂直置中的屬性比較好記. 是指每個個別的 item 跟怎麼跟側軸對齊

{% highlight css %}
.container {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
{% endhighlight %}


* flex-start - 貼齊側軸的起始點 `cross-start`.
* flex-end - 貼齊側軸 `cross-end` 排列
* center - 置放於側軸的中央
* baseline - item 會對齊 baseline
* stretch - 預設值, 自動把 item 的高長滿 container

### align-content

![](https://cdn.css-tricks.com/wp-content/uploads/2013/04/align-content.svg)

這個屬性比較像是垂直版的 `justify-content`, `align-items` 是以 item 的角度對齊, 而 `align-content` 比較像是根據 row 的高這個角度, 這個屬性的行為就是分配剩下的空間, item 高都計算完成之後, 該如何剩餘的空間. 

注意, 當只有一行的時候這個屬性沒有效果

{% highlight css %}
.container {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
{% endhighlight %}

* flex-start - 對齊 container 側軸的起始點(cross-start)一行一行排列, 在一般水平為主軸的情況下看起來像是整塊對齊 top
* flex-end - 對齊側軸尾巴(cross-end)
* center - 整塊置中, 意思是剩餘的空間上下留白
* space-between - 頭尾對齊邊緣, 剩下的空間分配到 item 之間的間隔
* space-around - 把剩餘的空間平均分配到 item 的兩邊, 頭尾並不會貼齊邊緣, 類似下了左右的 margin. 不過這次是側軸的方向.

### 小結

概略來說 container 決定 item 排列方向, 用 justify-content, align-content 來處理分配剩餘空間, 對齊. 側軸方向上還可以針對 item 該怎麼對齊用 align-items 甚至可以輕鬆完成每個項目同高, 或是各自有自己的高, 內部 item 可以依照比例伸縮, 且不會超過容器.
可動態適應各種不同的螢幕尺寸.


# Flex Item 屬性

![](https://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-items.svg)

### order

![](https://cdn.css-tricks.com/wp-content/uploads/2013/04/order-2.svg)

預設來說 item 會依照程式碼出現的順序排列, 然而 Flexbox 強大之處就是你可以設定 `order` 來調整排列順序, 順序用一個整數來表示, 也可以使用負數, 不過下面的 flex-grow 就不能用負數

{% highlight css %}
.item {
  order: <integer>;
}
{% endhighlight %}

### flex-grow

![](https://cdn.css-tricks.com/wp-content/uploads/2014/05/flex-grow.svg)

此屬性用來讓 item 具有伸展擴大的能力, 使用一個沒有單位的整數(負數不是合法的值), 如果有剩餘的空間就會依照數字的比例去伸長 item.
舉例來說如果所有的 item 的 `flex-grow` 都設為 1 那麼當需要展延的時候所有 item 會分配到相同的長度. 如果你賦予其中一個 item 的 flex-grow: 2
這個 item 就會被分配到比較多的空間.

或者更確切的例子假設容器 1000px, 有三個 item 各為 200px 加上 `margin: 0 20px`, 如此一來 `1000 -  (200 + 20 * 2) * 3` = 280
這 280px 會依據 `flex-grow` 設定的比例去分配

另外需要注意的就是如果 item 設為 `flex-grow: 0` 則不參與空間分配

{% highlight css %}
.item {
  flex-grow: <number>; /* default 0 */
}
{% endhighlight %}

### flex-shrink

當空間不足時使 item 具有縮小的功能即依照比例縮小, 同樣負數無效
舉例來說容器 200px 三個 item 各自 100px 加上 item 有 `margin: 0 10px` 那麼三個 item 的總寬是 `(100 + 10 * 2) * 3 = 360`
總共超過 160px, 所以為了要能夠塞進容器裡, 現在我們需要砍掉 160px, 如果設定是 1:1:1 的話就是 `160/3=53.33` 也就是每個 item 要砍掉 53.3px
也就是說在 flex-shrink 裡數字越大砍掉越多.

{% highlight css %}
.item {
  flex-shrink: <number>; /* default 0 */
}
{% endhighlight %}

### flex-basis

設定 item 的初始值, 即在 flexbox 機制開始分配剩餘空間調整 item 大小之前賦予一個初始值, 預設值是 auto
當 `flex-basis` 不設定或設為 auto 的時候則 item 的尺寸會根據自身的屬性(width)加上 content 的寬度去計算, 概念上我們可以把其當成是 `min-width`.

{% highlight css %}
.item {
  flex-basis: <length> | auto; /* default auto */
}
{% endhighlight %}

### flex

flex-grow, flex-shrink, flex-basis 的濃縮寫法第一個參數 flex-grow, 第二個是 flex-shrink 第三個是 flex-basis 預設值為 `0 1 auto`

{% highlight css %}
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
{% endhighlight %}

W3C 文件建議我們善用這個屬性取代其他三者

### align-self

![](https://cdn.css-tricks.com/wp-content/uploads/2014/05/align-self.svg)

剛剛上面我們看過了 `align-items` 不過 `align-items` 是從 flex container 的角度讓所有的 item 設定同一個值, 而 `align-self` 則是從 item 的角度去設定, 可以讓我們達到某一個 item 覆寫不同值

{% highlight css %}
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
{% endhighlight %}

再次強調 `float`, `clear`, `vertical-align` 對於 item 是沒有作用的.

### 小結

簡單來說 container 處理整體的方向, 空間開如何分配而 item 的屬性部分就是 order 順序, 和當空間超出去或不夠的時候 item 該如何動態增減其長度這個部分主要是針對 main size.

# 參考
[a-guide-to-flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

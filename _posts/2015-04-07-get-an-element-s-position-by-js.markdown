---
layout: post
title: '使用 Javascript 取得元素的座標'
date: 2015-04-07 17:31:00
categories: Javascript
---
# 前言
由於使用 Javasript 偵測元素尺寸的方式並不直覺，每個元素有 6 個 DOM 的尺寸的屬性 `offsetWidth`, `offsetHight`, `clientWidth`, `clientHeight`, `scrollWidth`, `scrollHeight`。
再加上 `offset[Top | Left]`, `scroll[Top | Left]`, `client[Top | Left]` 等方向距離的屬性導致這件事變得異常複雜，外加它們都是整數因此在一些操作上會有些誤差。
在開始之前對於那些急性子的人我先提供他們一些對於座標屬性的整理，您可以先大略看過這些整理，後面我們將針對一個實際的例子來練習，這些整理應該可以讓一些老手快速的回復記憶，其實就是因為這樣所以我才紀錄了這篇文章。

先附上兩張圖式
![]({{ site.url }}/assets/images/tutorials/position_1.png)
![]({{ site.url}}/assets/images/tutorials/css_box_model_1.png)

## CSS Box 為 content-box 時的情況(下面為 HTML DOM 屬性)

- `offsetWidth`/`offsetHeight`: 
  + 元素 Box Model 總寬高 (width + padding + border)
  + 在 `box-sizing: content-box;` 時 width = content area width
  + [範例](http://codepen.io/AndyYou/pen/bNyQrv)
  + 不管是否超出父元素限制範圍都是總寬高

- `clientWidth`/`clientHeight`:
  + 一般情況下即元素 Box Model 可視區的 width + padding
  + `可視區` 只針對取值的元素本身，以元素本身的角度出發，意思是當我們限制元素寬高時只有能看見的部分會列入計算，扣掉 scrollbar width
  + 如果有個子元素超過自己的寬高則 clientWidth 和 clientHeight 仍然是 width + padding
  + [範例](http://codepen.io/AndyYou/pen/emaQvX)
  
- `scrollWidth`/`scrollHeight`:
  + 整個框內的總寬高
  + 元素本身的 padding + 內部元素寬高，
  + 舉例 (scrollHeight) 會是`self_padding` + `children_margin` + `children_border` + `children_padding` + `children_height`
  + [範例](http://codepen.io/AndyYou/pen/Joqeog)

- `offsetTop`/`offsetLeft`:
  + 定義 `elem.offsetTop` 為唯讀的屬性，會回傳目前元素與 `offsetParent` 元素的距離
  + 當 `position` 為 `static` (沒設定預設是 static) 時 `offsetParent` 就會是根節點(root) 或是外層結構中最接近的 table cell 元素。其他有 `position` 的屬性( fixed, absolute, relative)都會讓被設定的外層元素變成 `offsetParent`
  + 計算元素和 offsetParent 的距離 => 從元素本身的 margin + offsetParent 的 padding
  + 當元素 CSS 有 `display: none;` 時，offsetParent 為 null
  + [範例](http://codepen.io/AndyYou/pen/QwRJaV)

- `clientTop`/`clientLeft`: 
  + 單純就是 border 寬度
  + 定義為回傳該方向的 border 寬度，單位用 px
  + 該屬性不包含元素的 padding 或者 margin
  + 用 `document.getElementById("elem").style.borderTopWidth` 這類方法取得一樣的值

- `scrollTop`/`scrollLeft`:
  + 如果該目標元素沒有 scrollbar 則值為 0 例如: 沒有 y 軸 scrollbar 則 `scrollTop = 0`
  + 從元素 border 內緣開始計算，scrollTop 與 scrollLeft 是取有捲軸的那個元素捲到哪
  + 按照規定 scrollTop 不會小於 0 但是在 OSX 下的 Chrome 和 Safari 可能會產生負值。
  + [範例](http://codepen.io/AndyYou/pen/LEoXBR)

## CSS Box 為 border-box 時的情況

- `offsetWidth`/`offsetHeight`: 
  + 由於 `border-box` 的關係 css 設定的 width 會等於總寬。
  + 注意: 在 `border-box` 模式下 width 不等於 content area 的寬，而是整個 block (不含 margin)

- `clientWidth`/`clientHeight`:
  + border-box 狀態時算法變成從外面減回去 width - border = content area + padding

- `scrollWidth`/`scrollHeight`:
  + 取得值沒有差異，雖然是往內扣但該有的 padding 還是存在。

- `offsetTop`/`offsetLeft`:
  + 取得值沒有差異

- `clientTop`/`clientLeft`: 
  + 取得值沒有差異

- `scrollTop`/`scrollLeft`:
  + 取得值沒有差異

# 理解 Javascript 與 DOM 座標

雖然大部份的開發者對於 CSS 的 Box Model 與排版操作十分熟悉不過當我們想要使用 Javascript 來操作這些位置的時候可能會遇到障礙。
問題通常出在如何準確擷取到您要的座標資料，一般來說元素本身的座標(排版位置)都不是您直接定義，而是被父元素的 `float`, `padding`, `margin`, `position` 等等屬性所影響。極少的情況下才會設定絕對位置的 x, y。

為了能夠精準的控制位置，我們得先知道元素的 x, y 座標。會搞得這麼複雜的原因是因為 HTML DOM API 或 Javascript 並沒有內建的功能來處理這些任務。

在這份簡短的教學中，您會學到如何取得 HTML 元素精準的座標和背後的原理。

# 我們有一個需求
在我們開始寫程式之前讓我們先假設遇到一個需求 - 我們想要取得 HTML 元素確切的 x, y 座標。

![]({{ site.url }}/assets/images/tutorials/position_3.png)

一般來說，所有的座標設定的行為都需要一個相對的起始點，通常我們會拿 document 最左上角的點來當對應的起始點。而我們放的位置`點`則對應到元素的最左上角。

現在問題來了您的 HTML 文件並不是一張 bitmap 或者 canvas。如同前面所提到的 HTML 和 Javascript 並沒有一套內建的機制處理這種情形。所以我們需要參考其他元素的相關設定或樣式來得到這個座標，因為他們通常都是透過一堆 CSS 樣式來排版的。
接著讓我們透過一個範例來理解其中是如何運作的，最後我們將融會貫通這些技巧來完成我們的需求。

# 取得座標的程式碼
當我們使用原生 Javascript 要取得 HTML 元素的 x ,y 的座標範例如下

{% highlight js %}
function getPosition (element) {
  var x = 0;
  var y = 0;
  // 搭配上面的示意圖可比較輕鬆理解為何要這麼計算
  while ( element ) {
    x += element.offsetLeft - element.scrollLeft + element.clientLeft;
    y += element.offsetTop - element.scrollLeft + element.clientTop;
    element = element.offsetParent;
  }
  
  return { x: x, y: y };
}
{% endhighlight %}

`getPosition` 這個 function 會取得 HTML 元素在 document 中的位置並回傳一個包含 x, y 的物件。

下面則示範了如何使用

{% highlight js %}
  var elem = document.querySelector('img');
  var position = getPosition(elem);
  alert("座標: " + position.x + ', ' + position.y);
{% endhighlight %}

上面我們使用了 `querySelector` 函式來找到我們要的元素，其他原生的函式包含 `getElementById`, `getElementByTagName`, `getElementByClassName`。又或者您可以使用 `jQuery` 的 `$( ".girl" )[ 0 ]` 來取得元素的 DOM。

# 範例運作的狀況
為了看看上面這段範例程式碼的效果您可以直接連到 [範例](http://codepen.io/AndyYou/pen/RNmvXa)

範例中的提示框會告訴我們元素座標，如果您去量測瀏覽器的 `viewport` 可視區域的左上角到該圖片的左上角會如下得到準確的座標，注意 border 不會算在其中。

![]({{ site.url }}/assets/images/tutorials/position_4.png)

我們簡單的示範了最單純的例子但是實務上總不是那麼容易。

# 背後的原理
對大多數的狀況來說一個元素的位置取決於自己的 CSS 樣式，不過更多的部分是受到父元素樣式的影響。這些樣式包含了 `padding`, `margin`, `border` ，同時再加上 `position` 那事情就變得越來越亂了。

# 小技巧
記得直接用瀏覽記得工具來協助計算

![]({{ site.url }}/assets/images/tutorials/position_5.png)

# 解析取得座標的程式碼
對於上面的程式碼您應該有個概略的理解，但實作的時候您必須要精準的理解每個片段才行。現在讓我們來認真看看這些程式碼是怎麽運作的

首先再複習一下剛剛取得座標的程式碼

{% highlight js %}
function getPosition (element) {
  var x = 0;
  var y = 0;
  // 搭配上面的示意圖可比較輕鬆理解為何要這麼計算
  while ( element ) {
    x += element.offsetLeft - element.scrollLeft + element.clientLeft;
    y += element.offsetTop - element.scrollLeft + element.clientTop;
    // 這邊有個重點，當父元素被下了 position 屬性之後他就會變成 offsetParent，所以這邊我們用迴圈不斷往上累加。
    element = element.offsetParent;
  }
  
  return { x: x, y: y };
}
{% endhighlight %}

第一步是 function 的部分

{% highlight js %}
function getPosition(element)
{% endhighlight %}

我們定義了一個 `getPosition` 函式，有一個參數 `element` ，如您從剛剛的用法我們把要取得座標的元素(DOM物件)傳進去。

接著我們定義了兩個座標 x, y 並且初始化為 0

{% highlight js %}
var x = 0;
var y = 0;
{% endhighlight %}

這兩個變數會被用來儲存 x, y 座標，因為 `offset[Top/Left]` 取的是該元素和 `offsetParent` 的距離，當我們沒有使用 `position` CSS 規則時 `offsetParent` 會是 `root` 或 外層結構中離元素最近的 `table cell` (在 HTML 標準兼容模式中 root 為 body)，另外一旦您對元素設定了 `display: none;` 那麼 `offsetParent` 就會回傳 `null`。

原來 `offset[Top | Left]` 並不是像我們一開始認為的直接算出到最左上角的距離，是和 `offsetParent` 的距離。
意思是當外層的結構有元素套用了 `position: absolute;` 或 `relative` 等等的時候 `offsetParent` 就換人了，`static` 則維持預設。

繼續回到我們的程式碼，因為 `offsetParent` 不會永遠是 `root` 所以這邊我們用了一個小小的技巧

{% highlight js %}
// 現在您懂了為什麼我們要在這邊用 while!!
while(element) {
    x += (element.offsetLeft - element.scrollLeft + element.clientLeft);
    y += (element.offsetTop - element.scrollTop + element.clientTop);
    element = element.offsetParent;
}
{% endhighlight %}

透過 `while` 先計算丟進來的元素跟 `offsetParent` 的距離，如果遇到這個元素有參考 `offsetParent` 我們就用 `element = element.offsetParent` 往外一層一層加上去直到 root 為止。

讓我們再更深入探討，稍早我曾提到排版會受到 `padding`, `margin`, `border` 影響。單純看這些屬性本身並不複雜但是混在一起使用時那就真的是災難。

# Offset 屬性
關於 `offsetLeft` 和 `offsetTop` 剛剛上面有提過它就是回傳元素和 `offsetParent` 之間的距離。

舉例來說當父元素設定了 `position: absolute;` 那麼我們得到的 `offset[Top | Left]` 就會是從該元素的 margin 再加上 offsetParent 元素的 padding

所以 offsetParent 元素 border 以內的 block 都被計算了，我們上面就是接著利用 while 把外層的這個元素做一樣的動作。一直做到 `root`

結果就是我們會得到元素到 root 之間的距離

# 等等！不要忘記 border
由於 border 在 CSS Box Model 中歸屬於 block 內部，所以這段 offset 距離中其實並沒有包含元素的 border 我們可以透過 `client[Top | Left]` 把它加進去，這樣才是我們要的。
用 `document.getElementById("elem").style.borderTopWidth` 這種方法取得值等價。

做到這一步幾乎可以處理所有的狀況，不過還有一個萬惡的捲軸。

# Scrolling 捲軸的部分
如果您設計的 block 可以捲動，一般來說就是父元素限制了寬高且 `overflow: auto;` ，但內容的寬高卻超過了這個尺寸，此時您的內容的確是在父元素內部，但取得的座標卻不是我們要的 - 從看到的元素左上角到 root 左上角為了取得這個座標於是我們要把 scrollTop 扣掉。直接看圖比較快

![]({{ site.url }}/assets/images/tutorials/position_1.png)

最後呢! 回傳 x, y

{% highlight js %}
return { x: x, y: y };
{% endhighlight %}

# 進階範例
最後我們透過這個[進階的範例](http://codepen.io/AndyYou/pen/vEwPGK) 將 `position`, `border`, `margin`, `padding` 混在一起，透過練習請確定您知道每個部分的值是怎麼得到的。

另外我知道這麼多規則很難記住，所以我整理了上面的總結，您可以在忘記的時候對照一下圖片和規則。

# 補充
在 jQuery 中 `$(element).offset()` 我們取得的是元素到 `document` 之間的距離( 自己的 margin 到 document)。
但是當元素設定 `position: fixed` 時取得的值可能不正確。
因為當 document 大於 viewport ，就是內容很長，當你捲到底下的時候 fixed 的元素取到的 offset() 會大於你所期待的值。

此時如果要取 fixed element 相對於 viewport 的位置則用 

{% highlight js %}
$("#el").offset().top - $(document).scrollTop();
{% endhighlight %}

# 參考
[StackOverflow](http://stackoverflow.com/questions/21064101/understanding-offsetwidth-clientwidth-scrollwidth-and-height-respectively)
[Get an Element's Position Using JavaScript](http://www.kirupa.com/html5/get_element_position_using_javascript.htm)




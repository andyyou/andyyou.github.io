---
layout: post
title: '再讀一遍 BEM'
date: 2015-08-11 05:30:00
categories: Program
tags: [css]
---

# 前言

BEM 不是什麼新東西了，會有這一篇純粹是因為之前都只是依樣畫葫蘆的去使用 BEM 看了幾篇 slider 就上沒有認真理解過。當然寫起來就滿頭包。
這一篇花了一點時間歸納總結這個看似簡單卻實用的 css 組織的方法
<!--more-->

# 介紹

在那些比較小型的網站中，如何組織樣式 css 並不是很大的問題，就是進到該專案的目錄直接撰寫一些 css 或者是 sass。接著透過 sass 的 production 設定編譯它變成一個單一的 css 檔案然後從模組中整理取得 css 變成一個整潔的 package

然而當這個專案變得越來越大越來越複雜，如何組織這些程式碼就變成效能的關鍵。重點不只在花多少時間，還包括寫了多少程式碼，以及瀏覽器要載入多少檔案。
當你處於團隊工作的狀況或需要高效能時這些尤其重要

對於那些長時間的專案或者前人留下來的程式碼當然也是很重要

# 方法
其實有很多方法目的都在處理簡化 css 程式碼和組織這些 css 讓協同開發者可以維護。很顯然的在像是 Twitter Facebook 和 Github 這種專案會需要不過其他專案通常隨著需求增加其實很快的也會變成大量的 css

* OOCSS Object Oriented CSS - 透過 css 物件來分離容器和內容
* SMACSS Scalable and Modular Architecture for CSS - 透過樣式指南或說規則來寫 css ，通常真對 css 樣式定義成 5 種分類
* SUITCSS - 結構化 class 名稱和有意義的 `-` 連字號
* ACSS Atomic CSS - 打破樣式組織的方式變成比較瑣碎的規則，像原子的概念一樣用多個 class 組出樣式，這些 class 通常具有不可分割性。

# 為什麼 BEM 超過其他

不管你在專案中選擇哪中方式您都能得到結構化 css 和 ui 的好處。其中一些東西並沒有很嚴格的限制並且具有彈性。那些方法通常非常容易理解而且適用於團隊工作。
與其我們自己提出 BEM 的優點，我們決定讓你看看其他人的看法

* 我選擇 BEM 的理由可以歸納成一點，比起其他方法它比較不會令人產生疑惑，還為我們想用的架構即 OOCSS 提供一個容易識別的術語
* 我很高興開始使用它，最終我得到了一個有秩序的東西。我得到了一個整潔的系統來命名元素。它釋放了在我腦袋中佔據的大量資源，因為即使像是替元素 class 命名這種微不足道的小事卻意外地佔據大量的大腦資源。
* 不像 OOCSS 它並不是為了處理關於 css 全部的模組化，反倒是像是命名空間的概念，透過 class 名稱建立各自獨立的 css 模組，並且不會互相干擾
* BEM 是一個非常有幫助，強大且簡單的命名慣例，讓我們前端的程式碼更好閱讀，好理解，容易維護，容易擴展，更強健也更明確

# BEM Blocks, Elements and Modifiers

你不會感到太意外，BEM 就是這些關鍵處理原則的縮寫 - Block, Element, 和 Modifier。一套嚴格的命名規則可以在另一篇[命名](http://getbem.com/naming)的文章找到

舉下面 Github 網站為例來說明
* Block - 一個獨立的區塊具備自己特有的意義，例如: header, container, menu, checkbox, input
* Element - Block 的一部分並且不具有獨立自己特有的意義，這些元素依賴 Block 的意義。例如: menu item, list item, checkbox caption, header title
* Modifier - Block 或 Element 上的特殊標記，用來改變原來行為或外觀例如: disabled, highlighted, checked, fixed, size big, color yellow

![](http://getbem.com/img/github_captions.jpg)

# 運作的機制與原理

讓我們來看看在頁面上的一個特定元素怎麼透過 BEM 來實作。我們將從 Github 的樣式指南中取出 button

![](http://getbem.com/img/github_buttons.jpg)

通常我們需要一個普通的按鈕針對大多數的出現在界面上的狀況然後其他兩種不同的狀態。因為 BEM 透過 class 選擇器來套用樣式，所以我們可以將樣式套用在任何元素上(例如按鈕的樣式套用到 button, a 甚至 div)。這個時候重點就是採用下面這種規格來命名 `block--modifier--value`

~~~html
<button class="button">
  Normal button
</button>

<button class="button button--state-success">Success button</button>

<button class="button button--state-danger">Danger button</button>
~~~

~~~css
.button {
  display: inline-block;
  border-radius: 3px;
  padding: 7px 12px;
  border: 1px solid #D5D5D5;
  background-image: linear-gradient(#EEE, #DDD);
  font: 700 13px/18px Helvetica, arial;
}

.button--state-success {
  color: #FFF;
  background: #569E3D linear-gradient(#79D858, #569E3D) repeat-x;
  border-color: #4A993E;
}

.button--state-danger {
  color: #900;
}
~~~

# 延伸閱讀

想知道更多範例可以閱讀[Building My Health Skills - Part 3](http://www.bluegg.co.uk/building-my-health-skills-part-3/)

# 優點

* 模組化 - Block 的樣式不應該相依於頁面中其他任何元素，亦為一個 Block 不管放在哪裡都要長得一致，因此永遠不會因為[濫用 css 繼承規則的部分](https://www.phase2technology.com/blog/used-and-abused-css-inheritance-and-our-misuse-of-the-cascade/)就是因為不斷堆疊樣式產生肥大的樣式而降低效能。同時這樣做也具備了讓我們可以輕鬆把一個已經做好的樣式轉換到另外一個專案
* 重複使用性 - 可以用不同的方式來組合各自獨立的 Block 達到重複使用性也減少 code 的數量，也比較好維護。如果你已經有設計指南或公司內部的一些規則那麼它就能協助你有效率的區分，建立，定義 Block
* 結構化 - BEM 讓我們的 css 具有容易理解的特性與結構，簡單易維護，不用再一直記著樣式之間的耦合關係。

# 命名
Phil Karlton 說過在電腦科學的領域只有兩個困難的問題: cache invalidation 和命名

> cache invalidation 快取無效化背後的意義就是什麼時候該把暫存刪掉

這是已知的事實，而正確的樣式指南可以明顯的增加開發的速度，debug 和在既有的程式中實作新功能的速度。很不幸的大部份的 css 沒有任何架構和命名規則。
這導致 css 長久以來都是處於異常難以維護的狀態。

這個 BEM 的方法確保每一個開發者使用相同的慣例，正確的命名讓我們未來在修改維護時相對輕鬆

* block 封裝一個獨立的區塊，上面說過這個區塊需要具備自己的意義。而 block 可以被嵌入其他 block 之中並與其互動，同時維持一致的語意。注意這裡並沒有優先順序或者繼承的概念。也不要使用 DOM 來表示整個區塊

  - 命名：block 的名字可以由英文，數字和連字號組成，目的是將 css class 的命名格式化，當然也可以加入一些簡短的前綴來達到命名空間的效果例如 .block
  - HTML：不要讓侷限 block 套用的元素標籤，任何 DOM 元素只要使用該 class 就能套用 block 的樣式
  - css：
    * 只使用 class name 選擇器
    * 不能直接用 tag 或 id 來選元素
    * 在頁面上不能相依其他 blocks 或 元素

* element 為 block 的一部分並且相依於 block 的意義，舉例來說就像是 list 中的 item 即為一個 element
  - 命名：element 的命名可由英文，數字，連字號(破折號)或底線組成，css class 的格式為 block 的名字加上兩個底線 `__` 再加上 element 名稱 舉例來說 `.block__elem`
  - HTML：任何在 block 內的 DOM node 就可以是一個 element，在給定的 block 內所有的 element 都具有相等的語意
  - css：
    * 同樣只用 class name 選擇器
    * 不用 tag 或 id
    * 在頁面上不能相依於其他 block 或元素
    * `.block_elem { color: #042 };` 正確
    * `.block .block__elem { color: #042; }` 錯誤

* modifier 是 block 或 element 上的特殊註記，用來改變外觀，行為，或狀態舉例來說就是一個 button 可以有正常可點擊的狀態和 disabled 停止使用的狀態，要注意的是這是附加的樣式不能單獨存在一定要搭配原來的 block/element 才行
  - 命名：同樣的 modifier 名稱可以由英文，數字，連字號(破折號)和底線所組成，css class 格式為 block 或 element 名稱加上兩個 dash `.block--mod`, `.block__mod--mod` `.block--color-black`
  - HTML：modifier 是額外的類別名稱讓我們可以加在 block 或 element 來使用，一個 modifier 只能被用在 block 或 element 並且要保留原來的 class，不能用 modifier 取代掉原來的 block / element
    * `<div class="block block--mod">正確</div>`
    * `<div class="block block--size-big block--shadow-yes">正確</div>`
    * `<div class="block--mod">錯誤，取代了原來的類別</div>`
  - css：直接使用 class name 選擇器
    * `.block--hidden { display: none; }`
    * 基於 block 層來調整底下的元素 `.block--mod .block__elem`
    * element 的 modifier `.block__elem--mod`


# 備註 sass 用法

~~~css
.block {
    &__element {
    }
    &--modifier {
    }
}
~~~

# 總結

簡單說來所謂的 B.E.M 就是用 block, element, modifier 三種分類針對 UI 去分析區分歸納，最後協助我們替 class name 命名的一組嚴格規則
block 為一個具有獨立意義的區塊，獨立的意思就是在每個頁面甚至在別的 block 裡面行為外觀都要一致。
element 則是相依於 block 就是 block 的子項目，沒有自己獨立的意義，其意義與使用時要依附在 block 底下
modifier 是一種特殊的註記用來調整改變 block/element 的`外觀(樣板)`，`行為`，`狀態`
明白這三種定義讓我們可以去歸納介面上的 css 該怎麼組織分類進而管理

因為一個 block 不能相依於其他標籤，`獨立不相依`真正的意義就是在任何頁面甚至是嵌入其他 block 外觀行為都要一致，不綁定不限制只能在特定元素使用。舉例來說就像 bootstrap 的 `.btn` 可以在 `a`, `button` 甚至 `div` 都能套用。當 block 之間不會互相污染或被連動影響那麼我們在撰寫修改上就不怕有太多不預期的行為

最後就是該怎麼用：`block/element/modifier` 都只用 class name 選擇器，不依賴 css 的繼承或是像 `.nav > h1` 這樣的方式。好處是可以大幅減少過度被覆寫或不斷被繼承而沒有實際效果的 css，只有單一階 css selector 就能選到元素效能也相對提升
第一個是 block 就用一般英文數字命名，實際上三者都是英數加上 `_` `-` 來完成命名
如果是 element 則用 `__` 加在 block 和 element 中間例如：`block__elem`
最後一個 modifier 則是 `--` 例如：`block--mod` 或 `block__elem--mod`
如果名字較複雜有兩個英文單字則中間用一個 `-` 連結 如 `block__nav-item--disabled`

撰寫的流程為先分析 ui 再照規則寫 css 就好了，再透過下面範例快速掌握

~~~html
<form class="form form--theme-xmas form--simple">
  <input class="form__input" type="text" />
  <input
    class="form__submit form__submit--disabled"
    type="submit" />
</form>
~~~

~~~css
.form { /* ... */ }
.form--theme-xmas { /* ... */ }
.form--simple { /* ... */ }
.form__input { /* ... */ }
.form__submit { /* ... */ }
.form__submit--disabled { /* ... */ }
~~~

# 參考資源

[BEM 101](https://css-tricks.com/bem-101/)
[getbem](http://getbem.com/introduction/)

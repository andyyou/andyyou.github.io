---
title: "[譯] CSS 容器查詢 (CSS Container Query)"
date: 2021-09-26 19:17:51
tags:
  - css
categories: Program
---

作為前端開發人員，在過去六年中，我沒有像現在這樣對 CSS 的新功能感到興奮。容器查詢（Container Query）現在已經可以通過在 Chrome 設定參數啟動支援。感謝那些像是 [Miriam Suzanne](https://css.oddbird.net/rwd/query/contain/) 的人的努力。

我記得看過很多關於支援 CSS 容器查詢的笑話，但最終它成為現實。這篇文章試著了解為什麼我們會需要容器查詢，以及我們如何使用它們簡化開發，更重要的是您可以實現更強大的元件和佈局。

## CSS Media Query 的問題

一個網頁通常由不同的區塊和元件組成，然後我們利用 CSS Media Query 讓它們支援 RWD。這機制沒有什麼問題，但有一些限制。例如我們可以使用 Media Query 在行動裝置呈現元件最小化的版本。

很多時候，RWD 並不是由 Viewport 或螢幕大小來決定的，而應該是由容器的大小來決定。試想下面的範例：

<!-- more -->

![](https://ishadeed.com/assets/cq/problem-1.jpg)

上面我們由一個非常典型的佈局，其中包含卡片元件。它有兩種形式

* 在側邊欄的堆疊格式
* 在右邊的水平排列版本

CSS 有很多方法可以實作，但最常見的是如下面。我們需要先建立一個基本的元件，然後加入變化的版本

```css
.c-article {
}

.c-article > * + * {
  margin-top: 1rem;
}

/* 水平版本 */
@media (min-width: 46rem) {
  .c-article--horizontal {
    display: flex;
    flex-wrap: wrap;
  }
  
  .c-article > * + * {
    margin-top: 0;
  }
  
  .c-article__thumb {
    margin-right: 1rem;
  }
}
```

注意到我們加了 `.c-article-horizontal` 來處理水平版本的元件。如果螢幕或瀏覽器寬（viewport）大於 46rem 那元件就會套用水平版本。

上面的作法還算沒問題，但有點受到限制。我們希望**元件可以直接根據父元素的寬來反應變化**，而不是瀏覽器的 viewport 大小。

思考一下，如果我們想要在右邊主要區塊使用預設的 `.c-article` 會變成怎樣？

那麼，卡片元件將會依據父元素的寬來伸展自己的寬，導致變的太大，如下圖：

![](https://ishadeed.com/assets/cq/problem-1-2.jpg)

這個問題我們就可以使用 CSS 容器查詢來解決。在我們深入之前我們先看一下我們遇到的需求

![](https://ishadeed.com/assets/cq/cq-story.png)

我們希望元件可以根據上層父元素的寬來調整，假如大於 400px 就切換成水平格式。範例程式如下：

```html
<div class="o-grid">
  <div class="o-grid__item">
    <article class="c-article">
      <!-- content -->
    </article>
  </div>
  <div class="o-grid__item">
    <article class="c-article">
      <!-- content -->
    </article>
  </div>
</div>
```

```css
.o-grid__item {
  contain: layout inline-size;
}

.c-article {
  //
}

@container (min-width: 400px) {
  .c-article {
    /* 直接套用水平樣式，而不是預設 .c-article 的樣式 */
  }
}
```


## CSS 容器查詢如何協助我們？

使用 CSS 容器查詢我們可以解決上面的問題並建置一個彈性的元件。這意味著在較窄的父元素中，元件會變成一般垂直排列的版本。父元素如果較寬則會使用水平版本。它們都是基於父元素的寬而不是瀏覽器 viewport。

下面是圖示：

![](https://ishadeed.com/assets/cq/intro.jpg)

右邊紫色寬表示父元素的寬。注意到當父元素變寬，元件會跟著變化。這就是 CSS 容器查詢強大的地方。

## 容器查詢如何運作？

我們現在可以在 Chrome 實驗這個功能。要啟動需要在網址列輸入 `chrome://flags` 搜尋 `container queries` 並啟動。

第一步是加入 `contain` 屬性。由於一個元件會需要基於父元素的寬變動，因此我們需要告訴瀏覽器只需要重繪（Repaint）影響的區域，而不是整個頁面。使用 `contain` 屬性，我們可以讓瀏覽器知道這個前提。

> 筆記：有興趣的深入理解 `contain` 可以參考：
>
> * [前端三十｜03. [CSS] Reflow 及 Repaint 是什麼？](https://medium.com/schaoss-blog/%E5%89%8D%E7%AB%AF%E4%B8%89%E5%8D%81-03-css-reflow-%E5%8F%8A-repaint-%E6%98%AF%E4%BB%80%E9%BA%BC-36293ebcffe7)
> * [提高 CSS 动画性能的正确姿势](https://github.com/chokcoco/iCSS/issues/11)
> * [CSS新特性contain，控制页面的重绘与重排](https://www.cnblogs.com/coco1s/p/14733974.html)
> * [MDN - CSS/contain](https://developer.mozilla.org/en-US/docs/Web/CSS/contain)

設定了 `layout` 的元素即設定了佈局限制，也就是內部（子元素）的樣式變化不會影響外部，外部也不會影響內部。如此可以將渲染元素的數量降低，而不是渲染整個頁面。

`inline-size` 表示只回應父級元素寬的變化，我嘗試過 `block-size` 但目前沒作用，如果有任何錯誤請糾正。

> 注意：這個部分會依據瀏覽器支援的更新有不同的狀態。

```html
<div class="o-grid">
  <div class="o-grid__item">
    <article class="c-article">
      <!-- content -->
    </article>
  </div>
	
  <div class="o-grid__item">
    <article class="c-article">
      <!-- content -->
    </article>
  </div>
</div>
```

```css
.o-grid__item {
	contain: layout inline-size;
}
```

第一步我們定義了 `.o-grid__item` 作為 `.c-article` 元件的父元素容器。下一步加入我們希望支援容器查詢的樣式。

```css
@container (min-width: 400px) {
  .c-article {
    display: flex;
    flex-wrap: wrap;
  }

  /* other CSS.. */
}
```

`@container` 等於是 `.o-grid__item` 元素，然後 `min-width: 400px` 是寬的條件。我們當然可以加入更多樣式。

下面是展示卡片元件行為的影片

<video src="https://ishadeed.com/assets/cq/cq-1.mp4"></video>

完成的樣式如下：

1. 預設卡片元件的樣式
2. 水平版本卡片樣式搭配小縮圖
3. 水平版本卡片樣式搭配大縮圖
4. 如果父元素太大，則樣式調整為類似橫幅的樣式

接著，讓我們更深入 CSS 容器查詢。

## CSS 容器查詢的使用情境

### CSS 容器查詢與 CSS Grid `auto-fit`

針對某些情況如在 CSS Grid 中使用 `auto-fit` 有可能會造成非預期的結果，比如一個元件變的太寬或內容不容易閱讀。

為了讓您能了解上面敘述的情境，下面提供一個 `auto-fit` 和 `auto-fill` 差異的視覺化圖片

![](https://ishadeed.com/assets/cq/auto-fit-fill.png)



注意！當使用 `auto-fit` 時，項目會自動擴展多餘的空間。但在 `auto-fill` 的情況，在欄位數量未達上限的情況是不會挪用當前閒置的空間的。

> 如果您不是很理解 `auto-fit` 和 `auto-fill` 可以參考下面資源：
>
> * [auto-fill ? auto-fit ? 我們不一樣。](https://jhlstudy.blogspot.com/2018/07/grid-layout-auto-fill-auto-fit_8.html)
> * [auto-fill vs. auto-fit](https://gridbyexample.com/examples/example37/)

您現在可能會想說這跟 CSS 容器查詢有什麼關係？那麼如果每一個 Grid Item 就是我們的容器呢，它們都套用了 `auto-fit`，而我們的元件需要基於它們來調整。

```html
<div class="o-grid">
  <div class="o-grid__item">
    <article class="c-article"></article>
  </div>
  <div class="o-grid__item">
    <article class="c-article"></article>
  </div>
  <div class="o-grid__item">
    <article class="c-article"></article>
  </div>
  <div class="o-grid__item">
    <article class="c-article"></article>
  </div>
</div>
```

```css
.o-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}
```

當我們有 4 個元素時如下

![img](https://ishadeed.com/assets/cq/use-case-1-1.png)

當文章數量變少的時候，因為使用了 `auto-fit` 容器會變寬。如下圖所示第一個看起來還可以，但後面兩個太寬了

![](https://ishadeed.com/assets/cq/use-case-1-2.png)

如果每一個文章元件都可以隨著自己的父元素寬變化呢？如此我們便能得到 `auto-fit` 的好處；如果 Grid Item 大於 400px，那麼元件就應該切換成水平版本。

這裡我們可以：

```css
.o-grid__item {
  contain: layout inline-size;
}

@container (min-width: 400px) {
	.c-article {
    display: flex;
    flex-wrap: wrap;
  }
}
```

同時我們希望當只有一個文章元件的時候顯示橫幅效果的樣式

```css
.o-grid__item {
  contain: layout inline-size;
}

@container (min-width: 700px) {
  .c-article {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 350px;
  }

  .card__thumb {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}
```

![](https://ishadeed.com/assets/cq/use-case-1-3.png)

就這樣我們的元件可以對應各種父元素的寬。不覺得這樣很棒？

## 側邊欄和主要區塊

很多時候我們需要調整元件讓它能夠適應寬比較窄的容器例如 `<aside>`。

下圖和此需求完美符合的就是訂閱通知的區塊。當寬變小，我們希望項目堆疊排列，以及當空間足夠的時候則水平排列。

![](https://ishadeed.com/assets/cq/use-case-2.jpg)

如上圖所示，我們的訂閱通知區塊在兩個大區塊都有。

* 側邊欄下方
* 主要區塊下方

如果沒有容器查詢，要完成這個需求會有點複雜，我們須支援不同的 CSS 類別，舉例來說 `.newsletter--stacked`等等。

我知道我們可以[使用 `flex` 的 wrap](https://css-tricks.com/useful-flexbox-technique-alignment-shifting-wrapping/) 來處理，當空間不夠的時候換行，但這還不夠。我們需要控制更多細節例如：

* 隱藏特定元素
* 讓按鈕支援全寬

```css
.newsletter-wrapper {
  contain: layout inline-size;
}

.newsletter {
  /**/
}

.newsletter__title {
  font-size: 1rem;
}

.newsletter__desc {
  display: none;
}

@container (min-width: 600px) {
  .newsletter {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .newsletter__title {
    font-size: 1.5rem;
  }
  
  .newsletter__desc {
    display: block;
  }
}
```

下面的影片就是結果的展示：

<video src="https://ishadeed.com/assets/cq/use-case-newsletter.mp4" />

* [Codepen範例](https://codepen.io/shadeed/pen/poRLxvO?editors=0100) 

## 分頁列

分頁列的部分也是很適用於容器查詢。一開始我們可以有上一頁和下一頁按鈕，隱藏其他按鈕，然後當空間夠時顯示全部分頁。

如下圖：

![](https://ishadeed.com/assets/cq/use-case-3.jpg)

要處理上述狀況，我們需要從預設情形開始處理，然後再處理其他情況。

```css
.wrapper {
  contain: layout inline-size;
}

@container (min-width: 250px) {
  .pagination {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .pagination li:not(:last-child) {
    margin-bottom: 0;
  }
}

@container (min-width: 500px) {
  .pagination {
    justify-content: center;
  }

  .pagination__item:not(.btn) {
    display: block;
  }

  .pagination__item.btn {
    display: none;
  }
}
```

* [Codepen範例](https://codepen.io/shadeed/pen/VwPQORy?editors=0100)

## 個人資料

![](https://ishadeed.com/assets/cq/use-case-4.jpg)

這是另一個非常適合在多個情境使用的例子。小的樣式適合小的 viewport 如側邊欄，大的樣式可以給較寬的情境。

```css
.p-card-wrapper {
  contain: layout inline-size;
}

.p-card {
  /* Default styles */
}

@container (min-width: 450px) {
  .meta {
    display: flex;
    justify-content: center;
    gap: 2rem;
    border-top: 1px solid #e8e8e8;
    background-color: #f9f9f9;
    padding: 1.5rem 1rem;
    margin: 1rem -1rem -1rem;
  }

  /* and other styles */
}
```

通過上面的方式，我們可以不額外使用其他 Media Query 讓元件在不同情境顯示不同資訊或樣式

![](https://ishadeed.com/assets/cq/use-case-4-in-action.jpg)



* [Codepen範例](https://codepen.io/shadeed/pen/NWdYaGY?editors=0100)

### Form 表單元素

我還沒深入表單的使用案例，但能直接想到是關於標籤水平和垂直排列的顯示

![](https://ishadeed.com/assets/cq/use-case-5.jpg)



```css
.form-item {
  contain: layout inline-size;
}

.input-group {
  @container (min-width: 350px) {
    // @container = .input-group
    display: flex;
    align-items: center;
    gap: 1.5rem;

    input {
      flex: 1;
    }
  }
}
```

* [Codepen範例](https://codepen.io/shadeed/pen/XWpEEzR?editors=0100)

## 測試元件

我們已經看了一些 CSS 容器查詢的使用案例，那我們要怎麼測試元件？感謝 CSS 的 `resize` 屬性；

![](https://ishadeed.com/assets/cq/cq-resize.jpg)

```CSS
.parent {
  contain: layout inline-size;
  resize: horizontal;
  overflow: auto;
}
```

從 [CSS-Only Resizable Elements - Bramus Van Damme](https://www.bram.us/2020/05/15/css-only-resizable-elements/) 的文章學習到可以利用 `resize` 來達成測試的目的。

### 能否輕易的使用 DevTool 為容器查詢偵錯？

目前的答案是 - 否。您不能看到任何像是 `@container (min-width: value)` 的東西，但我認為只是時間問題，未來應該會支援。

### 是否能提供不支援的瀏覽器替代的方案？

可以透過特定的方式來支援。下面有兩篇文章您可以參考：

- [Container Query Solutions with CSS Grid and Flexbox ](https://moderncss.dev/container-query-solutions-with-css-grid-and-flexbox/)by Stephanie Eckles
- [Container Queries are actually coming](https://piccalil.li/blog/container-queries-are-actually-coming) by Andy Bell

## 結論

目前來說瀏覽器還沒正式支援，但您已經可以在瀏覽器測試這項功能。作為前端開發者我們應該協助那些功能的開發人員測試。在實驗階段越多的測試，後續的問題將越少。

## 參考資源

* [Say Hello To CSS Container Queries](https://ishadeed.com/article/say-hello-to-css-container-queries/)
* [CSS Container Query Units](https://ishadeed.com/article/container-query-units/)



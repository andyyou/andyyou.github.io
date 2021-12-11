---
title: "[譯] CSS 容器查詢單位"
date: 2021-09-29 09:31:10
tags:
  - css
categories: Program
---

幾天前（2021-09-18）我看到 Miriam Suzanne 的一則關於 CSS Query 單位支援的 [Tweet](https://twitter.com/TerribleMia/status/1436087037040414720) 。這個功能最一開始是由 [Una Kravets](https://github.com/w3c/csswg-drafts/issues/5888) 建議提案。我忍不住試試看它們，看看我們可以從這個功能中得到什麼好處。

本文會嘗試解釋每個單位的運作和我們該如何使用它們，尤其是元件該如何因應父元素的寬。如果您還不知道什麼是 CSS 容器查詢。這裡有一些[範例介紹](https://ishadeed.com/article/say-hello-to-css-container-queries/)和[關於容器查詢如何影響我們的實作](https://andyyou.github.io/2021/09/26/css-container-query/#more)，如果您還沒有任何概念強烈建議您先閱讀該文章。

<!-- more -->

> 提醒：目前 CSS 容器查詢在 Chrome 需要額外設定；到 `chrome://flags` 搜尋 "Enable CSS Container Queries" 並啟動。

## 介紹

在 CSS 中我們有很多單位並可以根據不同的目的使用它們。最常見的是 `px`，`rem`，`em`。如果要說什麼東西跟 CSS容器查詢單位的運作方式最接近，那麼當屬 viewport 單位。

CSS viewport 單位會根據瀏覽器視窗大小調整。很方便，但我們不希望這個單位總是基於瀏覽器的寬高變化。如果我們可以基於容器元素的寬呢？這就是容器查詢單位存在的目的。

為了清楚理解，我們特別標出 viewport 單位和容器查詢單位的差異在下圖：

![](https://ishadeed.com/assets/query-units/intro.jpg)

在左邊 `font-size` 是由 `rem` 和 `vw` 組成也就是字體大小受到瀏覽器寬的影響。在一些情況下，這種方式沒什麼問題，但的確可能造成非預期的問題因為它是跟瀏覽器視窗關聯。

在處理元件中的 `font-size`，`padding`，`margin` 時，容器查詢單位可以節省我們的開發時間。我們不用手動調整，而是使用查詢單位。

想像一下下面的範例：

![](https://ishadeed.com/assets/query-units/intro-2-1.jpg)

我們從卡片元件預設堆疊的樣式開始，到大橫幅樣式。像這樣的一個元件我們可能需要調整下面這些東西

* 縮圖的大小
* 字體大小
* 元素之間的空間

如果沒有查詢單位，我們須手動調整

```css
.card {}

.card__title {
  font-size: 1rem;
}

/* 水平樣式 v1 */
@container (min-width: 400px) {
  .card__title {
    font-size: 1.15rem;
  }
}

/* 水平樣式 v2 */
@container (min-width: 600px) {
    .card__title {
        font-size: 1.25rem;
    }
}

/* 橫幅樣式 */
@container (min-width: 800px) {
    .card__title {
        font-size: 2rem;    
    }
}
```

注意到 `.card__title` 的字體大小在不同查詢條件一直在改變。我們可以利用查詢單位搭配 `clamp()` 避免重複變更 `font-size` 的部分

```css
.card__title {
  font-size: clamp(1rem, 3qw, 2rem);
}
```

上面這種方式我們只要定義一次即可。

![](https://ishadeed.com/assets/query-units/intro-2-2.jpg)

如果您已經熟悉 viewport 單位，那麼容器查詢單位對您來說並不是什麼新東西，只是從基於瀏覽器 viewport 換成容器元素。

* [範例](https://codepen.io/shadeed/pen/ZEyxzRQ?editors=0100)

## 容器查詢單位（Query Units）

現在我們已經對於查詢單位要解決的問題有了概念，是時候來了解單位本身了，根據 [CSS 規範](https://drafts.csswg.org/css-contain-3/#container-lengths)

* Query Width （`qw`）：容器寬的 1%。例如 5qw 等於容器元素寬的 5%。
* Query Height（`qh`）：容器寬的 1%。例如 5qh 會解析為容器元素高的 5%。

還有 `qi`，`qb`，`qmin`，`qmax` 可以參考規範的說明。

> 提醒：目前這些單位還在實驗階段，譯者測試除了 qw 之外其他單位尚未理解具體的計算方式。

## 範例和使用情境

### 卡片元件

除了上面的範例，我們也可以使用查詢單位讓堆疊樣式的卡片加入其他樣式例如依據寬讓字體稍微變大：

![](https://ishadeed.com/assets/query-units/eg-0.jpg)

注意到字體會基於寬稍微變大。如此一來無論卡片放在哪，它都可以運作的非常良好。

* [範例](https://codepen.io/shadeed/pen/yLXKBqN?editors=0100)

### 根據重要性調整 `font-size`

當一個元素比其他元素大的時候，有可能表示它比較重要。例如 `aside` 和 `main` 。`<aside>` 的標題應該要比 `<main>` 的標題小。

![](https://ishadeed.com/assets/query-units/eg-1.jpg)

使用查詢單位的話，我們可以輕鬆完成這個需求：

首先，我們需要將兩個容器定義為 `inline-size` 的容器，

```css
aside,
main {
  container: inline-size;
}
```
> [MDN 文件 - container 屬性](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Container_Queries#container-type)

然後，我們就可以使用查詢單位來設定標題樣式

```css
.section-title {
  font-size: clamp(1.25rem, 3qw, 2rem);
  margin-bottom: clamp(0.5rem, 1.5qw, 1rem);
}
```

* [範例](https://codepen.io/shadeed/pen/oNwqvap?editors=0100)

### 簡歷元件

![](https://ishadeed.com/assets/query-units/eg-2.jpg)

簡歷元件可以在較小的容器例如側邊欄或行動裝置呈現，或則者較大的區塊如頁面的表頭。

我們可以讓元件適應其容器的寬，在這個範例使用者的大頭貼和字體大小都基於寬調整。

```css
.bio {
    container: inline-size;
}

.c-avatar {
    --size: calc(60px + 10qw);
    width: var(--size, 100px);
    height: var(--size, 100px);
    margin-bottom: clamp(0.5rem, 3qmin, 2rem);
}
```

* [範例](https://codepen.io/shadeed/pen/wvemwNb?editors=0100)

### 計數標題

有些情況我們需要顯示計數數量，這種情況也非常適合使用查詢單位。

![](https://ishadeed.com/assets/query-units/eg-3.jpg)

```css
.article-body {
  counter-reset: heading;
}

h2 {
  container: inline-size;
  font-size: clamp(1.25rem, 3qw, 2rem);
  margin-bottom: clamp(0.5rem, 1.5qw, 1rem);
}

h2:before {
  --size: calc(1.25rem + 3qw);
  content: counter(heading);
  counter-increment: heading;
  width: var(--size);
  height: var(--size);
  font-size: calc(0.85rem + 1qw);
  margin-right: calc(var(--size) / 4);
}
```

* [範例](https://codepen.io/shadeed/pen/xxrWxxp?editors=0100)

### 動態間距

過去，我們可以利用 viewport 單位來為 Grid 建立動態間距例如

```css
.wrapper {
    gap: calc(1rem + 2vw);
}
```

那如果是特定容器中的元件呢？使用 viewport 單位情況會比較複雜，如果使用查詢單位就相對適合。

![](https://ishadeed.com/assets/query-units/eg-4.jpg)

```css
.card {
    display: flex;
    flex-direction: column;
    gap: calc(0.5rem + 1qmin); // qi qb 較小的那個
}
```

### 如果使用查詢單位但沒有設定 `container` 會如何？

瀏覽器會使用 viewport 單位的方式處理，例如下面範例：

```css
h2 {
    font-size: clamp(1.25rem, 3qw, 2rem);
}
```

如果沒有設定容器，那麼 `<h2>` 的 `3qw` 會變成 viewport 寬的 3%。因此如果使用查詢單位請注意要設定容器。

## 總結

暫時您可以先知道將來有這些單位即可，這些單位還不適合直接使用在正式環境上。

## 參考

* [CSS Container Query Units](https://ishadeed.com/article/container-query-units/)
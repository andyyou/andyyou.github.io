---
layout: post
title: '[譯] CSS 載入的未來'
date: 2016-02-18 00:30:00
categories: css
---

這週閱讀到這篇有意思的[文章](https://jakearchibald.com/2016/link-in-body/)，於是便動手
寫下間單的翻譯，如果有理解錯誤的地方歡迎指教。

Chrome 正在試圖改變當 `<link rel="stylesheet">` 寫在 `<body>` 的行為，從 `blink-dev` 的文章並不能很清楚的知道其優點。
所以這篇文章想要深入的介紹這點。

> blink 是 Chrome 和 Opera 渲染引擎，而 blink-dev 是其開發社群

# 目前的 CSS 載入機制

{% highlight html %}
<head>
  <link rel="stylesheet" href="/all-of-my-styles.css">
</head>
<body>
  …content…
</body>
{% endhighlight %}

當 CSS 段落在渲染時，會讓使用者瞪著白白的頁面直到 `all-of-my-styles.css` 完全下載完畢。

通常我們會把站內所有的 CSS 封裝成較少，可能只有一兩個資源檔，但這同時也意味著使用者需要下載大量
的樣式設定(CSS Rules)卻沒有在該頁面使用。因為一個網站包含著各種不同的頁面與元件，而這些東西需要
套用不同的樣式規則，如果因此分拆成許多檔案而產生大量的請求 `request` 這在 `HTTP/1` 是非常耗效能的。

不過在 `SPDY` 和 `HTTP/2` 卻不是這樣，傳輸許多分散的小資源只會增加一點點的開銷，並且這些東西
可以個別暫存(cached)。

{% highlight html %}
<head>
  <link rel="stylesheet" href="/site-header.css">
  <link rel="stylesheet" href="/article.css">
  <link rel="stylesheet" href="/comment.css">
  <link rel="stylesheet" href="/about-me.css">
  <link rel="stylesheet" href="/site-footer.css">
</head>
<body>
  content
</body>
{% endhighlight %}

這樣一來就解決了許多問題，我們可以拆成很多小檔案個別載入，不過也意味著當我們在 `<head>` 下 `<link>` 時，
得要知道這些頁面各自需要哪些資源。
另外，瀏覽器在開始輸出之前，仍然得下載所有的 CSS。如果出現一個下載比較慢的 CSS 例如 `/site-footer.css`
將會造成渲染的東西延遲。請觀察[範例](https://jakearchibald-demos.herokuapp.com/progressive-css/head.html)。

# 現階段載入 CSS 較先進的機制

{% highlight html %}
<head>
  <script>
    // https://github.com/filamentgroup/loadCSS
    !function(e){"use strict"
    var n=function(n,t,o){function i(e){return f.body?e():void setTimeout(function(){i(e)})}var d,r,a,l,f=e.document,s=f.createElement("link"),u=o||"all"
    return t?d=t:(r=(f.body||f.getElementsByTagName("head")[0]).childNodes,d=r[r.length-1]),a=f.styleSheets,s.rel="stylesheet",s.href=n,s.media="only x",i(function(){d.parentNode.insertBefore(s,t?d:d.nextSibling)}),l=function(e){for(var n=s.href,t=a.length;t--;)if(a[t].href===n)return e()
    setTimeout(function(){l(e)})},s.addEventListener&&s.addEventListener("load",function(){this.media=u}),s.onloadcssdefined=l,l(function(){s.media!==u&&(s.media=u)}),s}
    "undefined"!=typeof exports?exports.loadCSS=n:e.loadCSS=n}("undefined"!=typeof global?global:this)
  </script>
  <style>
    /* The styles for the site header, plus: */
    .main-article,
    .comments,
    .about-me,
    footer {
      display: none;
    }
  </style>
  <script>
    loadCSS("/the-rest-of-the-styles.css");
  </script>
</head>
<body>
</body>
{% endhighlight %}


上面程式碼，我們有一些內嵌的樣式(inline style)來讓我們可以快速的渲染初始化的樣式，接著把那些
還沒取得樣式的部分隱藏起來，然後開始透過 Javascript `非同步`下載剩餘的樣式。這些`剩餘的 CSS`
會覆寫掉在 `.main-article` 和其他選擇器內的 `display: none`。

這種第一次先快速的初始化渲染，然後持續匯入的方法是許多效能專家所推薦的。

* [為 CSS 傳送進行最佳化處理](https://developers.google.com/speed/docs/insights/OptimizeCSSDelivery)
* [How we make RWD sites load fast as heck](https://www.filamentgroup.com/lab/performance-rwd.html)
* [Optimizing the Critical Rendering Path](http://www.lukew.com/ff/entry.asp?1756)

[看看範例](https://jakearchibald-demos.herokuapp.com/progressive-css/two-phase.html)

原作者實作了[wiki-offline](https://wiki-offline.jakearchibald.com/)並將狀況紀錄如下圖

![]({{ site.url }}/assets/images/tutorials/css_loading_new_way_01.png)

上面這張圖片是在 3G 的環境下測試。

不過這樣的方法還是有些不足的地方:

### 需要一個輕量的 Javascript 函式庫

不幸的，因為 WebKit 的實作。當 `<link rel="stylesheet">` 一被加到頁面時，WebKit 會阻塞
渲染(render)，直到樣式都被載入，即使這個樣式是透過 Javascript 加入的。

在 Firefox 和 IE/Edge，透過 JS 加入的樣式完全是非同步載入。Chrome 當前穩定版本然仍是遵循
WebKit 的行為，不過在 `Canary` 已經跟 Firefox/Edge 一樣了。

### 載入流程被限制在兩個階段(Inline css and A css file)

根據上面的模式，內嵌 CSS(inline CSS) 透過 `display: none` 隱藏尚未套用樣式的內容，然後非同步
得載入 CSS 之後呈現內容。如果您需要增加多個 CSS 檔案那麼結果可能就是內容不按順序出現

[檢視範例](https://jakearchibald-demos.herokuapp.com/progressive-css/naive.html)

![]({{ site.url }}/assets/images/tutorials/css_loading_new_way_02.png)

如果內容還在異動的過程結果周圍的廣告就先出現這通常會讓使用者覺得不開心就關閉你的網站了。

因此被限制在只有兩個載入階段，你必須要決定哪些是第一次渲染時就要出現，哪些是比較晚的。
當然，你希望上方的區塊越快顯示越好，不過所謂"上方的區塊"取決於 `viewport` 可視區域的大小。
最後你可能決定定義一個尺寸範圍套用在所有人身上。

如果你想讓事情更加複雜，當然你可以選擇[客製 CSS 相依屬性來建立 CSS 之間渲染的相依性](https://jakearchibald.com/2016/css-loading-with-custom-props/)

### 更簡單，更好的方式

{% highlight html %}
<head>
</head>
<body>
  <!-- HTTP/2 push this resource, or inline it, whichever's faster -->
  <link rel="stylesheet" href="/site-header.css">
  <header>…</header>

  <link rel="stylesheet" href="/article.css">
  <main>…</main>

  <link rel="stylesheet" href="/comment.css">
  <section class="comments">…</section>

  <link rel="stylesheet" href="/about-me.css">
  <section class="about-me">…</section>

  <link rel="stylesheet" href="/site-footer.css">
  <footer>…</footer>
</body>
{% endhighlight %}

這個概念是透過每一個 `<link rel="stylesheet">` 在其下載樣式時去阻塞跟在後面的內容，但允許
前面的內容先開始渲染。
樣式表(CSS)本身的載入機制是平行的，但是套用樣式卻是要照順序的。這讓 `<link rel="stylesheet">` 的行為類似為 `<script>`

假設說 `site-header`, `article`, `footer` 的樣式已經載完了，但剩下的還沒，其行為如下：

* Header: 已輸出
* Article: 已輸出
* Comments: 未呈現，在該標籤之前地 CSS 還沒載完(`./comment.css`)
* About me: 未呈現，在該標籤之前地 CSS 還沒載完(`./comment.css`)
* Footer: 未呈現，在該標籤之前地 CSS 還沒載完(`./comment.css`)，即使自己的 CSS 已經載完了

這讓我們可以照順序輸出頁面。您甚至不需要決定哪些是"上面的區塊"，只要在元件之前匯入元件需要的 CSS 即可。

不過你還是需要`注意當使用內容決定佈局(layout system)`，例如 table, flexbox 時，在載入期間
應避免內容異動。
這不是現在才產生的問題了，只是在`逐步顯示`這種機制之下更常遇到。如果你看不懂這段在描述什麼看一下下面的影片就知道了。

<iframe width="560" height="315" src="https://www.youtube.com/embed/vPryjyFP5FM" frameborder="0" allowfullscreen></iframe>

意思是說雖然 `flexbox` 已經很不錯了，但 Grid 還是更推薦的 Layout system。

### Chrome 的改變

[HTML規範](https://html.spec.whatwg.org/multipage/semantics.html#the-link-element)並不涵蓋網頁渲染時是否應該或該如何被 CSS 阻塞，
並且也不鼓勵把 `<link rel="stylesheet">` 寫在 `body` 中，不過所有瀏覽器都允許這麼做。

當然他們也都各自使用了自己的方式處理在 body 中的 `<link>`

* Chrome & Safari: 一旦發現 `<link rel="stylesheet">` 就停止渲染(render)，直到發現的樣式被載入完畢。往往導致在 `<link>` 上面還未渲染的內容被卡住。
* Firefox: `<link rel="stylesheet">` 在 `<head>` 中會阻塞渲染直到所有發現的樣式都被載入完畢。在 body 中的話不會阻塞渲染，除非 head 裡已經有樣式阻塞住
這可能會導致 FOUC (Flash of unstyled content) ，就是畫面先以預設的樣子呈現後閃一下才套上樣式。
* IE/Edge: 中斷分析器直到樣式載完，不過允許 `<link>` 上面的內容渲染(render)。

我們偏好像 IE/Edge 的行為，所以 Chrome 將會跟隨這樣的機制。目前 Chrome/Safari 的行為就只是
會卡比較久的時間，Firefox 的行為相對較複雜一些，不過有一些小技巧。

### Firefox 處理方式

因為 Firefox 不會因為 body 中的 `<link>` 阻塞渲染。我們需要一點小技巧來避免 FOUC。幸虧有個非常簡單的方式就是透過 `<script>` 中斷解析器
讓他等一等待處理狀態的樣式

{% highlight html %}
<link rel="stylesheet" href="/article.css"><script> </script>
<main>…</main>
{% endhighlight %}

這個標籤不能完全沒有內容，所以我們需要留一個"空白"。

### 實際執行的結果

[查閱範例](https://jakearchibald-demos.herokuapp.com/progressive-css/)

Firefox 和 Edge/IE 將會循序的載入輸出，而 Chrome 和 Safari 則是先看到空白的頁面一陣子直到
CSS 全部載完。目前 Chrome/Safari 的行為比起把樣式連結放在 `<head>` 也沒有差到哪去，所以
現在就可以開始使用這個方式，很快的 Chrome 將會往 Edge 的機制修正，這麼一來渲染會更加迅速。

以上就是一個簡單的技巧協助我們加快速度並逐步載入 CSS

---
layout: post
title: '[譯] Houdini: 你還沒聽說！這可能是 CSS 開發中最令人興奮的事！'
date: 2016-03-31 17:30:00
categories: Program
tags: [css]
---

> 其實...我想說這可能是最令我感到興奮..但又害怕頭痛的功能... - [原文連結](https://www.smashingmagazine.com/2016/03/houdini-maybe-the-most-exciting-development-in-css-youve-never-heard-of/)

你曾經想要使用某個 CSS 的新功能，但是最後卻因為這個功能瀏覽器還未全面支援而放棄了嗎？甚至更糟糕的狀況，瀏覽器已經支援了但卻充滿問題。我敢打賭這些情況你肯定遇過了。如果上面這種情形你曾經遇過，那麼你是應該關心一下 `Houdini`。

<!--more-->

`Houdini` 是 w3c 新的任務團隊，他們的終極目標就是解決上面這些問題 - 讓瀏覽器更迅速廣泛的支援 CSS 新特性。這個計畫試圖透過提供一系列新的 API，也是第一次讓開發者可以擴充 CSS 本身的能力。透過 hook 的方式讓我們可以在瀏覽器渲染的過程動點手腳。

> Hooks 簡易說明：Hooks 英文翻譯為鉤子，在程式術語中所表達的是在程式特定位置埋入一段預留的程式碼，用來呼叫其他對應的程式碼。可以大略想成在某個片段先空出一個位置，這個位置可以在事後再放入動作，不放也沒關係。

不過具體來說這到底是什麼意思？這麼做真的好嗎？還有這樣做到底能在我們的開發過程提供些什麼幫助。

在這篇文章，我將試著回答這些問題。不過在這之前我們必須要先搞清楚，到底我們在今時今日遇到什麼問題，為什麼我們需要做這些改變。接下來我們將會更具體的說明 `Houdini` 是什麼東西和這傢伙會怎麼解決這些問題，並且列出目前開發中一些令人興奮的功能。最後我將提供一些比較具體實例的說明。

# Houdini 試圖要解決什麼問題？

每一次當我寫了一篇文章來介紹一些 CSS 的新功能時總是會有人留言像是 `這實在太棒了! 不過我們可能要再等個 10 年才能使用` 我能體會這種毫無建設性的留言....從過往的歷史來看，這的確需要花上好幾年的時間從功能提案到廣泛採用。而其中最關鍵的原因是唯一讓 CSS 增加新功能的方式就是透過下面這個標準流程。

![](https://media-mediatemple.netdna-ssl.com/wp-content/uploads/2016/03/01-standards-process-preview-opt.png)

因為瀏覽器本身牽涉太廣泛，對於這樣的流程我本身沒有任何反對意見。當然我們也知道這會耗掉很長的時間。舉例來說 `flexbox` 首次提案是在 2009 年，而時至今日我們還是見到開發者在抱怨支援的瀏覽器不夠。這個問題正緩慢的解決中，因為幾乎所有的瀏覽器都會自動更新，不過即使是現代瀏覽器(Modern Browser)在功能提案到實際能使用該功能還是有一段挺長的時間。

有趣的是並不是所有 Web 領域的東西都處在這樣的情況，看看最近的 Javascript，為什麼 Javascript 可以發展如此迅速

![](https://media-mediatemple.netdna-ssl.com/wp-content/uploads/2016/03/02-polyfill-process-preview-opt.png)

從上圖這種流程，構想提案到實際用在產品上有時候只需要幾天的時間。例如：我已經在產品上使用了 `async/await` 的功能。這個功能甚至還沒有一個瀏覽器支援。

你還可以看到兩個社群截然不同的狀況，在 Javascript 社群你可以聽到人們開始在抱怨更新速度太快，而在 CSS 社群則多是看到一堆人在抱怨學那麼多東西沒用，還需要非常久的時間才能使用。

# 那為什麼我們不可以使用 CSS Polyfills？

看完 Javascript 的流程第一個直覺的想法是那我們也來為 CSS 提供 Polyfills，聽起來可能蠻可行的，如果有 Polyfills 那麼 CSS 也可以像 Javascript 一樣快速的演進，不是嗎？可惜的是，這並不像想的那麼容易，用舊有技術實現新的功能或 API 在 CSS 中異常的困難，在大部分的情況下會整個犧牲掉效能。

Javascript 是一個動態語言，意味著我們相對容易用 Javascript 替 Javascript 補上 Polyfills。對 CSS 來說我們相對很少使用 CSS 來做 Polyfills。一般頂多就是使用轉譯器來產生 CSS 例如 PostCSS 就是這樣的東西。
如果你想直接對 DOM 結構或者元素的 Layout，位置加上 Polyfills，那麼我們就必須要在客戶端執行對應的邏輯程式。

不幸的是瀏覽器對於這方面並不提供任何簡單的方式。

下圖我們簡單的歸納瀏覽器從收到 HTML 文件到顯示在螢幕上概略的流程。藍色區塊就是 Javascript 能夠控制結果的關鍵點

![](https://media-mediatemple.netdna-ssl.com/wp-content/uploads/2016/03/03-rendering-process-preview-opt.png)

在上圖中我們認知到身為一個開發者，你對於瀏覽器解析 HTML 和 CSS 轉換成 DOM, CSSOM 的過程幾乎沒有控制權，尤其在瀏覽器對元素佈局以及渲染方面。唯一在這個過程中我們能夠掌握的就是 DOM 的存取，是不是該換 CSSOM 做些開放了。不過這邊先提一下 Houdini 網站上提到的`改良 CSSOM`的部分：確認跨瀏覽器行為不一致以及`缺少關鍵功能`的問題。

關於缺少關鍵功能部分，舉例來說，瀏覽器中的 CSSOM 並不會顯示跨站存取的樣式規則，而且會直接忽略那些它看不懂的樣式，這也意味著如果你想增加一個瀏覽器未支援的功能，你是不可以使用 CSSOM。反而要用 DOM 去找到 `<style>` 和 `<link rel="stylesheet">` 自己取得 CSS 然後解析再把它們補回 DOM。當然，更新 DOM 通常也表示瀏覽器必須要重新執行整個流程。

![](https://media-mediatemple.netdna-ssl.com/wp-content/uploads/2016/03/04-rendering-process-polyfilled-preview-opt.png)

雖然整個流程重新渲染看起來並不會造成非常嚴重的效能問題，甚至這可能已經是常出現的情況，不過如果我們需要處理像是 `scoll` `window resize` `mouse` `keyboard` 這些事件，大量的更新次數可能就會很明顯地看出變慢了。更慘的是當你發現大部分的 CSS Polyfills 都有它們自己的解析方式和套用邏輯，而解析和套用是非常複雜的事情，導致這些 polyfills 通常不是很肥就是容易出錯。

總結上述所說的就是，現階段如果你想要瀏覽器對於`呈現`做些不同的行為那麼你就必須要靠 DOM 去調整。

# 但，為什麼我需要修改瀏覽器內部的渲染引擎？

對我來說，這絕對是能夠回答整篇文章最關鍵的問題。如果你已經看到這邊，請仔細的閱讀這個部分!

看到這邊我很肯定有一部份的人會想：我根本不需要這個。我只是建置一個一般的網頁，我不會想去對瀏覽器內部動手腳。如果你是這麼想的，那麼我強烈建議你先停一下，去檢查過去你已經用在開發中的那些技術。對瀏覽器套用樣式的流程取得一些存取權限和 hooks 並不是只為了創造一些很屌的火力展示 - 這主要是希望給開發者和框架作者有更多的`解決方案`，`功能`去完成下面這兩件事：

* 統一跨瀏覽器樣式的不同行為
* 開發新功能或者解決相容性問題 - 補上新功能的 polyfills ，讓我們可以快點使用到這些新技術

如果你曾使用過像是 jQuery 這類的函式庫，你已經受惠於這種功能！事實上，這也是現今大多數前端框架或函式庫的主要賣點。Github 上 5 個流行的 Javascript 專案 - AngularJS, D3, jQuery, React, Ember 都處理了很多跨瀏覽器不同行為的問題。

不過上面說的是 Javascript 的部分，現在讓我們來想想關於 CSS 和所有跨瀏覽器的問題。即便是最流行的 CSS Framework 像 Bootstrap，Fundation 也聲明跨瀏覽器相容性的問題他們並沒有完全處理 - 只是避開那些問題。跨瀏覽器 CSS 的 bug 不只在過去存在，即使是今天也還存在像是 [flexbox` 的問題](https://github.com/philipwalton/flexbugs)。

想像一下如果任何 CSS 的樣式規則可以確保在任何瀏覽器有著一致的行為那我們的開發生涯該是多麽美好。再進一步如果任何你從 Blog，Conferences 或 Meetup 得知的新功能像是 [CSS Grids](https://drafts.csswg.org/css-grid/), [CSS Sanp points](https://drafts.csswg.org/css-snappoints/), [Sticky positioning](https://drafts.csswg.org/css-position-3/#sticky-pos) 我們馬上就能夠在今天開始使用，而你需要做的就是從 Github 下載程式碼。

這就是 Houdini 的理想。這個未來正是該工作團隊試圖實現的目標。你說你根本不想去撰寫那些 polyfills 也沒關係，但你大概會想使用吧！？畢竟只要有人寫出 polyfills 我們就可以從中得到幫助。

# 那麼 Houdini 現階段有什麼功能正在開發？

如同上面提到的，開發者對於瀏覽器渲染的過程沒有太多存取控制的機會，現階段只能從 DOM 下手。

為了解決這個問題，Houdini 團隊將提倡了一些新的規範，關於在渲染流程中賦予開發者其他部分的控制權。下圖顯示新的規範中開發者可以操作的部分。注意規範中灰色區塊是已規劃但還未被寫入規範的。

![](https://media-mediatemple.netdna-ssl.com/wp-content/uploads/2016/03/05-spec-coverage-preview-opt.png)

下面我們將簡介每個新的規範並且說明其功能。完整的清單請查閱[Houdini Drafts](https://github.com/w3c/css-houdini-drafts)

# CSS Parser API

[CSS Parser API](https://drafts.css-houdini.org/css-parser-api/) 目前還未被寫入規範，意思是這邊提到任何內容極有可能會改變。不過基本來說它的概念是讓開發者可以擴充 CSS Parser ，提供新的語法結構，舉例來說：新的 media 語法，新的偽類 pseudo-classes 等等

一旦解析器知道關於新的結構，它就可以正確地將其運用在 CSSOM 上。

# CSS 屬性與值的 API

CSS 已經具有客製屬性的功能，並且在之前的[文章](http://philipwalton.com/articles/why-im-excited-about-native-css-variables/)說明過對此感到非常興奮。[CSS 屬性與值的 API](https://drafts.css-houdini.org/css-properties-values-api/)除了自訂屬性將更進一步使其具備型別。

關於加入型別有非常多的好處，不過大體來說最大的賣點就是讓開發者可以客製 transition 和 animate。這在今天是辦不到的。不懂！？看看下面的例子

~~~css
body {
  --primary-theme-color: tomato;
  transition: --primary-theme-color 1s ease-in-out;
}
body.night-theme {
  --primary-theme-color: darkred;
}
~~~

上面的例子如果 `night-theme` class 被加到 `<body>` 元素那麼該頁面有參考 `--primary-theme-color` 屬性的值將會慢慢的從 `tomato` 轉換到 `darkred`。如下範例

~~~css
p {
  color: var(--primary-theme-color);
}
~~~

在現階段，如果你要完成這個功能你得為每一個元素撰寫 transition，因為現在過渡效果是跟在元素上並不是跟著屬性。另一個有趣的功能就是`註冊 hook`，提供開發者一個方式來修改自訂屬性的最終值，用在 polyfills 方面這可能是非常實用的。

# CSS Typed OM

[CSS Typed OM](https://drafts.css-houdini.org/css-typed-om/)大概就是目前 CSSOM 第二版的概念。目的是解決當前 CSS 模型的問題，也包含 CSS Parser API 和 CSS 屬性和值 API 追加的新功能。

Typed OM 另外一個主要的目標是改善效能。將 CSSOM 目前使用的字串值換成具意義型別，因此在Javascript 操作的表現會得到顯著效能的提升。

# CSS Layout API

[CSS Layout API](https://drafts.css-houdini.org/css-layout-api/)提供開發者撰寫自己的佈局模組。透過`佈局模組`意味著任何東西可以被傳入 `display` 屬性。這將是第一次開發者有辦法透過原生的方式改變佈局提供像是 `display: flex` 和 `display: table` 這樣不同的模組。

例如 `Masonry Layout 流瀑式版型` 這樣複雜的佈局方式在今天是不可能單單只透過 CSS 就完成的。雖然它的效果令人驚艷但可惜的是通常都有效能相關的問題，在低階的裝置上尤其明顯。

CSS Layout API 讓開發者透過 `registerLayout` 的方法，允許註冊一個 layout 的名稱，然後用一個 Javascript class 來組織邏輯。下面就是一個簡單的範例

~~~js
registerLayout('masonry', class {
  static get inputProperties() {
    return ['width', 'height']
  }
  static get childrenInputProperties() {
    return ['x', 'y', 'position']
  }
  layout(children, constraintSpace, styleMap, breakToken) {
    // Layout logic goes here.
  }
}
~~~

如果你感受不到上面範例的意義，也不用擔心。最主要的是說你可以像下面範例這樣使用，你只要找到別人寫好的例如 `masonry.js` 接著在 CSS 中使用就好了。

~~~css
body {
  display: layout('masonry');
}
~~~

# CSS Paint API

`CSS Paint API` 和 `Layout API` 非常類似，不過它提供一個 `registerPaint` 方法。開發者可以在 CSS 中使用 `paint()` 函式，透過傳入的名稱產生一個 CSS 圖片。這邊有個簡單的範例就是繪製一個有顏色的圓形

~~~js
registerPaint('circle', class {
  static get inputProperties() { return ['--circle-color']; }
  paint(ctx, geom, properties) {
    // Change the fill color.
    const color = properties.get('--circle-color');
    ctx.fillStyle = color;
    // Determine the center point and radius.
    const x = geom.width / 2;
    const y = geom.height / 2;
    const radius = Math.min(x, y);
    // Draw the circle \o/
    ctx.beginPath();
    ctx.arc(x, y, radius, 0, 2 * Math.PI, false);
    ctx.fill();
  }
});
~~~

CSS 的用法如下

~~~css
.bubble {
  --circle-color: blue;
  background-image: paint('circle');
}
~~~

現在套用 `.bubble` 的元素會產生一個藍色圓形的背景圖。這個圓會跟元素尺寸一樣且置中。

# Worklets

關於上面列出的規範和一些範例你可能會想知道你該把它們放在哪裏？答案是 [worklet](https://drafts.css-houdini.org/worklets/) scripts。`worklets` 類似於 `Web workers`，它允許我們匯入 script 檔案，可以在不同的階段執行 Javascript 程式碼，並且它不相依於主要執行緒。`worklet script` 為了確保效能會嚴格的限制能夠操作的型別。

# 捲軸效果和動畫

雖然針對捲軸效果(scrolling)和動畫方面還沒有官方的規範，不過這是 Houdini 其中一個備受期待的功能。最終 API 會允許開發者將程式邏輯放在一個`負責組織執行的 worklet`，而不是主要執行緒上。這個東西會支援修改一些 DOM 的屬性。透過這種方式就可以只更新特定屬性，而不是全部重新渲染。

如此一來開發者就可以容易的做出高效能的 scrolling 和動畫類型的應用，例如 `sticky scroll header` 或是平行捲動 `parallax` 的效果。你可以在[使用案例](https://github.com/w3c/css-houdini-drafts/blob/master/composited-scrolling-and-animation/UseCases.md)中得知這些 API 試圖要解決的問題。

儘管還沒有正式的官方規範，實驗性的開發已經在 Chrome 上展開了。事實上 Chrome 團隊目前正在使用這些主要的 API 實作 `CSS Snap point` 和 `Sticky positioning`。這是很驚人的，因為這意味著 Houdini API 有足夠的效能讓 Chrome 新功能採用。
如果你仍然擔心 Houdini 效能的問題，你可以看看下面這個[影片](https://www.youtube.com/watch?v=EUlIxr8mk7s)以及[原始碼](https://github.com/GoogleChrome/houdini-samples/tree/master/twitter-header)

# 那我現在可以做些什麼？

如同上面提到的，我想每一個網站開發者都應該關注 Houdini。這將是讓我們開發生活變得更美好的未來。即使你不會直接使用 Houdini 規範內定義的東西，你幾乎會用到架構在它之上的許多東西。雖然這個未來還不會立即到來，但它可能比我們想的還要快到來。

所有主流瀏覽器的廠商在 2016 年初已經在雪梨有場面對面的會議交流，對於 Houdini 的建置與開發沒有太多異議。可以說 Houdini 是否會成為下一件大事已經不是個問題，問題應該是何時導入比較恰當。

當然瀏覽器供應商對於建置這些新功能自有一些優先順序，通常會根據那些使用者迫切需要的功能為優先。所以如果你關心這個關於樣式與佈局的擴展功能，你希望可以不用等待瀏覽器開發商經過漫長的開發程序，就使用 CSS 的新功能請告訴那些相關的開發成員你需要這個功能。

另一方面你可以透過提供一些真實世界的使用案例，就是那些你想做的功能但現今技術卻異常難實現。已經有一些[案例](https://github.com/w3c/css-houdini-drafts)列在這個 Github 專案。你可以透過發個 `Pull Request` 來加入協作。

Houdini 開發團隊真的希望能夠從網頁開發人員中取得真正需求與問題，使這些功能更加完善周到。因為通常參與規格書制訂的工程師只專注在瀏覽器身上，他們並不知道應用程式開發者的痛點。全靠我們來告訴他們了。

# 參考資源

* [CSS-TAG Houdini Editor Drafts](https://drafts.css-houdini.org/) - W3C 最新草稿
* [CSS-TAG Houdini Task Force Specifications](https://github.com/w3c/css-houdini-drafts) - 官方 Github 專案
* [Houdini Samples](https://github.com/GoogleChrome/houdini-samples) - 實驗範例
* [原文出處](https://www.smashingmagazine.com/2016/03/houdini-maybe-the-most-exciting-development-in-css-youve-never-heard-of/#comment-1288360)

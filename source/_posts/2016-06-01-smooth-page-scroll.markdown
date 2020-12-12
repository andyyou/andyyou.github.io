---
layout: post
title: '[譯] scroll-behavior 滑順的捲動效果'
data: 2016-06-03 12:00:00
categories: Program
tags: [javascript]
---

眾所皆知 HTML 錨點(anchor link)透過給定標籤 `id` 屬性跳到頁面上特定位置的功能。不過這個效果感覺上就像是閃一下就切換到該位置。
為了使用體驗上的感覺有時候網站會設計一種`平滑捲動到該位置`的效果。

在過去這樣的效果通常會透過 jQuery 來達成，但有時候一些簡單的頁面為了達成這個功能就需要載入一堆函式庫或框架這未免有點矯枉過正。
最新的 Javascript 提供了一個更有效率，加強原生 `window.scrollTo` 的方式。

<!--more-->

一個標準的錨點已經是一個被廣泛使用的基本技巧: 透過這種方式就算新的 `smooth scroll` 滑順捲動的語法不被支援就的方法仍然會運作，就是跳到該位置。

```html
<a href="#dest">Click to somewhere</a>
...
<p id="dest">This is the target</p>
```

> 要注意的是頁面內容要超過可視區域就是至少 scroll bar 要出現，如果瀏覽器已經在畫面上顯示出兩者且沒得捲就沒有效果。因此我們需要在連結和錨點之間補上一些內容。

# 兩種方式

由於 `Smooth Scrolling API` 有兩種，一種是 CSS, 一種則是 Javascript。也因此造成混亂的原因是部分瀏覽器有支援上不一致。

CSS 的方式非常簡單，只要在該元素設定 `scroll-behavior: smooth;`

```css
body {
  scroll-behavior: smooth;
}
```

> 注意是 `behavior` 而不是 `behaviour`

這個方式非常方便不過目前只有 Firefox 支援，[查閱 Can I Use](http://caniuse.com/#search=scroll-behavior)。

# Javascript

然後是 Javascript 的方式

```js
var anchor = document.querySelector('a[href="#dest"]')
var target = document.getElementById('dest')
anchor.addEventListener('click', function (e) {
  if (window.scrollTo) {
    e.preventDefault()
    window.scrollTo({'behavior': 'smooth', 'top': target.offsetTop})
  }
})
```

注意到 `window.scrollTo` 跟現有的 Javascript 在參數上有些不同，如果你直接用在 Chrome 下，您就會出現參數數量不對的錯誤，所以實務上要應用還是需要額外做些處理。

另外這種方式有一個缺點，那就是我們不能自訂 `timing function`。

# 延伸

上面的 script 已經可以讓單一的錨點正常的運作，不過這種方式有點面對大量連結的時候有點麻煩。假如我們在這個頁面有幾個錨點都要這功能，那麼我們可以簡單的實作如下。


```js
var applyScrolling = function (arr, cb) {
  for (var i = 0; i < arr.length; i++) {
    cb.call(null, i, arr[i])
  }
}

// 注意如果有使用 router 那麼自訂一個 class 可以避免一些問題
var anchors = document.querySelectorAll("a[href^='#']")
if (window.scrollTo) {
  applyScrolling(anchors, function (index, el) {
    var target = document.getElementById(el.getAttribute('href').substring(1))

    el.addEventListener('click', function (e) {
      console.log(target)
      e.preventDefault()
      // 這邊跟新的 method 參數是不同的。
      window.scrollTo(0, target.offsetTop)
    })
  })
}
```

# 譯者小結

目前在 Firefox 下兩種方式都可以使用，而 Chrome 則需要額外的開啟設定。本文就是先記錄一下這些新的屬性與 API。

# 參考

* [Smooth Page Scroll in 5 Lines of JavaScript](http://thenewcode.com/507/Smooth-Page-Scroll-in-5-Lines-of-JavaScript)

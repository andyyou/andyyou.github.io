---
layout: post
title: '參透 OOCSS'
date: 2015-08-11 05:40:00
categories: css
---

# OOCSS 的兩個核心觀念
* 分離結構(html tag 結構)與樣式(ui 的樣式)  Separate structure and skin
* 分離容器(layout 佈局)與內容(直接包 content 的 tag) Separate container and content
最後達到重複使用樣式的最高原則

# 搭配 BEM 的歸納邏輯
如果套上 BEM 的規則那就是

```
structure -> container (block)
          -> content (block | element)
skin -> modifier
```

1. 先找出相同結構的 html tag structure 以 blog 系統為例 列表頁有文章的 meta-data, 文章頁也有 meta-data, html 的結構一樣但視覺 ui 長的不同。這些結構就可以定義一個 object 按照 OOP 的觀念但實際上只是 html 和 css 的組成

這邊有一點要強調不管是 BEM 或 OOCSS 甚至其他方法都提到不要讓樣式相依於結構。舉例如下

{% highlight html %}
<div class="navbar">
  <span class="navbar-button">Logo</button>
</div>
{% endhighlight %}

{% highlight css %}
.navbar > span { /.../ } // 錯誤：這樣樣式就會相依於結構，一定要 .navbar 搭配子項目 span 才會成立
/* 另外也不要直接用 tag 選擇器，會導致對元素依賴，實際的例子即 bootstrap 的 .btn 可以套用到 a, button 上 */

.navbar-button { /.../ } // 此時應該獨立出一個 class name 針對該元素
{% endhighlight %}

第一步驟的重點在於找出可重複使用的 html 結構，概念上我們認為這個東西可以抽象化為一個物件

2. 找出結構後，再來分析誰是容器, 誰是內容，透過這樣的明確的定義與分類讓我們的 css 容易維護與增加新功能而不會影響舊有的程式碼。舉上面 meta-data 的例子，這是開發時很常見的狀況

{% highlight html %}
/*post中的meta-data*/
<div class="post">
  <p class=”metadata”>
    <a>Author name</a>commented on<a>21-02-2010</a>@
  </p>
</div>
/*comment中的meta-data*/
<div class="comment">
  <p class=”metadata”>
    <a>Author name</a>commented on<a>21-02-2010</a>@
  </p>
</div>
/*userinfo中的meta-data*/
<div class="user-info">
  <p class=”metadata”>
    <a>Author name</a>commented on<a>21-02-2010</a>@
  </p>
</div>
{% endhighlight %}

接著我們就會直覺得這樣做

{% highlight css %}
.post .metadata {css code}
.comment .metadata {css code}
.userInfo .metadata {css code}
{% endhighlight %}

一旦這麼做 meta-data 就會相依於容器，我們應該使用擴展的方式來做，結構只下`基礎通用的樣式`，而同樣結構不同樣式的部分我們透過擴展 class name 來做。這個時候就會建議使用 BEM 的命名原則 - 所以我們可以總結`容器`和`內容`在 OOCSS 裡都是屬於一種物件

{% highlight css %}
.metadata--post {css code}
.metadata--comment {css code}
.metadata--user-info {css code}
{% endhighlight %}

> 在使用 OOCSS 的過程中很容易掉入`表面外觀語意`的陷阱，例如 col-left, col-middle, bg-gray, text-border 這樣的命名。記得保持一種重點堅持以邏輯和語意來給元素命名，不要因為懶惰而隨意命名，例如 bg-red 應該用 bg-danger

3. 從 code 方面著手，對 code 重構，找出重複的 css rules, 通常會是像設定 background 等針對外觀，也就是說把 skin 抽出來。

# 重點回顧與注意事項
* Separate structure and skin 分離結構(html tag 結構)與樣式(ui 的樣式)  
* Separate container and content 分離容器(layout 佈局)與內容(直接包 content 的 tag) 
* 讓 css 不要具有相依性也是程式中所謂的低耦合的概念，透過 BEM + OOCSS 的拆分與命名原則可以達到
* 注意不要掉入`表面外觀語意`的陷阱，`堅持以邏輯和語意來給元素命名`

# 總結
簡單的來說，OOCSS 即透過兩個主要的原則將我們的 css 分類與抽象化成物件，兩個原則分別為 `Separate structure and skin` 和 `Separate container and content`，接著實作的第一步先找出重複的結構就是 html ，再依據其周遭元素的關係分類成`容器 container`和`內容 content` 再依據不同頁面或者同結構不同長相的部分來做 skin。我們定義好的這一些 class name 就好比是一個一個的物件。

過程中低耦合，即減少選擇器的相依性是主要的重點。如此才能達到重複使用也不會產生改一個爆一個的狀況。
低耦合的重點:
* 單純用 class name 選擇器，不要使用 tag, id 等，也不要從`物件`定義的外部來影響樣式。不使用 tag 意味著也不會限制使用的元素，class name 應盡可能讓所有元素都能套用。
* 堅持以邏輯和語意來給元素命名，搭配使用 BEM 實作起來更有規則
* 先下通用的基礎樣式，再使用擴展的方式來增加不同的 skin(modifier)

最後搭配 BEM 的圖示讓我們快速記住關係

~~~
structure -> container (block)
          -> content (block | element)
skin -> modifier
~~~

# 參考資源
[OOCSS觀念篇](http://www.w3cplus.com/css/oocss-concept)
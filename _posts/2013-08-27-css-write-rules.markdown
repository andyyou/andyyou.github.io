---
layout: post
title: 'CSS 覆寫規則'
date: 2013-08-27 11:55:00
categories: CSS
---
你是否曾經遇過這種情形，當您試著要套用某個 CSS 規則到元素的時候，它就是不產生變化。
網頁看起來就是忽略了您的 CSS ，然後根本搞不清楚到底是為什麼。
也許您使用了 !important 作為您最後的手段，這裡就有一個很好的機會來理解 CSS 選擇器的優先順序。

深入理解 CSS 優先順序可以幫助你減少 CSS 挫折，讓你的程式碼更加簡潔，更有組織，所以讓我們來看看三個控制 CSS 的規則。

* 選擇器的權重計算(Specificity)
* 繼承
* CSS 規則的累加

精通這些規則將會帶領你進入 CSS 開發的下一個層次

# 選擇器的權重計算

舉例來說您有一個 p 標籤包含著一段文字，設定如下

{% highlight html %}
<style>
p {font-size: 12px;}
p.bio{font-size: 36px;}
</style>
      
<p class='bio'>text here</p>
{% endhighlight %}

 您會猜測上面這個範例文字應該是 36px，第二句的選擇器 `p.bio` 更明確的指向該元素。
 然而有些時候選擇器並不是那麼單純好分辨。
 
{% highlight html %}
<div id='sidebar'>
	<p class='bio'>text here</p>
</div>

<style>
div p.bio {font-size: 14px;}
#sidebar p {font-size: 12px;}
</style>
{% endhighlight %}

乍看之下第一句 CSS 看起來更明確的指定元素，但實際上顯示的樣式是第二行。

為什麼？

要回答這個問題，我們需要考慮 CSS 的選擇器的權重計算。

關於 CSS 選擇器權重計算，是透過計算每個樣式的組成然後產生一組表達式`(a,b,c,d)`


* 元素，構造虛擬類別元素(a:root) =>   d = 1  // (0,0,0,1)
* class, 虛擬類別(a:hover), 屬性 => c = 1  // (0,0,1,0)
* Id => b = 1 													 // (0,1,0,0)
* Inline Style => a = 1 								 // (1,0,0,0)

你可以透過上述的規則開始計算選擇器的權重，算法就是每使用到一種就在該欄位`(a,b,c,d)` 加 1。
注意：(0,0,1,0) 的明確性大於 (0,0,0,15)

範例如下：
- p: 1 element – (0,0,0,1)
- div: 1 element – (0,0,0,1)
- #sidebar: 1 id – (0,1,0,0)
- div#sidebar: 1 element, 1 id – (0,1,0,1)
- div#sidebar p: 2 elements, 1 id – (0,1,0,2)
- div#sidebar p.bio: 2 elements, 1 class, 1 id – (0,1,1,2)

所以上面 sidebar 的範例就會是如下

{% highlight css %}

div p.bio{font-size: 14px;} //(0,0,1,2)
#sidebar p{font-size: 12px;}  //(0,1,0,1)

{% endhighlight %}

最後在我們繼續往下之前要先瞭解。`!important` 勝過 `Specificity`。

{% highlight css %}
div p.bio {font-size: 14px !important}
#sidebar p {font-size: 12px}
{% endhighlight %}

事實上，如果你能明確知道規則的優先順序你應該是不需要使用 `!important`。

# 繼承

繼承背後的想法是比較容易理解的。元素繼承父容器的樣式。如果你設置 body 標籤，`color: red;` 然後 body 內的所有元素的文本字體也將是紅色的，除非另外有其他設定。

但是並非所有的 CSS 屬性都是可繼承的。例如 `margin` 和 `padding` 是非繼承的屬性。如果您對一個 div 設置了 margin 和 padding 。你會發現 `div` 裡面的段落沒有繼承你的 `div` 設置 margin 和padding。該段將使用預設的瀏覽器
margin 和 padding，直到有另外的設定。

你可以使用明確宣告設定屬性的繼承，從它的父容器繼承樣式：

{% highlight css %}
p {margin: inherit; padding: inherit}
{% endhighlight %}

然後 p 就會繼承包復他的父元素。


# CSS 規則的累加

CSS 規則的累加與優先順序的工作原理如下。

1. 找到所有的 css 宣告和對應的元素。
2. 根據原始碼和優先權排序。

  !important
    
	頁面設定為優先 

		{% highlight html %}
		<style>
		p {font-size: 12px}
		</style>
		{% endhighlight %}    

	外部載入
		{% highlight html %}
		<link rel='stylesheet' href='style.css' />
    {% endhighlight %} 
3. 選擇器的權重計算(Specificity)
4. 如果兩個規則相等則宣告的最後宣告的規則會覆寫。
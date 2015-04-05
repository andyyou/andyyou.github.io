---
layout: post
title: 'JSX 常見的陷阱'
date: 2014-02-10 11:00:00
categories: Reactjs
---

# JSX 常見的陷阱
JSX 看起來像 HTML 但有一些您應該知道關鍵性的差異。
注意：對於和 DOM 之間的差異，例如行內式屬性設定(inline style)，請查閱[這裡](http://facebook.github.io/react/docs/dom-differences.html)。

> DOM 的差異：
React 為了跨瀏覽器和提升效能的因素，實作一套和瀏覽器本身無關的 events 以及模擬 DOM 的機制。我們可以借由這個機制處理一些關於原始 DOM 設計上一些不足的地方。
* 所有的 DOM 屬性 `Properties` 和 `Attributes` (包含事件)都應該使用駝峰式命名 `camelCased` ，這和一般的 Javascrpt 程式碼風格一致。我們故意在這邊違背 html 規格 ，因此這和 html 規格是不同的。
* `style` 屬性透過 Javascript 物件和駝峰式的屬性來設定，而不是 CSS 字串。所以設定 CSS 的語法風格會和 DOM, Javascrit 屬性一致，外加這麼做可以防止 XSS 攻擊。
* 所有在事件符合 W3C 規範，且所有事件(包含 submit)傳遞都遵照 W3C 規範，查閱 [Event System](http://facebook.github.io/react/docs/events.html) 取得更多資訊。
* 關於 `onChange` 事件行為就跟你所期待的一樣，當一個表單欄位改變了，事件就會被觸發，而不是在 `onblur` 失去焦點的時候才觸發。 我們特意違背現有的瀏覽器行為，因為原始的 `onChange` 事件行為跟其名稱並不符合，React 需要正確的用到這個 Event ，當使用者輸入資料的同時 React 就會及時反應。查閱[Forms](http://facebook.github.io/react/docs/forms.html)得知更多資訊。
* 表單輸入的屬性例如 `value` `checked` 更多關於一些命名，用法，等請查閱 [Forms](http://facebook.github.io/react/docs/forms.html)

# 移除空白字元
JSX 不像 HTML 在渲染時如果在同一個點重複空白字元會保留一個其他自動移除，關於這點 JSX 會移除所有在 `{ }` 之間的空白。如果你需要加入空白字元則要使用 `{' '}`。

{% highlight html %}
<div>{this.props.name} {' '} {this.props.surname}</div>
{% endhighlight %}

如果你對這個設計有什麼想法，歡迎加入[Issue #65](https://github.com/facebook/react/issues/65)討論。

# HTML 字元實體
您可以插入 HTML 字元實體在 JSX 裡：

{% highlight js %}
<div>First &middot; Second</div>
{% endhighlight %}

如果你想要顯示一個 HTML 字元實體在動態的內容中，你會遇到重複跳脫字元的問題。因為 React 為了防止 XSS 會把所有要呈現的文字都先跳脫(escapes)。

{% highlight html %}
// 不好的示範: 會輸出 "First &middot; Second"
<div>{'First &middot; Second'}</div>
{% endhighlight %}

這裡有一些方式可以解決這個問題。最簡單的方式就是在 Javascript 直接寫 `unicode`，不過你需要確定檔案被存成 `UTF-8` 格式。

{% highlight js %}
<div>{'First · Second'}</div>
{% endhighlight %}

一個更安全的替代方式式找到 `unicode` [對應的編碼](http://www.fileformat.info/info/unicode/char/b7/index.htm)

{% highlight html %}
<div>{'First \u00b7 Second'}</div>
<div>{'First ' + String.fromCharCode(183) + ' Second'}</div>
{% endhighlight %}

也可以把字串混合進陣列裡面

{% highlight js %}
<div>{['First ', <span>&middot;</span>, ' Second']}</div>
{% endhighlight %}

當你要插入 HTML 的時候你可以用這最後一招

{% highlight html %}
<div dangerouslySetInnerHTML={{__html: 'First &middot; Second'}} />
{% endhighlight %}

# 自定 HTML 屬性
如果你傳給 HTML 元素的屬性並不在 HTML 規範中，React 並不會渲染它。如果你想要自訂一個屬性(attribute)。你應該使用前綴詞 `data-`。

{% highlight html %}
<div data-custom-attribute="foo" />
{% endhighlight %}

無障礙網站的話屬性使用 `aria-` 開頭。

{% highlight html %}
<div aria-hidden={true} />
{% endhighlight %}


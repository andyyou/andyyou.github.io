---
layout: post
title: '在 CodePen 中使用 React'
date: 2014-09-16 13:20:00
categories: Reactjs
---

## 在 codepen.io 上使用 React
為了能夠在 CodePen 上使用 React 和 JSX 您必須要:

1. 加入這支 script 到 CodePen `http://codepen.io/chriscoyier/pen/yIgqi.js`
2. React: `http://fb.me/react-0.11.1.js`
3. JSX Transformer: `http://fb.me/JSXTransformer-0.11.0.js`

![](http://i.imgur.com/kEvEjr2.png?1)
![](http://i.imgur.com/Yke1r46.png)

## 緣由
React 是一個由 Facebook 團隊所提供的一組 Javascript 函式庫。
當您開始使用 CodePen 撰寫一些 React 範例時會發現 CodePen 無法正常運作。
這是因為當您在 `Javascript` 區塊輸入程式碼時，他其實只是在您的文件上加上一個
`<script>` 標簽且並沒有定義任何 `type` 。
所以當您想使用 CodePen 轉寫一些小範例時您有幾種選擇:
1. 不使用 JSX，使用類似像 `React.DOM.div` 之類的原生 JS 取代
2. 將 JS 寫在 `html` 區塊，並且使用 `<script type='text/jsx'>`
3. 使用上述的方式加入一段 script

關於上面這一小段程式碼是由[Mark Funk](http://codepen.io/mfunkie/)所提出的一個解法。

{% highlight js %}
(function() {
  function runScripts() {
    var bodyScripts = 'body script:not([src])';
    Array.prototype.forEach.call(document.querySelectorAll(bodyScripts), function setJSXType(element) {
      element.setAttribute('type', 'text/jsx');
    });
  };

  if (window.addEventListener) {
    window.addEventListener('DOMContentLoaded', runScripts, false);
  } else {
    window.attachEvent('onload', runScripts);
  }
})();
{% endhighlight %}

簡單來說這段程式碼會尋找 CodePen 放置到預覽中的 script 標簽並且加入 `type` 。
附帶一提的是，JSX Transformer 是用來協助您方便開發的並不適用于發佈的產品上。
最後，當您發生錯誤時請檢查您瀏覽器的 `console`，JSX Transformer 會很貼心的
提示您錯誤訊息。

![](http://i.imgur.com/yjSKQod.png)

---
title: Vue.js 由 1 到 2 的旅程 - (2)
tags:
  - javascript
  - vuejs
categories: Program
date: 2016-12-16 16:00:00
---


第二集，我們要聊的是一個很瑣碎的點也許會造成您困惑，在精神不濟的情況下建議略過本文。如果您跟我一樣閱讀的是英文版的文件加上英文又不是很牛，那麼我想這個點可能會造成您的困惑。
如果您也曾對 `string templates` 和 `non-string templates` 兩個詞有疑惑的話就讓我們繼續看下去...

<!--more-->

# 在開始之前

我們要先介紹的是 `Vue 的 templates 提供了兩種實作方式`。什麼意思，就是我們可以透過 Vue 的 template 屬性（options）或者 DOM templates 的方式來撰寫樣板。看看下面兩種範例

### No 1 template 屬性

```js
// template 屬性
new Vue({
  el: '...',
  template: `<div>1</div>`
})
```
### No 2 DOM templates

不同的 HTML 與 Javascript 檔案：

```html
<div id="app">
  <!-- 標籤結構寫在 HTML 中的 -->
  <p>{{ message }}</p>
</div>
```

```js
new Vue({
  el: '#app',
  data: {
    message: 'Hello, Vue.js'
  }
})
```

小結一下就是 template 可以寫在 Javascript 又或者寫在 HTML 裡面。

# 疑惑的產生

當我們開始閱讀 Vue 2 官方文件時，會發現在第一版時並沒有 [Template Syntax](https://vuejs.org/v2/guide/syntax.html) 一章。
原因是在該章節的一開始便開宗明義的提到：

> Under the hood, Vue compiles the templates into Virtual DOM render functions.

這個點算是第二版一個極大的變化。所以我們理解 Vue 會將 `template 屬性` 或者 `DOM Templates` 即標籤結構寫在 HTML 裡面的作法編譯成 `render function`。

接著 [Template Syntax](https://vuejs.org/v2/guide/syntax.html#Raw-HTML) 一節提到

> Vue is not a string-based templating engine

所以 Vue 並不是 string-based 的樣板引擎會編譯成 render function 我們懂了。
但是 [Components](https://vuejs.org/v2/guide/components.html#DOM-Template-Parsing-Caveats) 一章竟然出現

> It should be noted that these limitations do not apply if you are using `string templates` from one of the following sources

* `<script type="text/x-template">`
* `JavaScript inline template strings`
* `.vue components`

奇怪？前面不是說 Vue `不是` string-based template 嗎？現在提出 string templates 還附上三種類型。雖然英文字不一樣不過換成中文來說到底是什麼意思啊？不都是字串類型的樣板嗎？

加上 [Components camelCase-vs-kebab-case](https://vuejs.org/v2/guide/components.html#camelCase-vs-kebab-case) 提到關於屬性（attributes）的命名方式在 string templates 與 non-string templates 有所差異。non-string templates 的 props 一定要用 kebab-case 的命名方式，而 string templates 則不受限制。心中出現滿滿的 WHY

我們想知道 `template 屬性`、`DOM templates`、`string-based template`、`string templates`、`non-string templates` 這些詞之間的關係到底是什麼？

# 問答

上面的說明肯定搞的您很混亂，沒關係讓我們先整理出問題。我們要問的是：

### 1. string-based templating engine 與 string-templates 的意義分別是啥？

Vue 文件一開始提到 Vue 並不是 string-based templating engine 意味著並不是組出字串後完全交給瀏覽器，而是會將這些 template 轉成 render function 的機制。

而 string templates 指的是在 Javascript 中寫入字串格式的 templates，即上面提到的三種形式：

* `<script type="text/x-template">`
* `JavaScript inline template strings`
*  `.vue components`

才屬於 string templates，雖然看起來不太一樣但它們都是屬於 Javascript 的範疇。

### 2. 文件中的 string templates 與 non-string templates 具體是指？

所謂的 string templates 概略來說是指在 Javascript 以字串形式出現的 templates。所以其他不在 Javascript 裡面的樣板都是 non-string templates 最一開始提到的 DOM templates 也屬於這個分類，換句話說其他在 HTML 寫的也就是 DOM templates 屬於 non-string templates。我們只要以檔案類型來區分便可。

造成困惑的地方 - 一開始說 Vue 不是 string-based 但後來又出現 string templates 這兩者指的是不同東西。一個是編好完整字串交給瀏覽器的作法，一個是指在 Javascript 字串形式的 templates。

驗證 template 屬性屬於 string templates 即使屬性使用 camelProp 也沒問題

```js
Vue.component('row', {
  props: [
    'camelProp'
  ],
  template: `
    <div class="row">
      Row: {{ camelProp }}
    </div>
  `
})

new Vue({
  el: '#root',
  template: `
    <row :camelProp="message"></row>
  `,
  data: {
    message: 'Hello, Vue'
  }
})
```

使用 inline-template 來觀察差異

* [範例1 string templates](https://jsfiddle.net/7vkzLdco/)
* [範例2 non-string templates](https://jsfiddle.net/7vkzLdco/1/) tag 的屬性若不是使用 kebab-case 則無法取得屬性

### 3. DOM templates 的方式也會編譯為 render function 嗎？如果會編譯成 render function 那為什麼 DOM templates 就不行？

Vue 會將所有 template 都編譯成 render function 所以 DOM templates 也會編譯成 render function。其中差異在於取得的資料的時間點由於

> Vue can only retrieve the template content after the browser has parsed and normalized it

在瀏覽器解析之後 camelCase 的屬性會全部轉成小寫而無法分辨與切割屬性名稱（myProp -> myprop）。也就造成和 props 定義的名稱不同。因此 DOM templates 無法支援 camelCase 的屬性名稱。

# 補充

在使用 DOM templates 時需注意：由於瀏覽器限制某些標籤存在的位置例如 `<option>` 只能在 `<select>` 裡面，`<table>` 裡面要放 `<tr>`, `<td>` 等等。當不合法的時候這些標籤會被移出該標籤內

```html
<table>
  <tr>
    <h3>Column</h3>
  </tr>
</table>
```

h3 會被移出 table 外層導致顯示錯誤，此時可以使用 is 屬性處理，或者就用 string templates 就正常了，為了避免問題產生還是遵循 HTML 規則較為單純。

沒有使用 is 不合法的元素被移出了 table 示意圖

![](http://imgur.com/CuQ3C1h.png)

# 總結

又來了，如果您一開始就直上 vue-loader 相信這些問題壓根都不會遇到。原因是你全部的 template 都屬於 string templates。相較之下如果您使用了像是小弟之前撰寫的[穠纖合度的整合 Rails、Webpack、Vuejs](http://andyyou.github.io/2016/11/07/integrate-rails-webpack-vue/)一文，那麼裡面使用的 inline-template 搭配 DOM template（為了取得 Rails View 來的資料）那麼上面提到的一些限制可能就是您會遇到的問題了。

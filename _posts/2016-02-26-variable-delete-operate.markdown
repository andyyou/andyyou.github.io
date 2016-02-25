---
layout: post
title: '關於 delete 變數詭異行為與解釋'
date: 2016-02-25 12:00:00
categories: javascript, rails
---

# 問題

發生在當我們撰寫下面程式碼的時候所發現的奇怪行為。

{% highlight js %}
var a = 1;
b = 2;

console.log(a); // 1
console.log(b); // 2

// 使用瀏覽器測試請使用 window 如果是用 node 的 REPL 環境那請改成 global
console.log(window.a); // 1
console.log(window.b); // 2

// 但接下來這麼操作

delete a; // false 不能刪
delete b; // true 可以刪

a; // 1
b; // undefined

{% endhighlight %}

# 解釋

首先根據[ECMA-262 §10.3,](http://ecma-international.org/ecma-262/5.1/#sec-10.3)的定義 `Variable environment` 是一種特定
型別的 `Lexical environment`，我們沒辦法透過任何方式直接存取。

一個 `Lexical environment` 用來記錄執行環境的資訊，可以把它想成是一個物件，我們會把在一個`執行環境 Context`的變數，函數都存在這個物件的屬性上
針對函數那些定義的參數(Parameter)也會被記錄，舉例來說 `function foo (a, b)()` 中的 `a` 和 `b` 就會被記在 `foo` 的執行環境資訊中。

一個 `Lexical environment` 也有一個連結可以連結到外在的 `Lexical environment` 就是所謂的 `scope chain`。
這個機制可以協助我們取得目前執行環境以外的變數，舉例來說就是 function 裡面可以拿到 global 的變數。

一個 `Variable environment` 就只是 `Lexical environment` 的一部份，本質上就是透過 `var` 宣告在執行環境中的變數或函數。

# 使用了 var

上面的 `a` 使用了 `var` 根據 ECMAScript 定義會被記錄在 `Variable environment` 根據定義 `Variable environment` 是不能手動刪除的。
也就是說除非用了 `eavl`，否則是不能被 `delete` 的。

{% highlight js %}
var a = 1;
delete a; // false
console.log(a); // 1

// 記得清除整個環境
eval('var a = 1; delete a; console.log(a)'); // undefined

{% endhighlight %}

# 沒使用 var

當我們賦值卻沒有用 `var`，Javascript 會嘗試在 `Lexical environment` 尋找同名的參考。
最大的不同是 `Lexical environment` 是嵌套的，就是它可以關聯到外面其他的 `Lexical environment`。
當在本地找不到的時候就會往上層去找，換句話說每個 `Lexical environment` 都有個`爸爸`，而最外層的就是 `global`
所以當我們不使用 var 而宣告一個變數時會開始在各個 scope 尋找同名變數，最終 Javascript 會拿一個 window(global) 的屬性來當作參考。
而物件的屬性是可以刪除的。

結論就是第一個 var 的變數被放在 `Variable environment` 是不能 `delete` 的而第二個沒有 var 的變數它是 global 的屬性。

---
layout: post
title: "Javascript 三種宣告的差異"
date: 2016-03-01 12:00:00
categories: Program
tags: javascript
---

關於下面這三種宣告的差異

<!--more-->

~~~js
var a = 0;
a = 0;
window.a = 0;
~~~


### var a = 0;

當我們在全域使用這樣的宣告時，就是`替 global execution context 在 `variable object` 中建立一個變數`。
在`瀏覽器`中這個 `variable object` 同時有個別名叫做 `window` 並且它是一個 DOM 是 window 物件而不只是一般的物件。
(在一般非瀏覽器的實作這可能是對的)。上面這個 `var a = 0` 的結果就是你在 `window` 中建立了一個屬性 `a` 且不能刪除。
它同時也在第一行程式執行之前被定義，注意在 IE8 之前在 window 建立的屬性並不屬於 `enumerable` 意思是你不能用 `for..in` 去遍歷。
在 IE9, Chrome, Firefox, Opera 這些屬性是 `enumerable` 的。

~~~js
var car = { name: 'Lexus RX200t' };

// 測試 toString 是否存在物件中
console.log('toString' in car); // true
console.log(typeof car.toString); // function

// 但，當我們用 for..in 卻看不到
for (var k in car) {
  console.log(k); // 只有 "name" 輸出
}

// 一個屬性是否是 enumerable 要看物件本身的 enumerable 屬性設定
// 我們可以透過 property descriptor 來觀察

var d = Object.getOwnPropertyDescriptor(car, 'name');
console.log(d.enumerable); // true

// 要拿到所有能用的屬性則是 getOwnPropertyNames
console.log(Object.getOwnPropertyNames(Object.prototype));
~~~

* [解釋 Enumerable](http://stackoverflow.com/questions/17893718/what-does-enumerable-mean/17893807)

### a = 0

在全域的情況下，隱含式的在 `window` 建立屬性。注意這並沒有在 `variable object` 建立變數，所以一個普通的屬性是可以被 `delete` 刪除的。
強烈建議不要這麼做。這會導致您的程式產生疑義語意不清。

### window.a = 0

顯式的宣告一個 window 的屬性，同樣是可以刪除的。

### 進一步 this.a = 0 呢

在全域的情況下，`this` 指向 `window` 所以這跟 `window.a = 0` 是一樣的

### 當 var 的時候，發生了什麼事

當我們透過 `var` 定義了一個`變數名稱`(指的是 var n = 0，n 這個識別符號)，這個變數名稱會發生在任何程式碼執行之前
詳細解釋請[參考](https://segmentfault.com/a/1190000004491834)

~~~js
console.log('foo' in window); // true
console.log(window.foo); // undefined
console.log('bar' in window); // false
console.log(window.bar); // undefined

var foo = 'foo'; // 看懂上面有無記錄在 variable object 和單純 property 的差異了嗎？
bar = 'bar';

console.log('foo' in window); // true
console.log(window.foo); // foo
console.log('bar' in window); // true
console.log(window.bar); // bar

~~~

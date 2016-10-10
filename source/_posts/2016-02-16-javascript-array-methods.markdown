---
layout: post
title: 'Javascript Array 筆記'
date: 2016-02-16 16:30:00
categories: Program
tags: javascript
---

方法範例

~~~js
// push
var arr = [1, 2, 3];
var result = arr.push(4); // result: 4, arr: [1, 2, 3, 4]

// pop
var arr = [1, 2, 3];
var result = arr.pop(); // result: 3, arr: [1, 2]

// length
var arr = [1, 2];
arr.length; // 2

// unshift
var arr = [1, 2, 3];
var result = arr.unshift(0); // result: 4, arr: [0, 1, 2, 3]

// shift
var arr = [1, 2, 3];
var result = arr.shift(); // result: 1, arr: [2, 3]

// splice 拼接
// Array.splice(start_index, remove_num, element01, element02...)
var arr = [1, 2, 3, 4, 5];
var result01 = arr.splice(2, 1, 6); // result01: [3], arr: [1, 2, 6, 4, 5]
var arr = [1, 2, 3, 4, 5];
var result02 = arr.splice(2, 0, 6); // result02: [], arr: [1, 2, 6, 3, 5, 6]
// 沒移除元素會放在該 index 元素會被往後推
var arr = [1, 2, 3, 4, 5];
var result03 = arr.splice(2, 0, 6, 7, 8) // result03: [], arr: [1, 2, 6, 7, 8, 3, 4, 5]

// slice 切片
var arr = [1, 2, 3, 4, 5];
var result01 = arr.slice(1, 3); // result01: [2, 3], arr: [1, 2, 3, 4, 5]
var arr = [1, 2, 3, 4, 5];
var result02 = arr.slice(-2); // result02: [4, 5], arr: [1, 2, 3, 4, 5]
var arr = [1, 2, 3, 4, 5];
var result03 = arr.slice(-2, 0); // result03: [], arr: [1, 2, 3, 4, 5]
var arr = [1, 2, 3, 4, 5];
var result04 = arr.slice(0); // result04: [1, 2, 3, 4, 5], arr: [1, 2, 3, 4, 5]

// indexOf
var arr = [1, 2, 3, 4, 5];
var result01 = arr.indexOf(3); // result01: 2
var arr = ['a', 'ab', 'abc', 'abcd'];
var result02 = arr.indexOf('a'); // result02: 0
var result03 = arr.indexOf('b'); // result03: -1
var result04 = arr.indexOf('ab'); // result04: 1

// lastIndexOf
var arr = [1, 2, 3, 2, 1];
var result01 = arr.lastIndexOf(1); // result01: 4
var arr = ['a', 'ab', 'abc', 'ab', 'a'];
var result02 = arr.lastIndexOf('a'); // result02: 4
var result03 = arr.lastIndexOf('b'); // result03: -1
var result04 = arr.lastIndexOf('ab'); // result04: 3

// some 陣列是否有元素符合測試
var arr = [1, 2, 3, 4, 5];
var result01 = arr.some(function (element, index, array) {
  return element > 3
});  // result01: true
var result02 = arr.some(element => element > 5); // result02: false

// every 陣列中是否所有元素都符合測試
var arr = [1, 2, 3, 4, 5];
var result01 = arr.every(function (element, index, array) {
  return element > 3
}); // result01: false
var result02 = arr.every(element => element < 6); // result02: true

// join 串聯成字串
var arr = [1, 2, 3, 4, 5];
var result = arr.join(','); // result: "1,2,3,4,5"

// sort 由小排到大
var arr = [5, 4, 3, 2, 1];
var result = arr.sort(); // result: [1, 2, 3, 4, 5], arr: [1, 2, 3, 4, 5]
var arr = ['c', 'b', 'a', 0];
var result = arr.sort(); // result: [0, 'a', 'b', 'c'], arr: [0, 'a', 'b', 'c']

// reverse 由大排到小
var arr = [1, 2, 3, 4, 5];
var result = arr.reverse(); // result: [5, 4, 3, 2, 1], arr: [5, 4, 3, 2, 1]
var arr = [0, 'a', 'b', 'c'];
var result = arr.reverse(); // result: ['c', 'b', 'a', 0], arr: ['c', 'b', 'a', 0]

// concat 組合並回傳一個新的陣列(flat element)
var a = [1, 2];
var b = [3, 4];
var c = a.concat(b); // a: [1, 2], b: [3, 4], c: [1, 2, 3, 4]
var d = [...a, ...b]; // a: [1, 2], b: [3, 4], d: [1, 2, 3, 4]
var e = a.concat(0, [3, 4]); // e: [1, 2, 0, 3, 4]

// forEach
var arr = [1, 2, 3, 4, 5];
arr.forEach(function (element, index, array) {
  console.log(element, index, array);
});
// 1, 0, [1, 2, 3, 4, 5]
// 2, 1, [1, 2, 3, 4, 5]
// 3, 2, [1, 2, 3, 4, 5]
// 4, 3, [1, 2, 3, 4, 5]
// 5, 4, [1, 2, 3, 4, 5]
// 注意: 例外無法阻止 forEach 停止

// map 遍歷所有元素至參數 callback 然後回傳一個新的陣列
var arr = [1, 2, 3, 4, 5];
var result01 = arr.map(function (element) {
  return element + 10;
}); //  arr: [1, 2, 3, 4, 5], result01: [ 11, 12, 13, 14, 15 ]
var arr = [1, 4, 9];
var result02 = arr.map(Math.sqrt); // arr: [1, 4, 6], result02: [1, 2 ,3]

// filter 過濾元素，產生新陣列
var arr = [1, 2, 3];
var result = arr.filter(function (element) {
  return element > 2;
}); // arr: [1, 2, 3], result: [3]

// reduce 從左至右當作累加器把陣列變成單值
var arr = [1, 2, 3, 4, 5];
var result = arr.reduce(function (previous, current, currentIndex, array) {
  return previous + current;
}, 100); // result: 115

// reduceRight
var arr = ['ab', 'bc', 'de'];
var result = arr.reduceRight(function (pre, cur, index, arr) {
  return pre + cur;
}, 'I am initial:'); // result: "I am initial:debcab"

// 其他補充
var a; // a: undefined
var b = a += 2; // a: NaN, b: NaN => undefined += 2 => NaN

var a = null;
var b = a += 2; // a:2, b: 2
~~~

# Array.forEach 額外補充

根據 MDN 的定義 `arr.forEach(callback[, thisArg])` 當我們要在 forEach 的匿名函數中使用 `this` 我們可以透過 `thisArg` 來設定。
如果不設定(undefined, null)或者不用 `var self = this;` 的方式保留 context，那麼預設會是指向 `global`

~~~js
var sum = 100;
function Counter () {
  this.sum = 0;
}

Counter.prototype.add = function (arr) {
  arr.forEach(function (el, i, arr) {
    this.sum += el;
  })
};

var o = new Counter();
o.add([1, 2, 3]);
console.log(o.sum); // 0
console.log(sum); // 106
~~~

在 Chrome 或者 Firefox console 底下執行的確會執向 global，但是在 nodejs v5.6.0 底下結果卻是

~~~js
console.log(o.sum); // 0
console.log(sum); // 100
~~~

要注意。

正確的範例如下

~~~js
function Counter () {
  this.sum = 0;
}

Counter.prototype.add = function (arr) {
  arr.forEach(function (el, i, arr) {
    this.sum += el;
  }, this); // <-- here this "thisArg"
}

var o = new Counter();
o.add([1, 2, 3]);
console.log(o.sum); // 6
console.log(sum); // undefined
~~~

# callback methods

* filter (has thisArg)
* some (has thisArg)
* every (has thisArg)
* forEach (has thisArg)
* map (has thisArg)
* reduce
* reduceRight

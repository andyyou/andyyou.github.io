---
title: "[譯]typeof 與 instanceof 技巧 - 簡易的動態型別檢查"
categories: Program
tags: [javascript]
---

這邊文字要講的是關於使 `instanceof` 可適用於更多的情形下。

# 1. 關於 typeof vs instanceof

在 javascript 中，當我們需要確認一個值的型別時，我們就必須要選擇對應的方法。大略來說：

* `typeof` 檢查某值是否為某種原生型別

```js
if (typeof value === 'string')
```

<!--more-->

* `instanceof` 用來檢查某個值是否為某 class 的實例物件或建構函式。

```js
if (value instanceof Map)
```

除此之外，`value.constructor` 和 `value.constructor.name` 偶爾也可以派上用場。

光是檢查原生型別和物件的方式不一樣這樣就已經非常不理想了，然後在加上 javascript 一些詭異的行為就讓事情變的更加複雜

* `typeof null` 為 `object` 並不是 `null`。(typeof undefined -> undefined)
* `typeof`  區分 `object` 和 `function` ，而兩者都是一種物件

上面這兩個詭異的行為導致`沒有簡單方式直接拿 typeof 來檢查是否為物件`

* 並非所有的物件都是 `Object` 的物件實例

```js
> Object.create(null) instanceof Object
false

o = {};
// 等於
o = Object.create(Object.prototype);
```

# 2. 讓 instanceof 可以檢查原生型別

使用特定的 `PrimitiveNumber` 類別，下面的程式碼可以讓 expression 表達式 `x instanceof PrimitiveNumber` 為 true。
而能夠辦到這樣的原因是因為我們使用 `Symbol.hasInstance` 實作了 PrimitiveNumber 的 [static method 靜態方法](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Classes/static)。

關於 [Symbol.hasInstance](http://exploringjs.com/es6/ch_oop-besides-classes.html#_property-key-symbolhasinstance-method)。
簡單說就把 `Symbol.hasInstance` 當作 method 的 key 就是提供我們客製 instanceof 行為的方法。

```js
class PrimitiveNumber {
  static [Symbol.hasInstance](x) {
    return typeof x === 'number'
  }
}

console.log(123 instanceof PrimitiveNumber) // true
```

3. 使用 TypeRight 函式庫檢查動態型別

TypeRight 是一個微型的函式庫用來檢查動態型別，另外它使用上面說明的方式實作。其 `instanceof 運算子` 如下

* PrimitiveUndefined
* PrimitiveNull
* PrimitiveBoolean
* PrimitiveString
* PrimitiveSymbol

TypeRight 目前並不提供`物件`檢查的型別（PrimitiveObject) 不過您可以輕易的加入。

通過這些基本的功能，我們可以使用 `TypeRight` 檢查函式的參數是否正確

```js
import * as tr from 'type-right'

function dist(x, y) {
  tr.force(x, tr.PrimitiveNumber, y, tr.PrimitiveNumber)
  return math.hypot(x, y)
}

dist(3, 4) // 5
dist(3, undefined) // TypeError
```

# 4. 其他檢查型別的方式

目前有兩個`提案`跟處理檢查型別有關的功能，正處於 stage 0 的狀態

### 4.1 模式匹配

這個`提案`稱為 ECMAScript Pattern Matching Syntax 是 Brian Terlson 和 Sebastian Markbage 提出的。
其中最主要的是 Symbol.matches 的概念，其大概的使用方式入下

```js
class PrimitiveNumber {
  static [Symbol.matches](x) { // (A)
    return x instanceof this;
  }
}
```

### 4.2 Builtin.is() 和  Builtin.typeOf()

這個提案是由 James M Snell 提出的。

`Builtin.is(value1, value2)` 可以檢查 `value1` 和 `value2` 是否參考同一個 constructor。這個提案也考慮到了[其他面向的問題](http://speakingjs.com/es5/ch17.html#cross-realm_instanceof)

```js
> Builtin.is(Date, vm.runInNewContext('Date'))
true
> Builtin.is(Date, Date)
true
```

而 `Builtin.typeOf()` 看起來就像是 `typeof` 的擴展，同時支援原生型別和內建的 class

```js
Builtin.typeOf(undefined); // 'undefined'
Builtin.typeOf(null); // 'null'
Builtin.typeOf(123); // 'number'

Builtin.typeOf(new Number()); // 'Number'
Builtin.typeOf([]); // 'Array'
Builtin.typeOf(new Map()); // 'Map'

// The builtin counts, not the user-defined class
class MyArray extends Array {}
Builtin.typeOf(new MyArray()); // 'Array'
```

# 參考

* [Beyond typeof and instanceof: simplifying dynamic type checks](http://2ality.com/2017/08/type-right.html)

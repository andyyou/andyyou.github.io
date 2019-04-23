---
title: "關於 Javascript {} + {}"
date: 2016-12-26 19:32:29
categories: Program
tags: ["javascript"]
---

這篇文章源自 [What is {} + {} in JavaScript?](http://www.2ality.com/2012/01/object-plus-object.html) 其實早在 2012 年就問世了。
時至 2016 年末純粹是在聊天時重提這個問題，但由於年紀大了記憶力不佳，竟然記`錯`了，所以才會有這一篇重新紀錄的筆記。

源頭是當時由 Gary Bernhardt 在閃電秀中指出 Javascript 的詭異行為 - [Wat](https://www.destroyallsoftware.com/talks/wat)

<!--more-->

在開始之前我們先補充一下關於 Javascript 型別的整理

* 基礎型別（Primitive Type）
  + string
  + number
  + boolean
  + nudefined
  + null
  + symbol（ECMAScript 6）
* 物件型別（Object Type）
  + object
    - Function
    - Array
    - Date
    - 其他

關於 Javascript 的加法其實是很簡單的：原則上您只能夠將數字（Number）或字串（String）相加。
於是其他的東西相加的時候將會被轉型成數字或者字串。為了理解`轉換`的機制我們需要先釐清一些事情，我們得引用 [ECMA-262 5.1](http://www.ecma-international.org/ecma-262/5.1/Ecma-262.pdf) 版規範的 9.1 章節或新版 ECMA-262 7 的 [7.1.1](https://www.ecma-international.org/ecma-262/7.0/index.html#sec-type-conversion) 的說明

讓我們來複習一下，在 Javascript 中關於型別的大分類 - 有兩種類型的`值`：

* primitive 原生
* object 物件

就像上面列出來的除了 undefined, null, boolean, number, string, symbol 之外的東西都是物件，當然陣列和函式都是物件的一種。

# 轉換

加法運算子整體來說會執行三種類型的轉換，結果就是它會將值轉成原生型別 primitive 中的 Number 或 String

## 1.1 使用 ToPrimitive() 將值先轉換成為原生型別

在內部 ToPrimitive() 的使用調用格式為 `ToPrimitive(input, PreferredType?)`
第二個為可選參數 PreferredType 值可以是 Number, String，這只是一個轉型偏好的註記，最終的結果可以是任一原生型別。
假設 PreferredType 是 Number 那麼執行轉換的步驟如下

1. 如果輸入是原生型別，直接回傳
2. 否則調用 obj.valueOf() 如果值是原生型別就回傳
3. 還不是原生型別的話則調用 obj.toString() 結果如果是原生型別就回傳
4. 否則拋出例外

如果 PreferredType 是 String 則 2, 3 步驟交換。
如果沒有 PreferredType 則 Date 預設為 String，其他型別則預設是 Number。

## 1.2 使用 ToNumber() 轉換為數字

下列說明 ToNumber() 是如何轉換原始型別為數字

* undefined -> NaN
* null -> +0
* boolean
  + true -> 1
  + false -> +0
* number -> 不轉換
* string -> 將字串轉換成數字，不過這其中有些小細節下面整理給您，結果與 Number(input) 是一樣的
  + +'23.1' = 23.1
  + +'2e1' = 20
  + +'25px' = NaN
  + +'p23' = NaN
  + +'010' = 10
  + +'0xf' = 15
  + +[1] = 1
  + +[1, 2] = NaN

過程是這樣的：一個物件 obj 透過呼叫 ToPrimitive(obj, Number) 轉換成原始型別，接著在使用 ToNumber() 取得最後的結果

## 1.3 使用 ToString() 轉換為字串

下面說明 ToString() 如何轉換原生型別為字串

* undefined -> 'undefined'
* null -> 'null'
* boolean
  + true -> 'true'
  + false -> 'false'
* number -> '1.234'
* string -> 不轉換

一個物件 obj 透過調用 ToPrimitive(obj, String) 轉換為原始型別，然後 ToString() 取得最後結果

## 1.4 實作

下面這個物件可以讓我們觀察轉換的過程

```js
var obj = {
  valueOf: function () {
    console.log('valueOf')
    return {}
  },
  toString: function () {
    console.log('toString')
    return {}
  }
}

Number(obj)
+obj // 等價
```

```
> valueOf
> toString
> TypeError: can't convert obj to number
```

當 Number() 作為 function 使用時內部會執行轉換 ToNumber() 的流程，根據上面的實作可以看出就如我們上面所敘述的規則流程一樣。
整個過程即先依據 PreferredType 轉換為原始型別，這裡要注意並不是最終結果，再來依據需要看是否要再將原生型別轉成數字或字串。

# 加法

舉例下面的例子

```
val1 + val2
```

要解析上面這個 expression 德遵循 ECMA-262 5.1 規範的 11.6.1 章節或新版 ECAM-262 7 版的 12.8.3 說明的步驟：

(1). 轉換兩邊的運算元為原生型別 Primitive

~~~js
prim1 = ToPrimitive(val1)
prim2 = ToPrimitive(val2)
~~~

由於 PreferredType 被省略了，因此物件除了 Date 是代入 String 外，其他的是 Number。

(2). 兩數相加的情況下，如果 prim1 或 prim2 只有要一個是 String 那麼兩者都會被轉成字串，最終結果就是串接字串。

(3). 否則，兩者都會被轉成數字並加總。

到這一步可能造成困惑的地方：

```js
+[] // Number('') = 0
[] + [] // '' + '' = ''
```
## 2.1 如同預期的結果

當您執行下面的範例，兩個陣列相加

```js
> [] + []
''
```

第一步使用 valueOf() 轉換兩個陣列 `[]` ，其會回傳陣列本身，因為還不是 Primitive 所以繼續使用 toString() 結果回傳一個空字串。
兩個空字串相加還是空字串。在只有單一 `+[]` 的狀況下 Javascript 會幫我們轉成數字 ToNumber()，一元運算子和二元的行為有些差異。

第二個例子我們相加陣列和物件

```js
> [] + {}
'[object Object]'
```

空物件跟陣列一樣 valueOf() 還是物件，然後 toString() 物件會轉換成 `[object Object]` 相加就是上面的結果。

```js
5 + new Number(7) // 12
6 + { valueOf: function () { return 2} } // 8
'abc' + { toString: function () { return 'def'} } // 'abcdef'
```

## 2.2 非預期的詭異結果

到這一步我們覺得已經掌握了 Javascript，但 Javascript 可怕的地方就是總是可以給您<del>驚喜</del>驚嚇。
當我們試著將兩個物件實字 `{}` 相加時

```js
> {} + {}
NaN
```

啥米鬼！？ 造成這個問題的原因是 Javascript 把第一個 `{}` 當作是 code block 並忽略它。這個結果等於 Javascript 只作 `+{}` 的運算。
上面有提過在 + 加號當作一元運算子的時候會嘗試把值轉成數字，下面就是等價執行過程：

```js
+{}
Number({})
Number({}.valueOf()) // 依然不是 Primitive 所以要繼續轉型
Number({}.toString())
Number('[object Object]')
NaN
```

那為什麼第一個 `{}` 會被解析成程式片段而不是一個物件實字呢？因為 Javascript 將其解析為一個 statement 。因此如果要處理這個問題我們可以透過 `()` 強迫 JS 將其視為 expression

```js
({} + {})
// [object Object][object Object]'
var o = {} + {}
o // '[object Object][object Object]'
```

其他還有一些技巧，如果您想知道更多細節請參考[重讀 Axel 的 Javascript 中的 Expression vs Statement 一文](https://segmentfault.com/a/1190000004565693)

經過了上面的解釋，我想您就不會很驚訝下面這段程式的結果了

```js
> {} + []
0
```

```js
+[]
Number([])
Number([].valueOf()) // 依然不是 Primitive 所以要繼續轉型
Number([].toString())
Number('')
0
```

有趣的是 Node.js 的 REPL 解析輸入的方式和 Firefox, Chrome 等瀏覽器不同，它會將下面的輸入解析成 expression

```js
> {} + {}
'[object Object][object Object]'
> {} + []
'[object Object]'
```

這個結果就好像把 input 放到 `console.log()` 的參數內一樣。

# 總結

在大多數的情況下，並不難理解 Javascript 加號的運作，您只能相加數字或字串。物件會被轉換成數字或字串。當然會造成混亂的還有一元運算與二元運算之間的行為差異這點值得注意一下，然後遵循上面談論的規則，相信您應該就能參透 Javascript 一些奇怪的行為。另外如果您想要合併陣列那您需要使用 `Array.concat([3, 4])`

```js
> [1, 2].concat([3, 4])
[1, 2, 3, 4]
```

如果是合併物件在快到 2017 的今天，您可以使用 `Object.assign` 或者其他函式庫例如 Underscore 等

```js
var o1 = { a: 1, b: 2}
var o2 = { c: 3, d: 4}
Object.assign(o1, o2)
o1
/**
 {a: 1, b: 2, c: 3, d: 4}
 */
```

# 參考

* [Fake operator overloading in Javascript](http://www.2ality.com/2011/12/fake-operator-overloading.html)
* [Javascript values: not everything is an object](http://www.2ality.com/2011/03/javascript-values-not-everything-is.html)
* [object plus object](http://www.2ality.com/2012/01/object-plus-object.html)

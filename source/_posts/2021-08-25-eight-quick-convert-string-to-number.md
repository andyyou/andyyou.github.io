---
title: JavaScript 8 種資料格式 4 種轉型數字方式
date: 2021-08-25 11:47:26
tags:
  - javascript
categories: Program
---

備忘範例

<!-- more -->

```js
// ~ Bitwise NOT Operator = ~N = -(N + 1)
// ~'abc' = -1
// ~~'abc' = 0


/* '1.1' */
+'1.1';           // 1.1
Number('1.1');    // 1.1
~~'1.1'           // 1
parseInt('1.1');  // 1


/* '1e1' */
+'1e1';             // 10
Number('1e1');      // 10
~~'1e1';            // 10
parseInt('1e1');    // 1


/* '1px' */
+'1px';           // NaN
Number('1px');    // NaN
~~'1px';          // 0
parseInt('1px');  // 1


/* 'p1' */
+'p1';          // NaN
Number('p1');   // NaN
~~'p1';         // 0
parseInt('p1'); // NaN


/* '010' */
+'010';           // 10
Number('010');    // 10
~~'010';          // 10
parseInt('010');  // 10 ECMAScript 3 值為 8 ，預設八進制 ES5 之後變更為 10 進制。


/* '0xf' */
+'0xf';           // 15
Number('0xf');    // 15
~~'0xf';          // 15
parseInt('0xf');  // 15


/* [1] */
+[1];           // 1
Number([1]);    // 1
~~[1];          // 1
parseInt([1]);  // 1


/* [1, 2] */
+[1, 2];            // NaN
Number([1, 2]);     // NaN
~~[1, 2];           // 0
parseInt([1, 2]);   // 1

// 只要 `+` 為 NaN， `~~` 為 0
// parseInt 則看起始字元是否為數值
```

### Falsy

```js
if (false)
if (null)
if (undefined)
if (0)
if (NaN)
if ('')
if ("")
if (``)
```
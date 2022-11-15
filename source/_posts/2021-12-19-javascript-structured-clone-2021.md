---
title: "[譯]在 JavaScript 使用 structuredClone 深度複製"
date: 2021-12-19 11:00:02
tags:
  - react
  - javascript
categories: Program
---


> 原文﹔[Deep-copying in JavaScript using structuredClone](https://web.dev/structured-clone/)

很長一段時間在 JavaScript 對於深度複製或說深拷貝我們都需要自己處理或者使用函式庫。現在平台支援了 `structuredClone` 

<!-- more -->

雖然在撰文的現階段各大瀏覽器的這個功能都還在試驗階段，但您可以持續觀察[Caniuse](https://caniuse.com/?search=structuredClone)。目前 Firefox 94 已經在正式版搭載，此外還有 Node 17，Deno 1.14 都支援了這個 API ，可以直接使用。

## 淺層複製

在 JavaScript 中複製值幾乎都屬於淺層複製/淺拷貝，和深度複製相反，意思是當您變更一個巢狀物件副本的值時(複製的物件屬性還包含著物件)會影響原本的物件屬性，反之亦然。因為複製的是參考。舉個例子我們常使用 [展開語法](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Spread_syntax) `...` 實作

```js
const original = {
  a: 'string',
  b: {
    num: 1,
    bool: true,
  },
};

const shallowCopy = {...original};
```

對複製物件的第一層一般屬性變更值不會有什麼問題

```js
shallowCopy.prop = 'a new prop';
console.log(original.prop); // undefined
```

但如果變更的屬性是物件的(巢狀物件)下一層屬性時就會影響

```js
shallowCopy.b.num = 2;
console.log(original.b.num); // 2
```

展開語法會遍歷物件屬性和值並將它們加入新建立的物件裡。因此達成了物件的複製，但關於值的部分卻有不同類型。原始值 (不屬於物件的資料類型 `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null`) 沒有問題。

但非原時值的資料如物件它們複製的時候是複製參考。也就造成上面說的會互相影響的問題。

## 深度複製

和淺層複製相反，深度複製會使用遞迴的方式處理，每一層的物件都會往下複製，直到下層沒有參考為止。這對於讓本尊和副本之間不要互相影響非常重要。在過去其實沒什麼很優的方式處理。大部分的人可能使用第三方函式庫例如  [Lodash 的 `cloneDeep()`](https://lodash.com/docs/#cloneDeep) 

或者

```js
const copy = JSON.parse(JSON.stringify(original));
```

事實上上面這個方法在過去甚至非常流行，因為效能好。但其實這招也是有缺點

* 遞迴資料結構﹔`JSON.stringify()` 無法處理遞迴資料結構(裡面包含了自己的參考)
* 內建型別﹔`JSON.stringify()` 無法處理 JS 一些內建型別例如 `Map`, `Set`, `Date`, `RegExp`, `ArrayBuffer`
* 函式﹔`JSON.stringify()` 無法處理函式

## 結構複製

實際上，平台本身就有很多地方需要深度複製例如﹔將資料儲存在 IndexedDB 時序列化和反序列化。利用 `postMessage()` 將資料傳給 Web Worker 等情境都需要類似的處理。而內部其實是使用一種稱為結構複製的演算法，過去這功能並沒有提供給開發者。

但現在我們可以使用 `structuredClone()` 了。

```js
const copy = structuredClone(original);
```



##  功能與限制

結構複製解決了大部分 `JSON.stringify()` 的問題，可以使用遞迴資料結構，JS 內建型別，效能也不錯。

但還是有些限制

* Prototypes﹔如果您使用 `structuredClone()` 複製某類別物件實例 Class Instance 您只會取得單純的物件不會包涵 `prototype` 的部分
* Function﹔如果您的物件包涵了函式則會被移除
* 不可複製﹔有些值是不可複製的例如 `Error` 和 DOM 節點

如果您的需求踩到上述的限制，建議您可以使用繼續 Lodash 來處理。

## 效能

目前尚未對其執行完整的性能測試，但之前 2018 年的時候曾經做過，當時 `JSON.parse()` 對於小型物件是快一點，預期結果應該是差不多的。但還是推薦使用 `structuredClone()`

## 結論

如果您需要深度複製資料您可以考慮嘗試使用 `structedClone()`
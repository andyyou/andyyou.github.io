---
title: "[譯] Vue.js 內部如何處理 computed property"
categories: Program
tags: [vuejs, javascript]
---


原文：[Vue.js Internals: How computed properties work](https://skyronic.com/blog/vuejs-internals-computed-properties)
在這篇翻譯文章中，我們將會透過撰寫一個簡單的實作來介紹關於 Vue.js 的 `computed properties` 是如何運作的。

<!--more-->

注意：

1. 這只是簡單的說明其運作方式，關於更複雜的物件與陣列的資料格式將不支援。
2. 這是在閱讀 Vue.js 原始碼之後得到的理解，根據我非常有限的理解，請注意內容可能大部分是錯誤的，如果您發現任何錯誤請糾正我們[作者 Email](skyronic@gmail.com)和[譯者小弟我](andyyu0920@gmail.com)。

# Javascript 屬性

首先要介紹的是 Javascript 有一個功能 - `Object.defineProperty` 。它有很多用途像是設定一些物件底層我們平常碰不到的屬性例如  `enumerable` 等等，但讓我先專注在其中一件事上

```js
var person = {}

Object.defineProperty(person, 'age', {
  get: function () {
    console.log('Getting the age')
    return 30
  }
})

console.log('The age is ', person.age)
```

每一次我們透過 `person.age` 取得資料的時候，雖然看起來我們像是在存取一個屬性但實際上我們是執行一個 function。

# Vue.js 觀察者模式的基礎

Vue.js 的一個基本核心功能就是讓我們可以將一般的物件套用上觀察者模式。下面就是一個極簡版的響應式屬性

```js
function defineReactive (obj, key, val) {
  Object.defineProperty(obj, key, {
    get: function () {
      return val
    },
    set: function (newVal) {
      val = newVal
    }
  })
}
```

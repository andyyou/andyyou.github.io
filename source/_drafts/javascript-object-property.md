---
title: 深入 ECMAScript 5 物件屬性
categories: Program
tags: [javascript]
---

ECMAScript 5 延續了 ECMAScript 3 增加了新的功能。在規範中也介紹了一些新的 API 其中相對實用的功能
莫過於對於物件屬性的改進。這些新的功能讓我們能夠更進一步的控制使用者使用物件的方式，例如：替物件增加 `getter`, `setter`
，設定可列舉屬性，限制動態 `刪除`, `增加` 屬性的用法，設定成唯讀物件等等。

而最棒的是上面說的這些功能大部分主流的瀏覽器都已經支援了，目前 ECMA-262 已經來到[第八版](http://www.ecma-international.org/ecma-262/8.0/index.html)。

<!--more-->

# Objects

從 ES5 開始我們可以設定物件是否可以被擴充，更具體的說就是設定物件能否加入新的屬性。

ES5 提供了兩個方法 `preventExtensions`，`isExtensible` 來支援這些功能。

```
var o = {}

Object.preventExtensions(o) // 傳回一個被無法擴充的物件
Object.isExtensible(o) // 傳回布林，該物件是否為可擴充物件
```

`preventExtensions` 會鎖定物件`從現在起不得再增加其他屬性`，但 `修改值`, `delete` 和 `Object.prototype`的異動仍可被執行。`isExtensible`只是單純用來判斷該物件是否有被鎖定擴充性。

```
var o = {}
o.name = 'Foobar'
console.log(o.name) // #=> Foobar
console.log(Object.isExtensible(o)) // #=> true

Object.preventExtensions(o)
o.url = 'http://www.google.com' // #=> TypeError in strict mode
console.log(Object.isExtensible(o)) // #=> false
```

# Properties 和 Descriptors

在支援新的方法之後屬性不再只是單純的鍵值對，我們可以完全的控制它們的行為。

這是因為物件中的每個屬性都有 `屬性描述器 Descriptor` 。根據描述器的設定，屬性會支援對應的行為，從此我們便可以控制屬性。進一步我們可以使用 `Object.defineProperty` 和 `Object.defineProperties` 設定屬性可否修改、刪除等。

在明白屬性與描述器之後，首先要特別提到的是，針對物件的屬性`取值`分成兩種

* 原本的直接取值
* 使用 getter, setter

```
var o = Object.defineProperty({}, 'name', {
  value: 'Foobar'
})
console.log(o.name) // #=> Foobar

var o = Object.defineProperty({}, 'name', {
  get: function () {
  	this.__name__ = this.__name__ || 'Foobar'
    return this.__name__
  },
  set: function (val) {
    this.__name__ = val
  }
})
console.log(o.name) // #=> Foobar

// 注意：兩種方式不能並存，如果兩種都設定會出現 TypeError: Invalid property descriptor. Cannot both specify accessors and a value or writable attribute
```

> 上面的 value, set, get 就是設定在屬性描述器上的。



除此之外，屬性的描述還支援

* writable: 可否寫入，即該屬性能否被修改
* enumerable: 是否可在 `for..in` 被列出
* configurable: 是否可 `delete` ，編輯描述器屬性

| 方法或描述/物件可否使用               | 編輯值  | 物件增加屬性 | delete 該屬性 | 物件 prototype 異動繼承   | 編輯 descriptor 或 重新定義 value |
| -------------------------- | ---- | ------ | ---------- | ------------------- | -------------------------- |
| Object.preventExtensions() | YES  |        | YES        | YES                 | YES (不得增加新屬性)              |
| 單一屬性 configurable: false   | YES  | YES    |            | YES                 |                            |
| 單一屬性 writable: false       |      | YES    | YES        | YES                 | YES                        |
| Object.seal                | YES  |        |            | 可新增但不能編輯已存在(含新增)的屬性 |                            |
| Object.freeze              |      |        |            | 可新增但不能編輯已存在(含新增)的屬性 |                            |


```
var o = Object.defineProperty({}, 'name', {
  value: 'Foobar',
  configurable: false
})

Object.defineProperty(o, 'name', {
  configurable: true
})
// #=> TypeError: can't redefine non-configurable property "name"
```

在 `descriptor` 總共有 5 個屬性可以使用其中 `value` 和 `get` `set` 只能擇一。

幾個重點：

* 當我們一般使用 `new` 關鍵字或字面值建立物件時 `writable`、`configurable`、`enumerable`預設為 `true`
* 當使用 `Object.defineProperty` 時`writable`、`configurable`、`enumerable`預設為 `false`
* 一旦 `configurable: false`就不能再重複定義

## Object.getOwnPropertyDescriptor

除了 `Object.isExtensions`如果我們想取得 `descriptor` 的資料也可以使用 `Object.getOwnPropertyDescriptor`

```
var o = Object.defineProperty({}, 'name', {
  value: 'Foobar'
})

var descriptor = Object.getOwnPropertyDescriptor(o, 'name')
console.log(descriptor) // #=> Object { value: "Foobar", writable: false, enumerable: false, configurable: false }

// 這個方法只能取得資料，修改取回的物件並不會實際影響原本屬性的設定
```

##  Object.defineProperty 和 Object.defineProperties

上面我們雖然開始使用 `Object.defineProperty` 但沒有對它有任何的介紹，這個方法的重點就是讓我們可以編輯設定 `descriptor`。接下來我們就直接用程式碼來觀察如何使用

```
var o = Object.defineProperty({}, 'value', {
  value: true,
  writable: false,
  enumerable: true,
  configurable: true
})

// 立即函式(IIFE) 是為了不讓 name 被其他地方存取
;(function () {
  var name = 'andyyou'
  Object.defineProperty(o, 'name', {
    get: function () {
      return name // 注意：這邊不可以使用 this.name 會陷入無限迴圈
    },
    set: function (val) {
      name = val
    }
  })
})()

console.log(o.value) // #=> true
console.log(o.name) // #=> andyyou

for (var prop in o) {
  console.log(prop)
}
// #=> value 
// 上面提到預設使用 Object.defineProperty 時 enumerable: false 所以不會列舉出來

o.value = false // 嚴格模式下 TypeError

// Object.defineProperties 使用方式
Object.defineProperties({}, {
  'value': {
    value: true,
    writable: false
  },
  'name': {
    value: 'andyyou',
    writable: false
  }
})
```

關於屬性描述和相關 API 的支援大概是 ES5 最重要的更新，它讓我們可以有更多的彈性和作法。

## Object.keys

這個方法會回傳物件內 `enumerable: true` 的屬性，不包含 `prototype` 就是繼承來的屬性，和 `for..in` 的結果一致。

```
var o = {
  a: 1
}
Object.defineProperties(o, {
  b: {
    value: 2,
    enumerable: true
  },
  c: {
    value: 3 // 預設 enumerable 是 false
  },
  d: {
    value: 4,
    enumerable: false
  }
})

console.log(Object.keys(o)) // #=> ['a', 'b']
```

## Object.getOwnPropertyNames

類似於 `Object.keys` 但它列出的是只要是定義在該物件上的屬性都列出來不管 `enumerable`，意思是只要不是繼承（從 `prototype` 找到的屬性）來的屬性都列出。

```
var o = {
  a: 1
}
Object.defineProperties(o, {
  b: {
    value: 2,
    enumerable: true
  },
  c: {
    value: 3 // 預設 enumerable 是 false
  },
  d: {
    value: 4,
    enumerable: false
  }
})

console.log(Object.getOwnPropertyNames(o)) // #=> ['a', 'b', 'c', 'd']
```

## instance.propertyIsEnumerable('property_name')

```
var o = {
  name: 'foobar'
}

console.log(o.propertyIsEnumerable('name')) // #=> true
```

## `prototype` 與 `__proto__`

`__proto__` 屬性就是原型鍊實際在找尋所謂的繼承屬性的地方，`prototype` 則是`建構函數`在使用的，即函式的一個屬性。建構函數在建立物件時會把自己的 ` prototype` 安裝到新建物件的 `__proto__`。

觀察下面程式碼：

```
function Car () {}

var car = new Car() // 使用建構函式建立物件

console.log(car.__proto__ === Car.prototype) // #=> true

// car.[[prototype]] 為 car.__proto__ getter 取得的 private 內部屬性
```

* 一般物件實字沒有 `prototype`
* 內建的 `Object`，`Array`，`Number` 等 9 種建構函式也都是函式物件

## Object.getPrototypeOf

除了上面的 `__proto__`之外也有一個新的方法用來取得內部的 `[[Prototype]]`屬性。

```
function Dog () {}

var dog = new Dog()

Object.getPrototypeOf(dog) // #=> function Dog ()
Object.getPrototypeOf(dog) === dog.__proto__ // #=> true
```

## Object.create(prototype, props)

建立一個新的物件，其 `新物件.__proto__` 等於第一參數傳入的值，`新物件.prototype` 則會等於傳入參數值的 `prototype`，也就是說如果傳入一個 `function` 就會有 `prototype`，一般物件實字則是 `undefined`。

第二個參數`props` 則相同於 `Object.defineProperties` 的使用方式

```
// 觀察不同類型的值建立的結果
var javascript = {creator: 'Brendan Eich'}
function Ruby () {
  this.creator = 'Matz'
}
var ruby = new Ruby()

var a = Object.create(javascript, {
  ext: 'js'
})
var b = Object.create(Ruby, {
  ext: 'rb'
})
var c = Object.create(new Ruby(), {
  ext: 'rb'
})
console.log(a.prototype, a.__proto__) // #=> undefined, {creator: 'Brendan Eich'}
console.log(b.prototype, b.__proto__) // #=> Ruby.prototype object, function Ruby ()
console.log(c.prototype, c.__proto__) // #=> undefined, {creator: 'Matz', constructor: function Ruby ()}

// 結論：通常我們不會使用第二種傳入 function 的方式，這麼作純粹是觀察 prototype 和 __proto__ 的差異
// 第一種方式傳入物件，如果要替全部產生的物件增加屬性則
javascript.version = 'ECMAScript-262 No 8'
console.log(a.version) // #=> ECMAScript-262 No 8

// 第三種方式使用建構函數建立物件，如果要替全部產生的物件增加屬性則
Ruby.prototype.version = '2.4.1'
console.log(c.version) // #=> 2.4.1
```

## Object.seal 和 Object.isSealed

`Object.preventExtensions()` 加上所有屬性鎖定 `configurable: false`。

>  物件本身不能擴充，但 prototype 可以

```
var o = {
  name: 'Foobar'
}

Object.isExtensible(o) // #=> true
Object.seal(o)
Object.isExtensible(o) // #=> false
Object.isSealed(o) // #=> true
Object.getOwnPropertyDescreptor(o, 'name') // #=> { value: "Foobar", writable: true, enumerable: true, configurable: false }

// 繼承來的屬性可新增，但不能編輯
function Car () {
  this.name = 'Benz'
}
var car = new Car()
Object.seal(car)
Car.prototype.wheel = 4
console.log(car.name, car.wheel) // #=> Benz, 4
car.name = 'Hyundai'
car.wheel = 5 // 繼承來的屬性可新增，但不能編輯
console.log(car.name, car.wheel) // #=> Hyundai, 4
```

## Object.freeze 和 Object.isFrozen

`Object.preventExtensions()` 加上所有屬性鎖定 `configurable: false`， `writable: false`。

> 物件本身不能擴充，但 prototype 可以

```
var o = {
  name: 'Foobar'
}

Object.isExtensible(o) // #=> true
Object.freeze(o)
Object.isExtensible(o) // #=> false
Object.isSealed(o) // #=> true
Object.isFrozen(o) // #=> true
Object.getOwnPropertyDescreptor(o, 'name') // #=> { value: "Foobar", writable: true, enumerable: true, configurable: false }
// 繼承來的屬性可新增，但不能編輯
function Car () {
  this.name = 'Benz'
}
var car = new Car()
Object.freeze(car)
Car.prototype.wheel = 4
console.log(car.name, car.wheel) // #=> Benz, 4
car.name = 'Hyundai'
car.wheel = 5
console.log(car.name, car.wheel) // #=> Benz, 4
```

一個 seal 後的物件僅能修改現有的值，一個 freeze 後的物件也屬於 `isSealed` ，總結當 `configurable: false` 加上不能擴充 `isExtensible => false`等於 `isSealed => true` 。加上 `writable: false` 變成唯讀物件 `isFrozen => true`。

參考上面整理的表格希望能夠協助您釐清這一系列的 API 關係與產生效果的差異。
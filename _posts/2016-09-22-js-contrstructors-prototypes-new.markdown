---
layout: post
title: '筆記 Javascript 的 constructors, prototypes, new 關鍵字'
date: 2016-09-22 12:00:00
categories: js
---

您是否曾困惑關於 Javascript 中的 `new` 關鍵字呢？是否曾想理解關於 function 和 constructor 的不同是什麼？
還有 `prototype` 到底是用來幹嘛的？

這篇文章會試著讓您釐清這些東西。

網路上已經有許多討論關於 Javascript 非典型物件導向(pseudo-classical)的說明 - [MDN](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript)
。您可以閱讀一下釐清這些術語。

大多數 Javascript 的新進開發者不太想要使用 `new` 關鍵字，因為這會讓程式碼寫起來像是 Java 並且在使用上造成一點混亂。
在這邊我並不想討戰也不會偏袒任何一種方式，我們只是要解釋它是如何運作的。我要說的只是這是個工作，如果在實務上能幫上忙就用吧！

# constructor 是什麼？

constructor 翻為建構子但為了讓您之後更好理解，這邊會直接使用 constructor。簡單說這和您之前學過的大部分物件導向語言不同，在 Javascript 中任何一個函式(function)都可以被當作 constructor。Javascript 並沒有明顯的區別兩者，也就是說 function 可以被當作 constructor 或者當作一般函式調用。

而 constructor 的用法就是 function 搭配 `new` 關鍵字：

```js
var Vehicle = function Vehicle () {

}

var vehicle = new Vehicle()
```

這可能會讓許多慣於使用 C++ 或 Java 的人感到困惑。但其實，在某種程度上你可以把 function 看做是 class。

# 當 constructor 被調用時發生了什麼事？

當 `new Vehicle()` 的時候，Javascript 做了 4 件事情

1. 產生一個新的物件
2. 將這個新物件的 `constructor` 屬性設為 `Vehicle` 這個 Function 物件
3. 讓這個物件繼承 `Vehicle.prototype` (Function 物件)
4. 在這個物件的 context 執行 `Vehicle()`

> 用個簡短的範例說明當我們在 Javascript 執行一個 foo() 時，一般來說它等於 `window.foo()`。這個 `window` 就是 context。上面的例子可以看成：Vehicle.call(新物件)

再看一次：

### 1. 產生一個新的物件

這一步沒有任何特別的，就只是建立一個新物件 `{}`

### 2. 將這個新物件的 `constructor` 屬性設為 `Vehicle` 這個 Function 物件

這步意味了 2 件事

```js
vehicle.constructor == Vehicle // true
vehicle instanceof Vehicle // true
```

這個 constructor 並不是一般的屬性，當我們列舉所有屬性時並不會出現。另外，我們可以試著對它設值，不過只是在這個特殊屬性之上再加上一個普通的屬性

```js
vehicle // {}

var Beer = function Beer() {}

vehicle.constructor = Beer
vehicle // {constructor: function Beer()}
vehicle.constructor == Beer // true
vehicle instanceof Beer // false
vehicle instanceof Vehicle // true
```

可以想成這時有兩個 constructor 一個是我們加上的`普通屬性`，一個是`底層屬性`。底層內建的 constructor 我們是無法變更的。
它是用來搭配 `new` 使用的。

### 3. 讓這個物件繼承 `Vehicle.prototype` (Function 物件的 prototype)

到這一步事情變得有趣了

一個 function 本質上是一個特殊的物件，並且就像其他物件一樣。它也是有屬性的。
而這個 `Function 物件` 預設會自動取得一個叫 `prototype` 的屬性，是一個空物件。這個物件會經過一些特別的處理。

當我們透過 `constructor` 的方式建立一個物件`時`，它會繼承這個 `prototype 物件` 的所有屬性。

讓我們來看一點程式碼理解一下

```js
Vehicle.prototype.wheel = 4
var v = new Vehicle()
v.wheel // 4
```

這個剛產生的 `v` 物件會從 Vehicle 的 prototype 中擷取 `wheel`。

看起來像是把屬性複製過去，但這個`繼承`的行為不只是單純的複製屬性到新物件上。這個物件代理了 `prototype` 所有的屬性。
也就是說即使在建立之後，`Vehicle Function 物件` 的 `prototype` 加了新東西，這個物件還是能夠取得。

```js
Vehicle.prototype.wheel = 6
Vehicle.prototype.brand = 'BMW'

v.wheel // 6
v.brand // BMW
```

重點來了，我們還是可以單獨覆寫 instance 物件實例的屬性，這麼做並不會異動 prototype 的屬性

```js
v.wheel = 8
v.wheel // 8
(new Vehicle()).wheel // 6
```

當然我們也可以有 `method` 的使用方式。在屬性上的 function 一般稱為方法(method)。

```js
Vehicle.prototype.go = function go() {return 'run'}
vehicle.go() // run
```

### 4. 在這個物件的 context 執行 `Vehicle()`

最後，constructor 會自己呼叫自己。這個 function 裡面的 `this` 會指向我們剛剛建立的物件。

> 探討 this, context, scope 已經超出本文範圍，可以參考[understanding scope and context in- javascript](http://ryanmorr.com/understanding-scope-and-context-in-javascript/)

所以我們可以這麼做

```js
var Vehicle = function Vehicle(color) {
  this.constructor
  this.color = color
}

(new Vehicle('blue')).color // blue
```

> 備註：上面提到使用 new 關鍵字搭配 constructor 回傳一個新的物件，本質上是對的。但如果這個 constructor 回傳了一個其他物件那麼原本預計建構的物件會被拋棄，只回傳該內容。詳細的說明請參考[javascript constructor value](http://www.neilstuff.com/winhttp.html#javascript-constructor-value)

# 小結

這種方式讓我們可以實作類似 Class 的用法

```js
// Class
var Vehicle = function Vehicle(color) {
  // Initialization
  this.color = color
}

// Instance Methods
Vehicle.prototype.go = function () {
  return 'run'
}
```

# 子類別

這種非典型物件導向得設計，或稱 `Prototype-based programming` 並沒有標準的方式實作子類別。
不過我們可以透過設定一個父類別的實例物件到 `prototype` 來達成這個功能。

> 也由於這種設計，所以不會有解構子 destructors

```js
var Car = function Car() {}
Car.prototype = new Vehicle('blue')
Car.prototype.honk = function honk() {return 'BEEEE!'}

var car = new Car()
car.honk() // 'BEEEE'
car.go() // run
car.color // blue
car instanceof Car // true
// 稍早我們把 Beer 設到 constructor 會是 false
car instanceof Vehicle // true
```

還有一個問題，由於 Vehicle constructor 只有在設定 Car.prototype 時被呼叫一次。我們會在設定時給予 color 參數。
於是我們就不能在初始化的階段給不同的物件不同的顏色。但這個問題有些 Javascript 框架已經實作它們的解決方案。

# 類似於 mixin 的作法

有時您並不想使用類別的概念，我們只想要某個物件能夠取得另一個物件的屬性並能夠覆寫。

function 可以協助我們完成這個任務。通常這類 function 會被命名為 `create` 或 `clone`

```js
function create (parent) {
  var fn = function () {}
  fn.prototype = parent
  return new fn()
}

var company = {brand: 'Benz'}
var car1 = create(company)
var car2 = create(company)
var car3 = create(company)
car3.brand = 'Audi'

car1.brand // Benz
car2.brand // Benz
car3.brand // Audi

company.brand = 'Toyota'
car1.brand // Toyota
car2.brand // Toyota
car3.brand // Audi
```


# 總結

關於 Javascript 的 prototype 相較於其他語言有點混亂難理解。即便讓語法看起來像是一般OOP但您還是需要掌握一些技巧並謹慎使用。

* function 可當作 class 看待
* 在 Javascript 中任何一個函式(function)都可以被當作 constructor。Javascript 並沒有明顯的區別兩者，也就是說 function 可以被當作 constructor 或者當作一般函式調用。
* class 所建立的 instance 屬性使用 `prototype` 加入繼承
* constructor instance 沒有該屬性時會代理 prototype 的，但覆寫後就為自己個人的屬性
* constructor instance 與 Function.prototype 是代理關係，也就是 prototype 異動時，物件的屬性會跟著更新(沒有覆寫的狀況下)

# 參考資源

* [javascript constructors prototypes and the new keyword](https://blog.pivotal.io/labs/labs/javascript-constructors-prototypes-and-the-new-keyword)
* [Introduction to Object Oriented JavaScript](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript)
* [understanding scope and context in javascrip](http://ryanmorr.com/understanding-scope-and-context-in-javascript/)
* [javascript constructor return value](http://www.neilstuff.com/winhttp.html#javascript-constructor-value)

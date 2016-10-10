---
layout: post
title: '認識 ES6 Decorator '
date: 2015-10-15 05:30:00
categories: Program
tags: [es6, javascript]
---

在這篇文章中我們將要探討如何使用 ES7 的新功能.

ES6 新增了一個簡單更具可讀性的語法讓我們可以建立類別(class). 搭配 ES6 匯入匯出模組的語法讓我們的程式更加清楚易懂.

而 Decorators 讓我們可以在設計時期透過註記的方式修改類別與屬性.
在 ES5 物件實字(Object Literal)支援`值`可以使用任意的表達式(Expression)
而 ES6 類別單純只支援`值`使用函式表達式或稱作函式常量(Function Literal)
現在 Decorator 讓 JS 具備了可維護性與可讀性的宣告式語法
<!--more-->
> `Object Literal` 是一個透過 `,` 逗號分隔的鍵值對列表, 再透過一個大括號包起來.

```
// object literal 又稱物件實字
let obj = {
  name: 'andyyou',
  age: 28,
}
```

> `Function Literal` 函式表達式或稱作函式常量 - 即定義一個不具名的 function, 單純觀察下面的範例會覺得其類似於一個 function 得宣告片段, 除了語法看起來像一段表達式和並沒有宣告函式名稱

```
// Function Literal
var func = function () {
  console.log('I am a function');
}

// 具名 Function
function F() {
  console.log('I am a function');
}
```

簡單說就是我們會對 `class` 或 `property` 使用 `decorator`

# 何謂 Decorators?

如果你不熟悉 `Decorator`, 他們類似於一種標記或描述資料(metadata), 不過不同的是它們會被附加套用在類別, 方法, 或者屬性上.
概念上就是你可以透過附加的方式來操作`定義`. 這麼說非常抽象讓我們看看範例:

```
// 可否測試
function Testable(target) {
  target.isTestable = true
}

@Testable
class OurClass {

}

// 接著我們就可以確認這個類別是否可以測試
OurClass.isTestable // => true
```

上面那個 `@Testbale` 就是 `decorator` 語法, 我們用白話文來說就是透過像加註解或標記的方式讓類別或屬性增加其他功能.
如果我們想要傳入參數也就是一個動態的`註記`函式, 我們可以透過 `factory pattern`

```
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable
  }
}

@testable(true)
class OurClass {

}

console.log("OurClass: ", OurClass.isTestable); // => OurClass: true
```

小結來說一個 `decorator` 是

* 一個表達式
* 一個 function
* 可以在其參數中取得目 target, name, property, descriptor
* 選擇性的回傳一個 descriptor 用來安裝在目標物件上

# 透過宣告的方式加入 Mixins

在程式語言的概念中 Mixin 是我真的覺得好用的功能(事實上在這裡它不叫 Mixin 稱為 traits)
因為在大多的開發過程我們只是要把一些`功能`, `方法`抽出來重複使用, 實務上常遇到不好決定到底該歸納到什麼類別. 如果你要說介面(Interface)那又扯遠了

簡單的說 mixin 就是把一些功能抽出去獨立一個模組, 根據不同語言其本質可能是另一個類別或函式等等.
所以我們需要的其實就是如何把這些函式或者方法抽出去和合併的方式.
在 ES5 時我們可以使用 `Object.assign` 來合併(merge) `prototype`, 背地裡其實是 Polyfill 例如使用 underscore 或 lodash 的 _.extend 實作.

```
$ npm i -S object.assign
```

```
var assign = require('object.assign');
function CarAbility() {};

CarAbility.prototype.run = function () {
  console.log('Car is running');
}

function ToyotaCar() {

}

assign(ToyotaCar.prototype, CarAbility.prototype);

var car = new ToyotaCar();
car.run(); // => Car is running
```

而在 ES6 當我們透過新的語法 `class` 來定義類別的時候, 我們並不能簡單的使用 prototype 當做 mixin. 且預設所有的方法(Medthod)並不是 `enummerable` 這是 JS 中物件底層的屬性
簡單的說就是 `for in` 會不會將其列舉出來. `Object.assign` 只會合併那些物件中 `enummerable` 的方法
此時我們可以改用物件來作為 mixin 接著透過 `Object.assign` 將其套用到目標類別上

```
const CarAbility = {
  run() {
    console.log('foo')
  }
}

class BMWCar {

}

Object.assign(BMWCar.prototype, CarAbility)

let car = new BMWCar();
car.run();
```

這是一種指令式的風格, 那如果我們可以在宣告類別的時候合併呢? 讓我們來建立一個簡單的 decorator 來完成這個需求

```
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}
```

然後我們就可以使用 ES6 的匯入語法

> 注意從這邊開始因為使用了 ES6 的語法所以你會需要 Babel 來編譯

```
import {mixins} from './mixins'

const CarAbility = {
  run() {
    console.log('I am running');
  }
}

@mixins(CarAbility)
class AudiCar {

}

let car = new AudiCar();
car.run();
```

# Traits

既然我們已經可以透過上面的方式實作 mixin, 那為什麼還要有個 Traits, 原因是有些情況您需要更多的控制權, 當你只想合併想要的功能時, 或者遇到不同物件卻有相同名稱的函式的狀況.
`Traits` 讓我們可以避免合併時的名稱衝突, 我們可以排除方法或者是透過 alias 別名的方式改變方法的名稱

關於實作面, 我們可以透過[CocktailJS](http://cocktailjs.github.io/)這個函式庫輕鬆實作 Traits, 這個函式庫包含了 `annotations`, `traits` 以及讓 class 具備更多陳述式的用法(或者說從外部合併屬性或方法的寫法)

這裡為了單純只說明觀念我們只使用[traits-decorator](https://github.com/CocktailJS/traits-decorator), 它是一個實驗 decorator 和 bind-operators 的函式庫

```
import {traits} from 'traits-decorator';

// 使用 class 為 traits
class CarTraitClass {
  run () {
    console.log('I am running');
  }
}

// 使用 object 為 traits

const CarTraitObject =  {
  start () {
    console.log('The car is started');
  }
}

@traits(CarTraitClass, CarTraitObject)
class HondaCar {

}

let car = new HondaCar();
car.run(); // => I am running
car.start(); // => The car is started
```

再次提醒如果你用 babel 實作需注意目前 `@` 語法要啟用 `experimental` 即 `stage 0`

```
$ babel [source].js -o [destination].js --stage 0
```

# 名稱衝突

剛剛上面有提到關於衝突的部分, 假設我們現在遭遇到 traits 甚至是類別本身的方法名稱產生衝突的狀況, 就是名字一樣

```
import {traits} from 'traits-decorator';

// 使用 class 為 traits
class CarTraitClass {
  run () {
    console.log('[class] I am running');
  }
}

// 使用 object 為 traits

const CarTraitObject =  {
  run () {
    console.log('[object] I am running');
  }
}

@traits(CarTraitClass, CarTraitObject)
class HondaCar {

}

let car = new HondaCar();
car.run();
```

重新編譯在執行會產生錯誤(編譯時不會出錯)

```
throw new Error('Method named: ' + methodName + ' is defined twice.');
```

很明顯的是因為我們的 `run` 定義了 2 次, 此時 Traits 讓開發者負責去解決這個衝突, 這部分和 mixin 不太一樣, mixin 的話會讓後面載入的方法覆寫掉前面的.

為了解決這個問題我可以排除我們不想要的 method 或者為其創造一個別名

```
import {traits, excludes} from 'traits-decorator';

// 使用 class 為 traits
class CarTraitClass {
  run () {
    console.log('[class] I am running');
  }
}

// 使用 object 為 traits

const CarTraitObject =  {
  run () {
    console.log('[object] I am running');
  }
}

// 別名的用法
// @traits(CarTraitClass, CarTraitObject::alias('drive'))
@traits(CarTraitClass, CarTraitObject::excludes('run'))
class HondaCar {

}

let car = new HondaCar();
car.run(); // => [class] I am running
```

您可能注意到一個奇怪的小東西 `::` 這是 `bind-operator` 基本上 bind operator 就是 `.bind()` 的縮寫, `::this.method` = `this.method.bind(this)`
`::Car.run` 會等於 `Car.run.bind(Car)`

而 `::` 和 `@` 這些語法在 Babel 都還處於實驗階段, 所以如果您要使用就必須要自己設定開啟 `stage 0`

# 總結

透過這種方式我們說我們可以在`設計時期`就可能是類別已經寫好了的情況下透過註記來增加功能, 不要用的時候也可以輕易移除.


# 參考

* [ES 7 descorator](http://elmasse.github.io/js/decorators-bindings-es7.html)
* [javascript-decorators](https://github.com/CocktailJS/traits-decorator)
* [exploring-es7-decorators](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841)

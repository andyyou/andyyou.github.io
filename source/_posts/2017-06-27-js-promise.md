---
title: Promise 學習筆記
tags:
  - javascript
categories: Program
date: 2017-06-27 17:53:32
---


這篇文章主要是為了複習與更加深入掌握 `Javascript Promise` 而產生的筆記。在直接閱讀關於 Promise A+ 或原理說明之前先通過比喻的方式理解可以更加深記憶。如果您對於 Promise 還是有點似懂非懂，不妨可以看看。

<!--more-->

# 理解 Promise

Promise 簡單來說：

想像一下你是個孩子，你媽`承諾(Promise)`你下個禮拜會送你一隻新手機。

現在你並不知道下個禮拜你會不會拿到手機。你媽可能真的買了新手機給你，或者因為你惹她不開心而取消了這個承諾。

這就是一個 `Promise`，一個 Promise 有三種狀態：

1. `pending` 未發生、等待的狀態。到下週前，你還不知道這件事會怎樣。
2. `resolved` 完成/履行承諾。你媽真的買了手機給你。
3. `rejected` 拒絕承諾。沒收到手機，因為你惹她不開心而取消了這個承諾。

總結來說狀態有 `等待`、`成功`、`失敗`，使用情境就是我們`當前還不知道結果，需要等待結果發生才繼續後續的處理`。

# 建立一個 Promise

看完上面的比喻讓我們對應到 Javascript。

```
var isMomHappy = false

var willIGetNewPhone = new Promise(function (resolve, reject) {
  if (isMomHappy) {
    var phone = {
      brand: 'Samsung',
      color: 'black',
      type: 's8'
    }
    resolve(phone)
  } else {
    var reason = new Error('Mom is unhappy')
    reject(reason)
  }
})
```

上面這段程式碼應該已經充分解釋概略的觀念。

* 第一行我們使用一個 Boolean `isMomHappy` 定義媽媽是否開心。
* 我們宣告一個 Promise `willIGetNewPhone`。這個 Promise 可能是被`履行(resolved)`又或者`拒絕(rejected)`。
* Promise 標準的語法可以參考 [MDN](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise)

```js
new Promise(function (resolve, reject) {})
```

* 我們需要記得的是如果一個 Promise 執行成功要在內部 function 呼叫 `resolve(成功結果)`，如果結果是失敗則呼叫 `reject(失敗結果)`。在我們的範例中如果媽媽開心，我們將得到手機因此我們執行 `reslove(phone)`，如果媽媽不高興則執行 `reject(reason)`。

> 一個 Promise 物件表達的是一件非同步的操作最終的結果，可以是`成功`或`失敗`。

# 使用 Promise

到這一步我們已經有了一個 Promise，讓我們接著來使用它。

```js
var askMom = function () {
  willIGetNewPhone
    .then(function (fulfilled) {
      console.log(fulfilled)
    })
    .catch(function (error) {
      console.log(error.message)
    })
}
askMom()
```

1. 首先我們有個 function 叫 `askMom` 在這個 function 中，我們將利用 Promise `willIGetNewPhone`。
2. 我們希望一旦`等待的結果發生時`可以採取對應的動作，我們可以使用 `.then` 或 `.catch` 來執行對應的行為。
3. 在這個範例中，我們在 `.then` 中使用 `function (fulfilled){}`，而這個 `fulfilled` 就是從 Promise 的 `resolve(成功結果)` 傳來的結果，範例中這個結果就是 `phone` 物件。
4. 在 `.catch` 中我們使用了 `function (error) {}`。而這個 `error` 就是從 Promise 的 `reject(失敗結果)` 傳來的即 `reason`。

* [範例](https://jsfiddle.net/3jkev2w1/1/)

# 鏈式調用 Promise

Promise 是可串連的。

假如你`承諾`您的朋友，如果你拿到新手機會借他們看看。這又是另一個 `Promise`。讓我們繼續來撰寫這個範例：

```js
var showOff = function (phone) {
  return new Promise(function (resolve, reject) {
    var message = 'Hey friend, I have a new ' + phone.color + ' ' + phone.brand + ' phone' + phone.type

    resolve(message) 
  })
}
```

1. 在這個範例，您可能發現到我們根本沒有呼叫 `reject`，這是可選的，我們可以省略不調用。
2. 另外，我們可以透過使用 `Promise.resolve` 簡化這個範例。

```js
var showOff = function (phone) {
  var message = 'Hey friend, I have a new ' + phone.color + ' ' + phone.brand + ' phone ' + phone.type

  return Promise.resolve(message)
}
```

接著讓我們來看看如何串連 `Promise`。在 `willIGetNewPhone` 這個 Promise 之後接續 `showOff` Promise。

```js
var askMom = function () {
  willIGetNewPhone
    .then(showOff)
    .then(function (fulfilled) {
      console.log(fulfilled)
      // 'Hey friend, I have a new black Samsung phone s8'
    })
    .catch(function (error) {
      console.log(error.message)
    })
}
```

這就是 `Promise` 串連的方式。

# 非同步

`Promise` 是非同步的，讓我們在呼叫 Promise 的前後加上 console.log

```js
var askMom = function () {
  console.log('before asking Mom')

  willIGetNewPhone
    .then(showOff)
    .then(function (fulfilled) {
      console.log(fulfilled)
    })
    .catch(function (error) {
      console.log(error.message)
    })
  
  console.log('after asking Mom')
}
```

關於上面這段程式碼，您認為 log 的順序會如何？大概我們會猜

```
1. before asking Mom
2. Hey friend, I have a new black Samsung phone s8
3. after asking Mom
```

然而真正的順序是

```
1. before asking Mom
2. after asking Mom
3. Hey friend, I have a new black Samsung phone s8
```

為什麼？因爲時間是不等人的 XD。這也是我們在寫 JS 的時候非常常見得情況。

想像您還是那個孩子，在你媽決定買手機給你之前難道你就一直待在那等結果嘛！？你應該是跑去玩了吧！
這就是我們說的`非同步`，程式碼不會卡在那等待結果。如果需要等待結果才能繼續處理的部分，我們就把它方到 `then` 裡面。

# ES5， ES6/ES2015，ES7/Next 的 Promise

## ES5

目前主流瀏覽器使用的 JS 版本。上面的範例如果想在全部的瀏覽器運行我們需要加入 Promise 函式庫 - [Bluebird](http://bluebirdjs.com/docs/getting-started.html)。這是因為 ES5 內建不支援 Promise，當然時至今日已經有許多瀏覽器陸續支援了。

## ES6/ES2015

Nodejs v6+ 開始原生支援 Promise，此外還有 `arrow function`、`const` 和 `let`

讓我們來看看 ES6 的程式碼


```
const isMomHappy = true

const willIGetNewPhone = new Promise((resolve, reject) => {
  if (isMomHappy) {
    const phone = {
      brand: 'Samsung',
      color: 'black',
      type: 's8'
    }
    resolve(phone)
  } else {
    const reason = new Error('Mom is not happy)
    reject(reason)
  }
})

const showOff = function (phone) {
  const message = 'Hey friend, I have a new ' + phone.color + ' ' + phone.brand + ' phone ' + phone.type
  return Promise.resolve(message)
}

const askMom = function () {
  willIGetNewPhone
    .then(showOff)
    .then(fulfilled => console.log(fulfilled))
    .catch(error => console.log(error.message))
}

askMom()
```
看到所有的 `var` 被換成 `const` 然後 `function` 換成 `(resolve, reject) => ` 關於更深入的介紹可以參考

* [JavaScript ES6 Variable Declarations with let and const](https://strongloop.com/strongblog/es6-variable-declarations/)
* [An introduction to Javascript ES6 arrow functions](https://strongloop.com/strongblog/an-introduction-to-javascript-es6-arrow-functions/)


## ES7

到了 ES7 更引進 `async await` 讓處理`非同步`的語法更加簡潔與具備可讀性。 

```js
const isMomHappy = true

const willIGetNewPhone = new Promise((resolve, reject) => {
  if (isMomHappy) {
    const phone = {
      brand: 'Samsung',
      color: 'black',
      type: 's8'
    }
    resolve(phone)
  } else {
    const reason = new Error('Mom is not happy')
    reject(reason)
  }
})

async function showOff(phone) {
  return Promise((resolve, reject) => {
    const message = 'Hey friend, I have a new ' + phone.color + ' ' + phone.brand + ' phone ' + phone.type
    return resolve(message)
  })
}

async function askMom () {
  try {
    console.log('before asking Mom')
    let phone = await willIGetNewPhone
    let message = await showOff(phone)
    console.log(message)
    console.log('after asking mom')
  } catch (error) {
    console.log(error.message)
  }
}

(async () => {
  await askMom()
})()
```

1. 當我們要在一個 function 中回傳 Promise 時，我們需要在該 function 前加上 `async` 
2. 當我們使用一個 Promise 或者 function 回傳 Promise （async）時需要使用 `await` 例如 `let phone = await willIGetNewPhone` 或 `let message = await showOff(phone)`
3. 使用 `try {} catch (error) {}` 來攔截例外，即 Promise `reject` 時。

# 為何及何時使用 Promise

在我們概略的介紹之後您肯定有些疑問，首先是為什麼我們需要 Promise？以及在沒有 Promise 之前的世界又是怎麼樣。在回答這些問題之前讓我先談談一些基礎的東西

## Function 與 Async Function

讓我們先來看看範例：使用函式來加總兩個數字

```js
// 普通的函式加總兩數
function add (num1, num2) {
  return num1 + num2
}

const result = add(1, 2)
// result 為 3
```

```js
// 呼叫遠端函式加總兩數
const result = getAddResultFromServer('http://example.com?num1=1&num2=2')

// result 會是 undefined
```

如果我們在本地端加總兩數，基本上我們會立刻得到其值，然而如果是透過遠端執行（呼叫 API)則需要等待其回應，不會立刻得到值。
或者這麼說，我們其實不知道 server 需要多久時間，甚至是否正常運作等等，這種情況下我們又不希望介面操作卡在這邊等待。於是像調用 API，下載檔案，讀檔等這些會需要花點時間的操作，在 JS 中通常都是使用非同步的機制來處理。

在 Promise 出現之前：

難道我們一定要用 Promise 來處理非同步嗎？不是，在 Promise 出現之前我們已經有個方式處理 - Callback。所謂的 Callback 就是一個 function ，當我們調用非同步的 function 時，我們把它當作參數一起傳出去，當非同步的處理完成後，會把結果丟進這個 function 繼續處理。讓我們來看看範例：

```js
function addAsync (num1, num2, callback) {
  return $.getJSON('http://example.com', {
    num1: num1,
    num2: num2
  }, callback)
}

addAsync(1, 2, success => {
  const result = success // 取得結果為 3
})
```

目前這個範例看起來還OK啊，為什麼我們需要 Promise？

# 假如我們需要連續處理多個非同步的行為？

現在假如我們不是只有相加兩數，我們想要相加 3 次，如下這個情況

```js
let resultA, resultB, resultC

function add (num1, num2) {
  return num1 + num2
}

resultA = add(1,2)
resultB = add(resultA, 3)
resultC = add(resultB, 4)

console.log('total', resultC)
console.log(resultA, resultB, resultC)
```

如果是上面這個情況使用 callback 呢？

```js
let resutlA, resultB, resultC

function addAsync (num1, num2, callback) {
  return $.getJSON('http://example.com', {
    num1: num1,
    num2: num2
  }, callback)
}

addAsync(1, 2, function (success) {
  resultA = success

  addAsync(resultA, 3, function (success) {
    resultB = success

    addAsync(resultB, 4, function (success) {
      resultC = success

      console.log('total', resultC)
      console.log(resultA, resultB, resultC)
    })
  })
})
```

這樣不斷內嵌的語法看起來就不是這麼友善了，在實務上我們非常容易遇到這個情形，例如：需要先使用 API 取得帳號資料，接著該帳戶的訂單，再取出某筆訂單的詳細資料。像上面這樣的程式碼通常人們稱為 `callback hell` 因為我們需要不斷的內嵌 callback，想像一下如果我們需要 10 層 callback 那會怎樣！

# 逃離回呼地獄（Callback Hall）

在大部分的程式語言中您比較少機會遇到這樣的問題，主要是因為它們以同步的機制為主，但在大量使用非同步的 JS 中我們就需要解決這個問題了。於是 Promise 就是為了解決這個問題而產生的。讓我們再來看看 Promise 的版本。

```
let resultA, resultB, resultC

function addAsync (num1, num2) {
  // fetch API 會回傳一個 Promise
  return fetch(`http://example.com?num1=${num1}&num2=${num2}`)
    .then(x => x.json())
}

addAsync(1, 2)
  .then(success => {
    resultA = success
    return resultA
  })
  .then(success => addAsync(success, 3))
  .then(success => {
    resultB = success
    return resultB
  })
  .then(success => addAsync(success, 4))
  .then(success => {
    resultC = success
    return resultC
  })
  .then(success => {
    console.log('total', success)
    console.log(resultA, resultB, resultC)
  })
```

透過 Promise 我們使用 `.then` 把 `callback hall` 不斷內嵌的語法給攤平了。

# 後起之秀 - Observables

在我們決定使用 Promise 之前，其實還有一個也是用來處理上面這些問題的新方法 `Observables`。

> Observable 是 lazy event stream 可以觸發 0 到 多個事件，這些事件也不見得需要完成。

相比，Observable 和 Promise 相比，主要的差異為：

* Observable 是可以被取消的。
* Observable 在需要時才會執行。

下面是使用 RxJS 的範例

```js
let Observable = Rx.Observable
let resultA, resultB, resultC

function addAsync (num1, num2) {
  const promise = fetch(`http://example.com?num1=${num1}&num2=${num2}`)
    .then(x => x.json())
  return Observable.fromPromise(promise)
}

addAsync(1, 2)
  .do(x => resultA = x)
  .flatMap(x => addAsync(x, 3))
  .do(x => resultB = x)
  .flatMap(x => addAsync(x, 4))
  .do(x => resultC = x)
  .subscribe(x => {
    console.log('total', x)
    conosle.log(resultA, resultB, resultC)
  })
```

* `Observable.fromPromise` 會將 Promise 轉換成 observable stream。
* `.do` 和 `.flatMap` 是 Observable 的運算子。
* 所謂的需要時才執行（`lazy`）意思是當我們呼叫 `.subscribe` 時才會開始執行。

Observable 可以將一些複雜的處理流程變的容易。舉例來說我們可以只用一行程式碼去 `delay` 延遲相加函式 3 秒

```js
addAsync(1, 2)
  .delay(3000)
  .do(x => resultA = x)
```
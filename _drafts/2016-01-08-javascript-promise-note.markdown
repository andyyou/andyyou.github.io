---
layout: post
title:  "Javascript Promise"
date: 2016-01-08 12:00:00
categories: Javascript
---

# 動機

非同步程式這種行為，大多只能在那些把函式是為一等公民的語言實作，其含意就是他們可以把 function 當作變數傳來傳去。這也是 callback 誕生的主要原因。
如果我們把一個函式A當作參數傳進另外一個函式B，在函式B中我們可以完成我們的任務之後再呼叫函式A。也就是說我可以把後續該怎麼處理先保留起來，等待要執行時再把這`實際的處理行為`當做變數傳進去

我們起初的動機其實很單純，好比讀取檔案，顯示內容這一系列動作，在阻塞式的程式中如 C#, Ruby 當我們要完成這些動作我們只要如下

```
string[] lines = System.IO.File.ReadAllLines("file.txt");
foreach (string line in lines)
{
    Console.WriteLine("\t" + line);
}
```

再往下執行之前，程式會卡在讀取那邊等待檔案讀完。

而非同步的 Javascript 並不會停下來等我們，所以例如說我們要儲存檔案時會這樣寫

```
Something.save(function(err) {  
  if (err)  {
    //error handling
    return;
  }
  console.log('success');
});

```

>callback 在 Javascript 中，簡言之即函式可以當作變數傳入另一個函式中，針對非同步的部分，我們需要在非同步行為處理完畢之後取得其結果並繼續加上我們後續的處理。
透過 callback 我們可以在該非同步行為完成式立即呼叫把該結果傳回，而處理的行為則允許我們之後再行規劃設計。

# Promise

試想下面這種用同步的方式來讀取 `JSON` 的 Javascript function。是不是簡單易懂，不過在大部分的程式中因為其阻塞的特性，就是要停下來等檔案讀取完畢的關係，你通常不會這麼寫。

```
function readJSONSync(filename) {
  return JSON.parse(fs.readFileSync(filename, 'utf8'))
}
``` 

為了讓我們的應用程式可以更有效率，即時的反饋，我們需要讓所有涉及 IO 操作的部分都改成[非同步](http://rowanmanning.com/posts/javascript-for-beginners-async/)。
最簡單的方法就是使用一個`回呼(callback)`。

> 意味著，這是等到檔案讀取完畢之後，立即執行 callback ，即後續的任務會放在 callback 內。

```
function readJSON(filename, callback) {
  fs.readFile(filename, 'utf8', function (err, res) {
    if (err) 
      return callback(err)
    
    callback(null, JSON.parse(res))
  })
}
```

這樣子的寫法容易造成混亂產生問題：

* 多餘的 callback 參數容易混淆我們的構想與判斷，到底哪些是傳入的值，哪些又是回傳的資料
* 它沒辦法用在所有原生控制流程語法中
* 另外當 `JSON.parse` 產生錯誤時，也沒辦法針對例外做處理

簡單的來說，所有的錯誤不應該直接就拋出例外，或在這邊就處理掉，而應該把錯誤傳給 callback，在 callback 裡面判斷處理。

> 開始讀取檔案 -> 處理取得資料 -> 解析要傳入的參數 -> 呼叫 callback 並帶入參數
> 簡單說前面三的步驟的過程不管產生錯誤，或正確回傳資料後續的行為都應該交給 callback 去處理

```
function readJSON(filename, callback) {
  fs.readFile(filename, 'utf8', function (err, res) {
    if (err)
      return callback(err)
    
    try {
      res = JSON.parse(res)
    } catch (ex) {
      return callback(ex)
    }
    
    callback(null, res)
  }
}
```

儘管我們完成了這個混亂的錯誤處理機制，但額外的 callback 參數這個問題仍然存在。

`Promises` 就是用來協助我們更自然的處理錯誤，讓程式碼更簡潔，並且省去額外的 callback 參數。

# 何謂 Promise ?

`Promise` 背後的核心概念就是一個 `Promise` 代表一個非同步操作的結果。一個 `Promise` 永遠處在下面三種狀態中的一種

* pending - Promise 初始化的狀態
* fulfilled - 代表 Promise 已經成功完成操作
* rejected - 代表 Promise 操作失敗

一旦 Promise 處於 `fulfilled` 或 `rejected` 的狀態，就不可以在產生變動。

# 建構 Promise

一旦所有的 API 回傳的都是 Promise 那麼我們就不需要自己撰寫下面這種函式了。不過此時此刻我們仍然需要 polyfill 或者如下範例

```
function readFile(filename, enc) {
  return new Promise(function(fulfill, reject) {
    fs.readFile(filename, enc, function(err, res) {
      if (err) 
        reject(err)
      else
        fulfill(res)
    })
  })
}
```

使用 `new Promise()` 即可建立一個 Promise 物件。我們在建構子帶入一個`工廠函式`使得 Promise 可以運作。
這個函式會立即被呼叫，並且帶入兩個參數，兩者都是函式。第一個 `fulfills` 能夠`完成`這個 Promise，而第二個 `rejects` 宣告 Promise 結果`失敗`。
一旦 Promise 操作完成我們就要呼叫適當的函式。

再次複習：一個 `Promise` 代表一個非同步操作的結果。

> 工廠函式: 透過一個函式產生需要的物件

# 等待 Promise

為了要使用 Promise 我們勢必需要某種方式來等待 Promise 內部呼叫 `fulfills` 或 `rejects`。
這個方法就是 `promise.done`

讓我們先來簡單的看一下 `readJSON` Promise 版本

```
function readJSON (filename) {
  return new Promise(function (fulfill, reject) {
    readFile(filename, 'utf8').done(function (res) {
      try {
        fulfill(JSON.parse(res))
      } catch (ex) {
        reject(ex)
      }
    }, reject)
  })
}
```

> 

# 轉換格式 / 鏈式

# 實作 Promise 原理

### 介紹

這個段落原出自於[Stack Overflow](http://stackoverflow.com/questions/23772801/basic-javascript-promise-implementation-attempt/23785244#23785244)。用意是讓您可以明白關於如何在 Javascript 中實作 Promise

### 狀態機

理論上一個 Promise 代表一個非同步的行為，為了達成有結果才繼續沒結果就卡住，那麼他勢必包含著某些狀態，例如是否完成該任務。

因此我們將其視為一個狀態機，我們就要開始思考到底一個 Promise 應該包含哪些我們需要的狀態


```
var PENDING = 0
var FULFILLED = 1
var REJECTED = 2

function Promise() {
  var state = PENDING
  var value = null
  var handlers = []
}
```

### 狀態的轉換

既然是一個狀態機，那麼當任務成功或失敗時，我們就需要修改狀態。所以讓我們來思考一下兩種勢必會產生的狀態 `fulfilling` 和 `rejecting`

```
var PENDING = 0
var FULFILLED = 1
var REJECTED = 2

function Promise () {
  var state = PENDING
  var value = null
  var handlers = []
  
  function fulfill (result) {
    state = FULFILLED
    value = result
  }
  
  function reject (err) {
    state = REJECTED
    value = err
  }
}
```

到了上一步，我們有了兩種內部底層轉換狀態的方法，現在讓我們來思考一個高階的轉換功能，讓我們暫且叫他 `resolve`。我們希望直接針對結果 `result` 來做處理。
結果如果是成功即一般的資料或物件那我們就呼叫 fulfill 如果失敗了就呼叫 reject 。

這裡還有另外一個希望的目標，那就是如果傳進來的是另外一個 Promise 那麼可以延續處理下去。

```
var PENDING = 0
var FULFILLED = 1
var REJECTED = 2

function Promise () {
  var state = PENDING
  var value = null
  var handlers = []
  
  function fulfill (result) {
    state = FULFILLED
    value = result    
  }
  
  function reject (err) {
    state = REJECTED
    value = err
  }
  
  function resolve (result) {
    try {
      var then = getThen(result)
      if (then) {
        doResolve(then.bind(result), resolve, reject)
        return
      }
      fulfill(result)
    } catch (ex) {
      reject(ex)
    }
  }
}
```

現在我們的 `resolve` 可以允許傳入另外一個 Promise 或者一個純物件。你可以看到上面的程式碼表明了，如果傳進來的物件可以拿到 `then` 這個方法，那麼就視其為 Promise
接著後續 `doResolve` 我們注意到 Promise 不會把 `fulfill` 交給別人處理，傳出去的是 `resolve` 因為其裡面還要判斷是不是 Promise 以此類推。

> 一個比較粗淺使用 Promise 的原因是因為我們想把 `a(b.bind(null, c))` 這種 callback 地獄換成 `a().then(b).then(c)`

```
function getThen (value) {
  var t = typeof value
  if (value && (t === 'object' || t === 'function')) {
    var then = value.then
    if (typeof then === 'function') {
      return then
    }
  }
  return null;
}

function doResolve (fn, onFulfilled, onRejected) {
  var done = false
  try {
    fn(function (value) {
      if (done) return
      done = true
      onFulfilled(value)
    }, function (reason) {
      if (done) return
      done = true
      onRejected(reason)
    })
  } catch (ex) {
    if (done) return
    done = true
    onRejected(ex)
  }
}
```

### 建立 Promise 建構式

現在我們已經完成初始化的狀態機制，



# 未整理

Pyramid of Doom 

動機就是 我們先想等 A 完成在做 B
在 JS 中如果我們要這麼做的話就是 A 會跟你說，你先把 B 交給我，等我做完我就會交代他繼續下去

A(B)

控制權要在我這，你先回傳一個東西 Promise 給我，我再去交待

do1().then(do2)

對於 callback 的精簡說明，可以把他想成是先在我們想要的地方埋入指標，指向另外一個片段的程式碼

```
function run () {
  // do our things
  pointer()
}

function pointer() {
  // keep going
}
```

在 JS 中為了要讓我們可以把後續的行為先拆出來，我們只能夠靠 callback 的方式

也就是說，我們必須要在執行當下行為的時候就得把後續的行為當作參數一起傳進去

一個 Promise 只是一個用來處理非同步流程的物件，透過傳入一個 callback
並且賦予你 resolve 和 reject 讓妳在事件完成時呼叫


* orderPizza
* eatPizza

function orderPizza (callback) {
  // after receive pizza
  callback()
}

function eatPizza () {
  
}

orderPizza(eatPizza)


    一个promise可能有三种状态：等待（pending）、已完成（fulfilled）、已拒绝（rejected）
    一个promise的状态只可能从“等待”转到“完成”态或者“拒绝”态，不能逆向转换，同时“完成”态和“拒绝”态不能相互转换
    promise必须实现then方法（可以说，then就是promise的核心），而且then必须返回一个promise，同一个promise的then可以调用多次，并且回调的执行顺序跟它们被定义时的顺序一致
    then方法接受两个参数，第一个参数是成功时的回调，在promise由“等待”态转换到“完成”态时调用，另一个是失败时的回调，在promise由“等待”态转换到“拒绝”态时调用。同时，then可以接受另一个promise传入，也接受一个“类then”的对象或方法，即thenable对象。

 * Promise
 * then
 * catch
 * function (resolve, reject) {}
 * Promise.resolve
 * 
 * 鏢局
 * 
 * then 收到得值來自 Promise 裡 resolve 傳入的
 * then 內部的 callback 是透過 resolve 觸發的
 * 如果 rejected 的話 then 就不會被呼叫 改呼叫 catch
 * 
 * 使用 Promise.all 只要一個人 reject 就會爆，都成功時回傳該陣列值
 * Promise.race 誰先發動就取誰

p.then(onFulfilled, onRejected);

p.then(function(value) {
   // fulfillment
  }, function(reason) {
  // rejection
});



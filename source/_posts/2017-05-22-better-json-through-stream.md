---
title: '[譯] 使用 stream 的方式處理 JSON 傳輸'
tags:
  - javascript
categories: Program
date: 2017-05-22 18:13:53
---


網頁應用程式在處理大量資料時一直都不是一件簡單的事。當需要接受大量資料時不只速度會變慢，且容易出錯，逾時等等。因此這是使用者體驗設計上的一個挑戰。
傳送大量資料同樣的也不容易，特別是在傳送前處理某些複雜的資料成為我們需要的資料時。

在這篇文章，我們將要來看看如何面對這些挑戰

<!--more-->

# 困難點

## 緩慢與高度依賴伺服器回應

當我們需要在伺服器端處理大量的 HTML 來回應使用者的請求時，速度理所當然就會變慢。更糟糕的是所謂的變慢，具體來說是指使用者的感覺。
當使用者點擊連結或發出請求後如果不能很即時的看到畫面，他們就會感到不開心。

提升伺服器回應的速度一直都是提升效能的目的，一旦回應需要執行一個非常慢的操作時，這件事就會變的棘手。如果回應的資料不只很大，還牽扯到需要使用其他第三方服務或 API 可想而知這個回應會需要更多的處理時間。

## AJAX 請求大量資料會很緩慢並容易出錯

關於伺服器回應緩慢的問題也說明了為什麼 AJAX 在 Web 開發中如此重要，因為我們就不需要等待伺服器幫我們編譯完整的頁面，而是把所需的資料直接傳回即可。一旦取得資料我們就在客戶端渲染，這可以省下不少時間。

然而將資料操作`整個`轉移到前端不見得是最快的作法。有可能變慢的原因來自大部分的個人電腦並沒有比伺服器的規格來的高，同時網路品質也比較慢。
使用者依然要等待所有資料下載完畢才能開始後續的處理，任何網路的問題都會造成資料不完全導致頁面無法正常呈現。

## 複數 AJAX 請求增加程式複雜度

一個相對不錯的方式；就是我們可以將取回一整包大量資料的 AJAX 請求打散成許多較小的請求，然後各自處理在頁面上對應的部分。
這種方式一般來說可以有效改善使用者體驗，因為使用者可以儘快的看到部分資料而不是等待整個頁面一起出現。
同時程式在網絡不佳的情況下也可用性也增加許多，已經完成下載的部分就可以運作，部分下載失敗的資料則不影響已經下載好的部分。

但很多時候處理多個請求也不見得都是那麼簡單，我們客戶端程式的邏輯會因此變的複雜許多。

## 模擬問題與實作

我們將要透過實作一個 Web 應用程式，使用它渲染顯示一張`貓在陽光下`的 8-bit 圖來示範如何解決這個難題。
這張圖片的構成資訊會包含在 JSON 裡。裡面包含多個物件的陣列，每個物件有 x, y, color 屬性。
而網頁頁面上將會是一個大的 div （一個 grid）包含許多小 div （row & cell）大概可以將其看成一張方眼紙。
我們希望一個格子對應一個點即使用物件中的 x, y, color，來完成這張圖。

### 限制

這裡為了模擬更多造成問題的原因，我們加入了些條件限制：

#### 多個資料來源

首先是關於貓的部分，每個點的資料會存放在我們伺服器上的 JSON 檔案。然後關於太陽圖案的部分則是存在 Github 的 JSON 檔案。

#### 回應大量資料

我們的 JSON 將會不合理的包含大量資料，這主要是為了凸顯我們要處理的問題。事實上貓圖案部分的 JSON 額外包含 100 個不需要的屬性，讓這個檔案變得較大，大約是 500k。如果這樣您還是覺得很正常，沒關係我們太陽的部分將包含 600 個不需要的屬性，整個檔案大約會來到 1 mb。

#### 不好的連線品質

我們同時希望使用者可以在網絡不好的狀況下使用我們的程式。

## 解決方案

我們的客戶端將會發出一個請求給伺服器。客戶端會有個事件處理程序，它會知道每當收到一個點的資料時該如何處理。
接著，我們的伺服器端會使用串流的方式回應 JSON ，如此一來我們就不需要等到所有資料都取得才開始處理。

最終我們將通過一個串流資料的方式取得所有圖案所需的資料。伺服器端將各自從兩個串流的資料來源取得資料並合併。這麼做是因為我們要模擬許多現實的情況，即資料來源一個較快（從硬碟讀取），另一個較慢（從網絡讀取）。

# 工具

## Oboe.js, 接收並處理串流類型的 JSON

`oboe` 讓我們在 JSON 整個完成傳輸之前便可以開始解析使用。一旦符合 pattern 格式的資料被解析完成就執行對應的動作。

```js
const source = 'url or readable stream'
const pattern = 'string representing a node in the tree'

oboe(source)
  .node(pattern, function (data) {
    // handle data
  })
```

## Highland.js - 處理串流資料的函式庫

一直以來我總是覺得自己實作串流機制的方式非常麻煩，而且容易不知道它們該怎麼和既有的工作流程整合。`Highland.js` 是一個函式庫，它使得串流資料變得容易被管理和操作。

```js
let data = ['one', 'two', 'three']
highland(data)
  .pipe(anotherStream)
```

# 接收串流的回應

我們將從客戶端開始作回來。首先是我們的回應是一個大量資料的 JSON，但我們不想等它們全部載完才開始動作。

第一步是使用 `Oboe` 來發起請求，Oboe 會在發出請求後開始解析回應。

```js
oboe('http://localhost:3000/data')
```

接著，我們要註冊兩個事件監聽用來處理回應。第一個會尋找 JSON 中具備 3 個屬性（x, y, color）的物件。在這個範例我們只會搜尋 `pixels` 陣列，接著我們就可以將資料更新到對應的元素上。

```js
.node('{x y color}', function (point) {
  let grid = document.querySelector('.grid)
  let cell = getCell(grid, point.x, point.y)
  cell.classList.add(point.color)
})
```

第二個監聽事件只會在整個回應都被解析完成時觸發一次

```js
.done(function () {
  let el = document.querySelector('#status')
  el.textContent = 'All data loaded'
})
```

看看下面的範例會更有感覺：

* [範例](https://jsfiddle.net/andyyu0920/myv0gjj1/)

# 傳送串流類型的回應

能夠處理串流的資料來源之後當然下一步，我們的應用程式將需要一個路由用來提供資料給客戶端。首先我們需要一個回應的樣板，讓我們可以組織 metadata 或其他屬性。
重要的是後續當我們取得 `pixels` 陣列時可以知道該怎麼和這個字串化的物件合併。您可以把這個過程想成我們在使用像 ejs, pug 這類樣板引擎一樣。
我們在需要後續填上資料的地方先留下一個佔位符

> 不懂這邊在說什麼，先耐著性子讀下去...

```js
router.get('/data', function (req, res) {
  let response = {
    exif: {
      software: '',
      dateTime: '',
      dataTimeOriginal: ''
    },
    pixif: {
      pixels: ["#{pixels}"]
    },
    end: 'test'
  }
})
```

`res` 物件是 express 傳到客戶端一個 WriteableStream，這意味著我們可以接續處理（pipe） ReadableStream 然後送到客戶端。
然而它只能夠`接收字串`所以我們要將物件轉換成字串，這也是上面需要樣板機制來處理的原因。 

```
let json = JSON.stringify(response)
```

在轉換成字串後，我們將要切割 response ，我們就可以將 `#{pixles}` 前後的部分拆出來。

```js
let parts = json.split('"#{pixels}"')
```

現在我們可以將除了 pixels 之外的部分建立一個新的 highland stream。

```js
highland({
  parts[0], // before placeholder
  parts[1] // after placeholder
})
```

接著我麼可以在串流上使用 `invoke` 來讓串流的每個元素執行 `split` 並使用參數 `''`，這表示我們串流中的內容會被切成一個個字元。

```js
.invoke('split', [''])
```

然後我們要告訴 `highland` 我們想要依序讀取每個元素。如此一來 highland 就會先讀取 `before` 的部分直到完成才接著讀取 `after` 的部分。
這點非常重要，因為我們需要的是一個符合規範的 JSON 不然的話我們取得的只是一堆亂碼。

```
.sequence()
```

最後我們可以開始傳送串流。

```js
.pipe(res)
```

到這邊我們有了基本的概念，也知道 `highland` 可以協助我們處理串流的部分，甚至直接幫我們處理 express 回應的部分

```js
router.get('/data', function (req, res, next) {
  var response = {
    exif: {
      software: 'http://make8bitart.com',
      dateTime: '2015-11-07T15:35:13.415Z',
      dateTimeOriginal: '2015-11-07T00:24:05.776Z'
    },
    pixif: {
      pixels: ["#{pixels}"]
    },
    end: 'test'
  }
  var json = JSON.stringify(response)
  var parts = json.split('"#{pixels}"')
  highland([
    parts[0],
    parts[1]
  ])
  .invoke('split', [''])
  .sequence()
  .pipe(res)
})
```

# 在回應中加入資料

這時我們的客戶端已經可以接收串流的回應了，我們可以繼續處理加上其他資料到這個回應中。現階段我們單純加入一些靜態的資料來驗證行為如同我們所想的。

```js
let points = [
  {
    x: 1,
    y: 2,
    color: 'orange'
  },
  {
    x: 2,
    y: 2,
    color: 'orange'
  }
]
```

我們可以為這個陣列建立一個串流

```js
let pointsStream = highland(points)
```

剛剛提到 express 只能接收字串串流，所以這邊我們也要轉成字串，我們可以使用 `map` 來完成這個需求

```
.map(point => JSON.stringify(point))
```

這個剛建立的陣列串流會被塞進 `"#{points}"` 佔位符的位置，即放在陣列裡面。由於最終我們需要的是一個符合規範的 JSON 因此在這個串流的每個元素之間我們還缺少一個逗號。要完成這個需求我們可以通過下面的方法告訴串流物件幫我們在每個元素之間加上逗號。

```js
.intersperse(',')
```

現在我們可以將這個陣列的 stream 組進剛剛的 `before` 和 `after` 中間完成一個完整的串流

```js
highland([
  parts[0],
  pointStream,
  parts[1]
])
.invoke('split', [''])
.sequence()
.pipe(res)
```

# 從其他模組取得資料

我們可以透過將資料拆離成模組簡化路由的程式

```js
let points = require('../data/points')
let pointStream = points.getStream()
```

也就是說不管模組做什麼只要最後傳回一個 stream 給我們就好，其他組合的部分就如同上面說明。

# 從檔案取得貓圖案部分的資料

現在讓我們來使用真的資料而不是我們模擬的假資料。首先我們需要從檔案讀取關於貓圖案的資料，將其一併從模組傳回。

```js
function getDataStream () {
  let catPath = path.resolve(__dirname, './cat-points.json')
  let catSource = fs.createReadStream(catPath)
}
```

關於檔案的內容將會是字串的串流，而我們最後需要的是物件的串流

```
let catStream = getPointStream(catSource)
```

要完成這一步我們需要再次利用 Oboe。這麼做我們將可以在讀取檔案時，每當一個點完成讀取就執行對應的操作而不需要等到整個檔案載人完成。
我們也會需要用到 highland 來建立 stream。這次我們使用 highland 並傳入一個 function 。該 function 有一個參數 `push` 。
在 function 內我們會調用 `push` 傳入錯誤或 null 為第一個參數，第二個參數則是我們的資料。

```js
function getPointStream (sourceStream) {
  return highland(function (push) {
    oboe(sourceStream)
      .node('{x y color}', function (point) {
        push(null, point)
      })
      .done(function () {
        push(null, highland.nil)
      })
  })
}
```

# 從網路讀取太陽圖案的資料

最後一個難題則是將太陽圖案的點加到我們的回應。幸運的是前面我們已經完成了類似的任務。
在 `getDataStream` 我們將類似處理貓圖案的部分加入其他點，只不過來源不同。這裡我們會使用 `request` 函式庫來取得在 Github 上面的檔案。
這一步會是比較慢的操作。

幸運的是 `request` 會回傳一個可讀的 stream 我們可以將其傳入 `getPointStream`，Oboe 將會將其處理為跟 `fs.getReadStream` 一樣的結果。

```js
function getDataStream () {
  let sunUrl = 'https://raw.githubusercontent.com/JuanCaicedo/better-json-through-streams/master/data/sun-points.json'
  let sunSource = request(sunUrl)
  let sunStream = getPointStream(sunSource)
}
```

現在我們有兩個 stream 了，`catStream` 和 `sunStream` 我們想要將它們合併傳回。我們可以使用 `highland` 來處理這兩個 stream，使用 `merge` 來合併成一個 stream。

```js
return highland([
  catStream,
  sunStream
]).merge()
```

# 總結

設計應用程式搭配串流作為主要的資料傳輸手段是非常強大的。它讓我們可以更有彈性的處理資料傳輸的部分。

# 參考

* [Node.js Streams](https://simeneer.blogspot.tw/2016/10/nodejs-streams.html)
* [完整範例](https://github.com/andyyou/stream-json-sample)
* [原文 - Better JSON through streams](http://engineering.curiositymedia.com/blog/better-json-through-streams)
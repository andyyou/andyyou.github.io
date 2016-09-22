---
layout: post
title: "學習 HTML Drag and Drop"
date: 2016-06-30 12:00:00
categories: js, html
---

在過去很長一段時間拖移元素這個功能我們得依賴 Javascript 來實作，這其中並沒有任何瀏覽器原生的實作方式。類似於 `scroll-behavior` 這個 css 屬性，在它出現之前，平滑捲動的效果也是得依賴 Javascript 實作。

在那段日子裡我們常常使用 jQuery 和 Dojo 這類的函式庫幫做我們實作這些效果。現在，在 HTML5 我們擁有了一個原生的方式來建立 `drag` 和 `drop` 拖移的效果(又簡稱 DnD)，簡略的說它就是定義了一系列對應事件，Javascript API 以及一些 HTML 屬性，這篇文章會介紹如何使用這種方式。

# 檢查是否支援

因為實務上有許多應用是不可缺少 DnD 的操作效果，舉例來說一個西洋棋的遊戲，一旦不能拖移棋子，立馬就變得非常難用。也因此對於那些不支援的瀏覽器我們勢必還是得提供優雅降級的替代方案，所以檢查瀏覽器是否支援原生 DnD 就變得很重要。當 DnD 不支援時我們得提供另外的操作方式，以確保程式可以使用。

一個重點是當我們需要相依於某個 API 時我們一定要檢查瀏覽器是否支援`該功能`，而不是簡單的判斷 `User-Agent`。針對這個需求，其中一個不錯的函式庫就是 [Modernizr](http://www.modernizr.com/)。不過在不依賴其他函式庫來說我們可以這麼做

```js
'draggable' in document.createElement('div')
```

> [Modernizr.draganddrop 因為一些原因被移除了](https://github.com/Modernizr/Modernizr/issues/637)

# 建立一個可以被拖移的內容

要讓一個元素可以被拖移是非常簡單的，只要替它設上一個 `draggable=true` 的屬性即可。圖片，連結，檔案和 DOM 節點都是可以設定提供拖移效果的。
舉個例子，讓我們來實作一個欄位可以被拖移排序的功能，下面是 HTML 結構

```html
<div id="columns" class="columns">
  <div class="column" draggable="true"><header>A</header></div>
  <div class="column" draggable="true"><header>B</header></div>
  <div class="column" draggable="true"><header>C</header></div>
</div>
```

值得注意的是在大部分的瀏覽器下，選取的文字，圖片，帶有 `href` 的標籤預設是可以被拖移的。然後它們可以被拖到 `<input type="file">` 元素中或者您電腦的桌面。如果您想要讓其他類型的內容可以被拖移那麼就要使用 HTML5 DnD API。

因為被選取的文字也能被拖移，所以我們得透過 CSS3 的一點小技巧讓上面 HTML 結構的效果看起來像是`欄位`，加上 `cursor: move` 讓使用者知道這個區塊是可以被拖移的。

```scss
/**
 * 防止可以被拖移的元素內部的文字被選取造成局部拖移
 */
[draggable] {
  -moz-user-select: none;
  -khtml-user-select: none;
  -webkit-user-select: none;
  user-select: none;
  /**
   * 舊版 WebKit 需要此設定讓元素可以被拖移
   */
  -khtml-user-drag: element;
  -webkit-user-drag: element;
}

.column {
  height: 150px;
  width: 150px;
  float: left;
  border: 1px solid #ccc;
  background-color: #eee;
  margin-right: 5px;
  text-align: center;
  cursor: move;

  header {
    text-shadow: #fff 0 1px;
    box-shadow: 5px;
  }
}
```

[CodePen](http://codepen.io/andyyou/pen/RRVLQm)

上面這個範例在多數的瀏覽器中您可以拖拉並產生一張半透明的元素內容快照圖片，其他(例如 Firefox)瀏覽器必須在拖移地操作過程中設定`傳輸資料`才會產生效果。在下個章節我們將會綁定一些事件(drag/drop)讓我們的欄位範例能有更多效果。

# 監聽拖移事件

HTML5 提供我們一些事件好讓我們在拖移的過程中執行一些對應的行為，在沒有專門的事件之前，所謂靠 Javascript 實作，簡略來說就是依靠 `mousedown` `mouseup` `mouseover` 等這些事件來模擬操作的行為(行動裝置上就是 touch events)。

在整個 `drag 拖移`，`drop 放入`的操作過程中，每一個步驟都有不同的事件會被觸發：

* dragstart 當使用者開始拖移一個元素觸發
* dragenter 當 `拖移中的元素` 進入目標元素(綁定此事件的元素)時觸發
* dragover 在`拖移狀態中`，當拖移元素的滑鼠指標經過該元素時觸發，只要在該元素上面就會一直觸發
* dragleave 在`拖移狀態中`當滑鼠指標進入後離開元素時觸發
* drag 在拖移元素的過程中會被一直觸發
* drop 當實際放入元素的行為被執行時觸發
* dragend 拖移過程中當使用者放開滑鼠即結束本次拖移時觸發

您可以透過下面的範例玩玩觀察這些事件觸發的時機：

![](http://i.imgur.com/UhqWpZD.gifv)

[CodePen 範例](http://codepen.io/andyyou/pen/GqmpNj)

為了掌握 DnD 的流程，我們需要一個來源元素，就是我們想拖移的目標，傳輸資料(我們試圖搬移的東西)，以及一個置放區域(一個能夠存放擷取東西的地方)。首先來源可以是圖片，清單，連結，檔案，一個 HTML 的區塊。置放區域(drop zone)，置放區域或稱作目標就是允許使用者把拖移的資料放進去的地方。記住並不是所有元素都可以成為`置放區域`(例如 圖片就不可以)。

# dragstart 開始拖移

一旦設定了 `draggable="true"` 那麼接下來我們就可以透過綁定 `dragstart` 事件來開始，這是一系列拖移操作的第一個事件。

```js
var columns = document.querySelectorAll('.columns .column')

;[...columns].forEach( (col) => {
	col.addEventListener('dragstart', dragStart, false)
})

function dragStart (e) {
  this.style.opacity = 0.4
}
```

[JSFiddle](https://jsfiddle.net/cq594vvn/5/)

> 因為 CodePen 的 Babel 有些問題所以這個範例在 JSFiddle 當然您也可以換成舊的寫法 `[].forEach(columns, fn)`

因為 `dragstart` 事件的 target 是我們拖移的元素所以我們可以透過 `this` 或 `e.target` 來設定取得其 DOM，將其透明度變成 40% 讓使用者感受到他正在拖移該物件。另外一方面當我們拖移結束時應該讓透明度回到 100%，當然最直覺的發生`事件`的時間點就是在 `dragend`

# dragenter, dragover 和 dragleave

dragenter, dragover, 和 dragleave 這三個事件可以在整個拖移過程中提供其他的視覺效果。舉例來說當拖移的欄位進入另外一個欄位位置時顯示虛線邊框。如此一來使用者就知道拖移的元素將被放到另一個元素的位置。

```scss
.column {
  &.over {
    border: 2px dashed gray;
  }
}
```

### dragover

透過下面這個範例，我們理解在拖移狀態下滑鼠指標移到`有綁定此事件`的元素上時會被觸發。

```js
function dragOver (e) {
	console.log(this.id)
  if (e.preventDefault) {
  	// 觸發 drop 需要這行
  	e.preventDefault()
  }
  return false
}
```

[JSFiddle](https://jsfiddle.net/andyyu0920/ajb13o2a/3/)

> 注意在還沒談論到 DataTransfer 物件之前這些範例最好使用 Chrome 測試

### dragenter

一樣在拖移狀態下，滑鼠指標移`進入`這個 `有綁定此事件` 的元素上時會被觸發，但只觸發一次並且 enter 會先於  over。

```js
function dragEnter (e) {
  this.classList.add('over')
}
```

最後加上 dragleave 事件，完整範例如下：

```js
var columns = document.querySelectorAll('.columns .column')

;[...columns].forEach( (col) => {
	col.addEventListener('dragstart', dragStart, false)
  col.addEventListener('dragover', dragOver, false)
  col.addEventListener('dragover', dragEnter, false)
  col.addEventListener('dragleave', dragLeave, false)
})

function dragStart (e) {
  this.style.opacity = 0.4
}

function dragOver (e) {
  if (e.preventDefault) {
  	// 觸發 drop 需要這行
  	e.preventDefault()
  }
  e.dataTransfer.dropEffect = 'move'
  return false
}

function dragEnter (e) {
  this.classList.add('over')
}

function dragLeave (e) {
  this.classList.remove('over')
}
```

[JSFiddle](https://jsfiddle.net/andyyu0920/ajb13o2a/12/)

上面這幾個事件有幾個重點需要注意

* `this` 或 `e.target` 會隨著觸發事件而改變
* 某些情況下我們需要在 `dragover` 使用 `e.preventDefault()` 和 `return false` 來阻止瀏覽器預設的行為，例如拖拉一個連結時。
* 在 `dragenter` 移除 css `.over` 而不是 `dragover` 因為 `dragenter` 只會在進入時被觸發一次，而 `dragover` 會持續觸發。
* 在拖移過程中當滑鼠進入子元素時一樣會觸發對應事件，於是可能會產生一些非預期的效果。

# 完成一次拖移操作

為了完成拖移並`改變元素位置`的動作，我們需要加上 `drop` 和 `dragend` 事件。在這個事件中同樣也需要阻止瀏覽器執行預設行為，例如在 Firefox 底下如果不阻止預設行為，那麼將會跳轉頁面。因為這個預設行為是因為事件往上層 DOM 傳遞而觸發的，我們可以透過 `e.stopPropagation()` 阻止事件往上層 DOM 傳遞觸發。

在這個拖移位置的範例中少了 `drop` 事件是無法完成的，不過在完成所有程式之前我們先透過 `dragend` 來移除剛剛產生的一些效果。`dragend` 觸發的時機比教單純一點就是這次拖移過程`放掉滑鼠`結束時。

```js
function drop (e) {
	console.log('drop')
	if (e.stopPropagation) {
  	e.stopPropagation()
  }

  return false
}

function dragEnd (e) {
  this.style.opacity = 1
	;[...columns].forEach( (col) => {
    col.classList.remove('over')
  })
}
```

到了這一步，您應該會發現我們的範例仍無法交換元素的位置。接下來我們該介紹 `DataTransfer` 這是我們在拖移過程中用來交換傳輸資料的物件。

# DataTransfer object

在拖移行為產生時，都會有一個資料傳輸屬性物件 `e.dataTransfer`。讓我們可以在過程中傳遞資料。通常在 `dragstart` 事件中設定，在 `drop` 事件中讀取與處理這些資料。例如調用 `e.dataTransfer.setData(format, data)` 可以設定內容與格式然後會被當成下個事件的參數傳入。在我們的範例中傳遞的資料就是 HTML

```js
var dragSrcEl;

function dragStart (e) {
  this.style.opacity = 0.4

  dragSrcEl = this
  e.dataTransfer.effectAllowed = 'move'
  e.dataTransfer.setData('text/html', this.innerHTML)
}
```

有 `setData()` 相對的也有一個 `getData(format)` 方法讓我們可以取得資料。下面我們就可以執行交換位置的動作

```js
function drop (e) {
	if (e.stopPropagation) {
  	e.stopPropagation()
  }

  if (dragSrcEl != this.innerHTML) {
  	dragSrcEl.innerHTML = this.innerHTML
    this.innerHTML = e.dataTransfer.getData('text/html')
  }

  return false
}
```

* `dataTransfer.effectAllowed = value` 是本次拖拉允許的行為，好比說 none 是不可以被 drop, copy 是到目標時複製一份，列出其值可能為下面 `none`, `copy`, `copyLink`, `copyMove`, `link`, `linkMove`, `move`, `all` 其中之一。詳細說明請參考[MDN](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer/effectAllowed)
* `dataTransfer.setData(format, data)` 加上指定的資料，在 Firefox 底下這個屬性一定要被設定才能夠拖移，否則就只會觸發 dratStart。
* `data.Transfer.clearData(format)` 針對特定格式清除所有資料。
* `dataTransfer.getData(format)` 取得特定格式或名稱的資料。
* `dataTransfer.setDragImage(img, xOffset, yOffset)` 設定拖移時半透明的圖片，預設會是拖移元素的快照，但我們可以設定其他圖片，目前支援的狀況是 Firefox 可以設定任何 DOM elements，但 Chrome 只能支援 HTMLImageElement。至於 offset 就是對照滑鼠的座標，(0, 0) 就是滑鼠跟在圖片的最左上角。

![](http://i.imgur.com/uwNPkwW.png)

想要換圖片的話可以參考類似做法

```js
var img = document.createElement('img')
/* img.src = "http://i.imgur.com/fEF6DWr.jpg" // Firefox support */
img.src = "data:image/jpeg;base64,[YOU PUT BASE64 FORMAT HERE]"
event.dataTransfer.setDragImage(img, 0, 0)
```

### 一個簡單的例子

如果您因為一口氣看到這些事件而感到混亂，那麼這邊用個例子簡單說明一下。假設我們有一個容器 A 允許我們把一些元素拖進去置放。
在這種情況下，元素需要的事件就是 `dragStart`。而容器 A 則會需要判斷是否拖拉進範圍內 `dragEnter` 或者拖出範圍 `dragLeave`，置放 `drop`。因為這些事件被觸發所以我們可以執行對應的處理。

# 實作簡單的 drag 和 drop 範例

現在我們來一步一步建立一個簡單的範例，

# 總結

* drag start
  + 設定 `e.dataTransfer`
  + `來源元素`的 `dragstart` 設定 `e.dataTransfer.effectAllowed = 'move'`, 然後`目標元素`的 `dragover` 使用`e.dataTransfer.dropEffect = 'copy'`
* drag
* drag enter
* drag over
  + 如果`目標元素 place element/target` 要觸發 `drop` 需要使用 e.preventDefault()
  + `e.dataTransfer.dropEffect = 'copy'` 可以限制後續 drop 是否會觸發
* drag leave
* drop
  + 少了 e.stopPropagation() Firefox 和部分瀏覽器會跳轉頁面
* drag end

# 參考資源

* [](https://developer.mozilla.org/zh-TW/docs/Web/Guide/HTML/Drag_operations)

---
layout: post
title: "Bacon.js 入門"
date: 2016-08-30 12:00:00
categories: js
---

# 註冊表單

在這篇教學中將會試圖建置一個完整但簡單的功能，為一個網站設計一個使用 AJAX 的註冊表單

![](https://raw.github.com/raimohanska/nulzzzblog/master/images/registration-form-ui.png)

# 原本的方式

下面的程式碼看起來相當簡單，對嗎？輸入使用者帳號，名字然後點擊註冊按鈕即完成這個流程

```js
registerButton.click(function (event) {
  event.preventDefault()
  var data = {
    username: usernameField.val(),
    fullname: fullnameField.val()
  }
  $.ajax({
    type: 'post',
    url: '/register',
    data: JSON.stringify(data)
  })
})
```

乍看之下我們的功能蠻單純的，但仔細再想想當我們想增進使用者體驗我們可能會開始考慮加入下面這些功能

1. 在使用者輸入 `username` 時檢查是否可以使用，是否有被其他用戶取走了
2. 當這個 `username` 不能使用時顯示訊息給使用者
3. 顯示 AJAX 讀取中的進度圖示或訊息
4. 當 `username` 和 `fullname` 未輸入時 `disable` 註冊按鈕
5. 當 `username` 不能使用時也 `disable` 註冊按鈕
6. 在 AJAX 檢查的同時註冊按鈕需要 `disable`
7. 當點擊按鈕時要立刻停用按鈕，防止使用者快速點兩次造成兩次 submit 送出
8. 開始執行 AJAX 註冊的流程時，顯示讀取中的進度圖示或訊息
9. 顯示處理的結果

好吧！這些需求聽起來還蠻合理的，在今時今日這樣的功能似乎隨處可見

![](https://raw.github.com/raimohanska/bacon-devday-slides/master/images/registration-form-thorough.png)

現在你清楚這些了，從上面具體的例子來看，`啟用/停用註冊按鈕`需要取決於挺多條件的，並且有些可能同時發生，沒有一定的順序我們稱之為非同步。

# 思考架構

通常這會是我們如何使用 `Bacon.js` 實作的過程。

1. 擷取出輸入的部分轉換成 `EventStream` 和 `Properties` 即事件流與屬性
2. 根據我們的需求，轉換與組合這些 signals 信號(即執行動作的觸發時間點和產生異動的資料)
3. 最後將我們會產生的一些操作分配到這些 signals

> 現階段我們有點難理解 EventStream 從字面上來看是事件的串流，這麼說有點難懂，因為串流這個詞有點抽象，我們可以概略理解為一個時間軸，每個時間點觸發的事件都會被觀察紀錄。

在實作時通常我們會先選擇一個功能完成，然後接著下一項功能，直到完成全部的功能。過程中最好能夠持續的重構保持整潔的原始碼。

有時在紙上先畫下來對我們也有幫助。在這個學習案例我們為你提供了下圖

![](https://raw.github.com/raimohanska/bacon-devday-slides/master/images/registration-form-bacon.png)

在上圖中綠色的區塊表示 EventStream (不同的事件)，灰色的區塊表示屬性(隨著時間變動的屬性值)最上方的三個區塊代表輸入的信號

* 兩個輸入欄位的 key-up 事件
* 註冊按鈕的 click 事件

在這篇文章中我們會捕捉輸入的信號，然後定義 username 和 fullname 的屬性。最終我們可以輸出值。

# 設定

您可以單純只閱讀文章或者您也可以試著實做一些簡單範例，如果您比較喜歡後者下面提供一些簡單的範例與實作。
首先您應該取得範例到您的電腦上

```
$ git clone https://github.com/raimohanska/bacon-devday-code.git
$ cd bacon-devday-code
$ git checkout -t origin/clean-slate
```

現在您應該複製了一份原始碼並且切換到 `clean-slate` 分支。又或者您想要 fork 專案然後再建立新的分支。

無論如何你現在應該可以打開 `index.html` 並看到註冊表單。同時您可能也用您慣用的編輯器開啟專案並大致瀏覽一下。
您會看到有一些函式簡單的協助我們存取 DOM 元素。

# 從 DOM 事件截取輸入

`Bacon.js` 並不是一個 jQuery 的 plugin 所以並不相依於 jQuery，不過當它發現您有安裝 jQuery，它會在 jQuery 加上一些輔助的方法(Methods)。這個 `asEventStream` 方法是用來截取事件轉換為一個 EventStream。

例如要擷取 username 欄位上的 `keyup` 事件，我們可以這麼做：

```js
$('#username input').asEventStream('keyup')
```

如此一來我們將會取得 jQuery keyup event 的 EventStream，接著我們可以試試在瀏覽器 console 使用下面的程式碼

```js
$('#username input').asEventStream('keyup').log()
```

現在只要事件被觸發就會執行 log 顯示資訊在 console。為了定義 username 屬性，我們需要轉換這個串流為另一個文字欄位值的串流，接著將其轉換為屬性：

```js
$('#username input').asEventStream('keyup').map(function (event) {
  return $(event.target).val()
}).toProperty('')
```

為了觀察這一小段程式碼做了什麼我們可以在最後面加上 `.log()` 您會看到結果顯示在 console

好啦！我們剛剛到底做了什麼？

我們使用了 `map` 這個方法轉換每一個事件變成 username 欄位的值。`map` 方法會回傳`另外`一個 stream 裡頭裝的是映射處理過後的資料。本質上就像是 `underscore.js` 的 `map` ，不過分成 EventStream 和 Properties

透過逐一處理串流中的資料，我們將原本的串流轉換成另外一種 Properties 串流(`toProperty('')`)，裡面的空字串表示字串初始化的值，這會是目前的值，直到觀察到我們原本的事件串流有事件發生，值才會產生對應的變化。

再一次強調 `toProperty` 方法回傳的是一個新的屬性，並不會改變原來串流中的資料。事實上 Bacon.js 中的所有方法回傳的東西都不會造成任何副作用即對原本的資料造成異動。這也是我們想從 Functional programming 中得到的好處。

到這邊 `username` 屬性已經可以拿來使用了。

```
username = $("#username input").asEventStream('keyup').map(function (event) {
  return $(event.target).val()
}).toProperty('')
```

這一步我們故意省略 `var` 如此我們可以方便地在 console 中測試

下一步我們繼續來定義 `fullname` 看起來程式碼很類似所以我們可以直接複製貼上？

這樣不好！我們可以稍微重構一下

```js
function textFieldValue(textField) {
  function value () {
    return textField.val()
  }

  return textField.asEventStream('keyup').map(value).toProperty(value())
}

username = textFieldValue($('#username input'))
fullname = textFieldValue($('#fullname input'))
```

不錯！但是事實上，`Bacon.UI` 已經內建了 `textFieldValue` 函式，所以我們只需要

```js
username = Bacon.UI.textFieldValue($('#username input'))
fullname = Bacon.UI.textFieldValue($('#fullname input'))
```

我們觀察到在我們已經解決過的許多問題中有些問題在不同的專案中都會有需求，於是我們將其抽出來為 helper，如果您遇到某些需求覺得應該提供 helper 不要客氣歡迎加入協作。

無論如何，當你完成了上面的範例程式您可以試著重新載入並在 console 中輸入

```js
username.log()
```

在瀏覽器的 console 中您可以觀察當每當 username 發生改變，就會在 console 輸出

# Mapping Properties 並執行操作(Side-Effect)

為了讓我們的 App 實際完成一些任務我們將會定義一些新的 Properties 串流並賦予一些產生副作用的操作。例如根據 `username` 和 `fullname` 的資料`啟用`/`停用`我們的註冊按鈕

我們將從 `buttonEnabled` 屬性開始

```
function and (a, b) {
  return a && b
}

buttonEnabled = usernameEntered.combine(fullnameEntered, and)
```

上面透過合併兩個 Property 搭配 `and` 成為另外一個按鈕的狀態的 Property。這個 `combine` 的運作是當 `usernameEntered` 和 `fullnameEntered` 任何一個發生改變時，最終按鈕的 Property 會得到新的值。這個新值會在建構時讓兩個屬性調用 `and` 函式。

一樣我們還能夠使用內建的方法

```js
buttonEnabled = usernameEntered.and(fullnameEntered)
```

這跟前一段程式完成一樣的任務，不過依靠內建的布林邏輯的方法(and, not, or)。
不過您現在肯定有些困惑，我們還沒定義 `usernameEntered` 和 `fullnameEntered`

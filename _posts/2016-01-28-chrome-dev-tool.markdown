---
layout: post
title: 'Chrome Dev tool 筆記'
date: 2016-01-28 16:30:00
categories: Tools
---

# Element Panel

* `⌥⌘I` 開啟 Dev Tool

### 選取元素的方式

* 對元素點右鍵 -> Inspect
* 開啟 Dev Tool -> 點擊 Dev Panel 左上角 bar 的箭頭 ICON -> 選取元素
* `⌘⇧C` -> 選取

### 編輯樣式

* 選定一個 Element 之後右邊 panel 會顯示該 Element 之樣式
* 開關樣式
* 新增或刪除樣式
* 右上角 -> Change state(hover, active, focus, visited)
* Computed -> 觀察 box model
* 點擊連結切換到 Source
* `Shift` + 點擊 CSS rule 的顏色會轉換 e.g. hex to rgb
* 編輯樣式時，選取某個字串 ->`Command+D`會把一樣的都選起來


### Source Panel

* 進入 Source panel `⌘S` 可以暫時編輯檔案並存到瀏覽器的 Storage
* 右鍵 -> Local Modifications 可以檢視歷史紀錄

### Console

```
console.log();
console.assert(false, 'Information'); // 錯了就噴第二個參數
console.count('go'); // go: 1
console.count('go'); // go: 2
console.log("document body: %O", document.body);

console.group("Authenticating user '%s'", user);
console.log("User authenticated");
console.groupEnd();

console.log('%c Hello', 'color: orange;'); // %c 使用 css
console.log("%O ", document.body); // %O Javascript Object
console.log("%O ", document.body); // %o DOM 
```
[詳細 Console API](https://developer.chrome.com/devtools/docs/console-api#consoleprofilelabel)

* 學會查看 Error，切換到 Source 
* 在 Source panel 下 ESC 開啟第二層 Console


```
$('selector') // 稱為 blind
// 如果載入 jQuery 預設的 blind 會被覆寫
// 回傳的是一個陣列
inspect($('#title')) // 選取元素並切換到 Element Panel
$0 // 當前選取的 Element
$1 // 上一個
```
* Source panel 右上角的 Pause on 按鈕打開的話，下次噴錯就會暫停
* Source panel 左下角的 `{}` 按鈕可以把 minifed 檔案轉換成較易讀的格式

### Local Storage

* 切換到 Resource panel

### Network panel

* Refresh 之後會記錄每個 Request
* `⇧` + Refresh 鍵強制全部重載
* Size = Transfer size
* Content = Actual size
* 最底下的 Bar 有總和
* Waterfall 裡面淺色的部分代表`發出 request`到`回應即開始傳輸資料`的時間
* 實心的部分代表開始下載資料
* 顏色的意義
  - HTML 藍色
  - JS 橘色
  - CSS 綠色
  - 圖片 紫色
  - 右邊垂直的藍色線代表 DOM loaded 意為瀏覽器解析完畢 HTML to DOM
  - 右邊垂直紅色線則表示圖片等資源下載完畢
* CSS 放在 JS 之前

### Timeline

* 顏色的意義
  - Loading 藍色
  - 執行 Script 黃色
  - Rendering 紫色
  - Painting 綠色
* [Size vs Content](http://stackoverflow.com/questions/8072921/chrome-dev-tools-size-vs-content)
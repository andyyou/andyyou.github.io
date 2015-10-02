在網站開發的領域中影像和聲音截取在過去很長一段時間是不可侵犯的神聖領域，我們必須要依靠在瀏覽器上安裝外掛程式(Flash, Silverlight)來完成這個任務。
而今 HTML5 試圖解決這個問題。也許不是一定能成功，但 HTML5 已經增加了許多對裝置存取的支援。GPS, Orientation API, WebGL 以及 Web Audio API。
這些功能異常的強大，並且提供高度抽象畫的 Javascript API 。這篇文章將會介紹一個新的 API `navigator.getUserMedia()` 讓我們能夠存取使用者的相機 WebCam 和麥克風。

## getUserMedia() 之路
如果您還不清楚關於它的歷史，getUsermedia() API 的演進是一個有趣的故事。
幾個媒體截取 API 在近幾年個別有了些進展。越來越多人認為 Web 應該要有能力存取操作裝置，但是這也導致每個人都將想將自己的東西加入新的規範中。
結果是事情整個變得非常混亂，所以 W3C 最終決定成立一個工作小組，DAP 工作小組則需要負責處理非常多的建議。

> DAP(Device Apis Policy)

## 第一回合: HTML 媒體截取
HTML Media Capture 是 DAP 第一個處理的是關於標準化網站上媒體截取這件事。它透過覆寫 `<input type='file'>` 加入新的屬性 `accept`。
例如: 如果想讓使用者透過 WebCam 截取一張快照那麼可能的寫法如下:
```
<input type="file" accept="image/*;capture=camera">
```
記錄影片或者聲音則類似:
```
<input type="file" accept="video/*;capture=camcorder">
<input type="file" accept="audio/*;capture=microphone">
```
看上去挺不錯的? 一般開發者可能蠻喜歡這樣的做法。不過這個方式沒辦法做到一些及時的效果如: 把正在記錄畫面的 WebCam 資料放進 Canvas 播放並且套用 WebGL 濾鏡。
HTML Media Capture 只能夠讓您記錄一個媒體檔案或者拍一張快照。

支援:
* Android 3.0
* Chrome for Android (0.16)
* Firefox Mobile 10.0
* iOS 6 Safari 和 Chrome (部分支援)

## 第二回合: 裝置元素
很多人覺得 HTML Media Capture 太多限制了，所以新的規範出現了: "支援各種類型的裝置"。這並不讓人感到意外，這個新的設計採用了一個新的元素 `<device>`
它也是 getUserMedia() 的前身。
Opera 是其中最早根據 <device> 元素實作影像截取的瀏覽器，不久後 WhatWG 決定放棄 <device> 標簽，而採用另一個方式，這次改用 Javascript API 稱為 `navigator.getUserMedia()`
而 <device> 用起來長得像:
```
<device type="media" onchange="update(this.data)"></device>
<video autoplay></video>
<script>
  function update(stream) {
    document.querySelector('video').src = stream.url;
  }
</script>
```

支援:
很不幸的，並沒有任何已釋出的瀏覽器曾經包含 <device> 。`<device>` 做了兩件很不錯的事: 1. 語意化 2. 容易擴展和支援更多的裝置。
深呼吸一下，這些東西進展的速度太快了。

## 第三回合: WebRTC
不管怎樣 `<device>` 最後的結局是 GG 了。

感謝 WebRTC 的努力加速了我們找到一個適用的截取 API。這份規範已經被 W3C WebRTC 工作小組所審查。Google, Opera, Mozilla 和其他開發者已經有了實作。
getUserMedia() 和 WebRTC 扯上關係是因為它是系列 API 的閘道，它提供了一種方式來存取使用者本地端的串流(Camera/Microphone stream)。

支援
getUserMedia() 已經在 Chrome 21, Opea 18, Firefox 17 版本支援。

## 快速入門
有了 `navigator.getUserMedia()`，我們終於可以觸及 WebCam 和麥克風等輸入而不需要安裝外掛。

### 偵測功能
偵測功能只不過是簡單的確認 navigator.getUserMedia 是否存在:

```
```

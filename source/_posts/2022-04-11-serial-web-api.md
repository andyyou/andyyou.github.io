---
title: Chrome 使用序列埠
date: 2022-04-11 15:18:10
tags:
  - javascript
  - chrome
  - serial-port
categories: Program
---

Web Serial API 連接裝置溝通。

> Web Serial API 為[新功能支援專案](https://web.dev/i18n/en/fugu-status/)的其中一部分，Chrome 89 已支援

<!-- more -->

## 何謂 Web Serial API?

序列埠是雙向溝通的介面支援傳送和接收。

Web Serial API 為網頁提供一種方式通過 JavaScript 讀取和寫入到連接裝置。連接裝置就是裝置連接到系統上的系列埠或者可移除的 USB 和藍芽模擬的系列埠。

換句話說 Web Serial API 連接了網頁和實體裝置使網頁可以和裝置例如微控制器或 3D 列印機溝通。

當作業系統要求應用程式使用 Serial API 而不是 USB API 來溝通時，Web Serial API 也是 [WebUSB](https://developers.google.com/web/updates/2016/03/access-usb-devices-on-the-web) 的好夥伴。

> 序列埠是依序通訊界面，在 60 年代初期推出。用於連續傳送和接收資料，一次傳輸一個 byte。常見的序列埠有 RS-232 ，60 年代的桌機幾乎都是標配，其他標準包含 RS-422 RS-485。雖然主流的電腦已經淘汰使用，但這些埠在產業還是很常見。大部分的序列埠使用 9-pin DB-9 連接器，通常稱為 COM 埠。
>
> 過去 COM 埠被用在連接硬體和電腦。硬體元件包含麥克風，數據機，印表機。然而今日 COM 埠大多只出現在工業環境。
>
> 序列埠非常可靠這已經被驗證了 60 多年。而 COM 埠最大的限制就是資料傳輸非常小，例如 RS-232 只有 1Mbps。
>
> 而 USB ，簡言之是 Universal Serial Bus 通用序列匯流排，也是一個埠支援電腦和硬體溝通。
>
> 總而言之，序列埠的設計是提供一個簡單的一對一連線，可靠，但速度不快。而 USB 提供了高速資料傳輸且一個埠可以掛載裝置

![](http://i1.kknews.cc/RJQ0DJB5Htoo9zR9UNnTYzqVsq_AEAotrnMEAaS-_5E/0.jpg)

## 建議使用情境

在教育，興趣，以及工業部門使用者會連接外部設備到電腦。這些裝置通常由微控制器透過客製的軟體使用序列連線來控制。而某些客製軟體是使用 Web 技術開發的。

- [Arduino Create](https://create.arduino.cc/)
- [Betaflight Configurator](https://github.com/betaflight/betaflight-configurator)
- [Espruino Web IDE](http://espruino.com/ide)
- [Microsoft MakeCode](https://www.microsoft.com/en-us/makecode)

有些情況下 Web 是通過手動安裝的代理應用程式和裝置通訊的。或者使用框架例如 Electron 打包的應用程式。還有使用者需要執行額外步驟例如複製編譯的應用程式到電腦。

上面這些情境，都可以利用提供讓網頁和裝置直接溝通來改善使用體驗

## 使用 Web Serial API

### 偵測功能

要檢查瀏覽器是否支援 Web Serial API

```js
if ('serial' in navigator) {
  // The Web Serial API is supported
}
```

### 開啟序列埠 - 連線

Web Serial API 在設計上是非同步的。主要是因為資料隨時都可以發送和接收，如此可以防止網頁 UI 阻塞。

要開啟序列埠首先需要存取 `SerialPort` 物件。為此，您可以調用 `navigator.serial.requestPost()` 開啟瀏覽器提示視窗選擇埠或者使用 `navigator.serial.getPorts()` 從已經授權的序列埠列表中選擇其中一個

```js
document.querySelector('button').addEventListener('click', async () => {
  const port = await navigator.serial.requestPort();
});
```

```js
const ports = await navigator.serial.getPorts();
```

`navigator.serial.requestPort()` 函式支援一可選物件參數可以定義過濾條件。可以用於過濾利用 USB 連線的特定裝置，例如符合 USB 供應商(`usbVendorId`) 或產品識別(`usbProductId`)

```js
const filters = [
  { usbVendorId: 0x2341, usbProductId: 0x0043 },
  { usbVendorId: 0x2341, usbProductId: 0x0001 },
];

const port = await navigator.serial.requestPort({ filters });

const { usbProductId, usbVendorId } = port.getInfo();
```

![](https://web-dev.imgix.net/image/admin/BT9OxLREXfb0vcnHlYu8.jpg?auto=format&w=964)

呼叫 `requestPort()` 提示視窗，讓使用者選擇裝置然後回傳 `SerialPort` 物件。一旦取得 `SerialPort` 物件，執行 `port.open()` 搭配希望的調變速率 (`baudRate`)。`baudRate` 屬性設定序列發送資料的速度。這是用來描述每秒幾 bit (bps) 的單位。確認裝置的文件取得正確的值，如果設定錯誤則發送和接受的資料會變亂碼。有些模擬序列埠的 USB 和藍芽裝置，這個值可以任意設定，因為會被忽略。

> **鮑**（Baud）即**調變速率**。鮑率可以被理解為單位時間內傳輸符號的個數（傳符號率）。以 RS232 為例通常是 300, 1200, 2400, 9600, 19200, 38400, 115200 等。假設設定為 9600 則等於每秒 9600 bit 的傳輸速率。

```js
const port = await navigator.serial.port.requestPort();

await port.open({ baudRate: 9600 });
```

在連接序列埠時，您也可以設定其他屬性；

- `dataBits`: 每幀的資料 bit 數量(7 或 8)
- `stopBits`: 幀尾的停止位數(1 或 2)
- `parity`: 同位元檢查模式 (`none`, `even`, `odd`)
- `bufferSize`: 讀寫緩衝區的大小(必須小於 16 MB)
- `flowControl`: Flow 控制模式(`none` 或 `hardware`)

## 讀取序列埠

Web Serial API 的輸入和輸出的串流由 Streams API 處理

> 如果串流對您來說比較陌生可以參考 [Streams API 概念](https://developer.mozilla.org/docs/Web/API/Streams_API/Concepts)。本文只涉及基本的串流處理。

在序列埠連線建立之後， `SerialPort` 物件的 `readable` 和 `writable` 會回傳 [ReadableStream](https://developer.mozilla.org/docs/Web/API/ReadableStream) 和 [WritableStream](https://developer.mozilla.org/docs/Web/API/WritableStream)。它們可以用來對裝置接收和傳送資料，資料傳輸使用 `Uint8Array` 物件實例。當裝置傳來資料時，`port.readable.getReader().read()` 會非同步的傳送兩個屬性; `value` 和 `done` 。如果 `done` 為 `true` ，那麼序列埠表示已關閉或沒有其他資料了。執行 `port.readable.getReader()` 會建立讀取器和鎖住 `readable` ，當 `readable` 鎖住時，序列埠不能關閉。

```js
const reader = port.readable.getReader();

// 監聽序列埠裝置
while (true) {
  const { value, done } = await reader.read();
  if (done) {
    reader.releaseLock();
    break;
  }
  // 值為 Unit8Array
  // 轉為字串
  // new TextDecoder().decode(value)
  // String.fromCharCode.apply(null, value)
  console.log(value);
}
```

一些非關鍵的讀取錯誤可能會發生例如緩衝溢出，偵發生錯誤，或同位元檢查錯誤(奇偶校驗)。這些例外可以透過在目前迴圈的外層檢查 `port.readable` 加入另外一個迴圈來擷取。這是因為只要不是嚴重錯誤 [ReadableStream](https://developer.mozilla.org/docs/Web/API/ReadableStream) 都會自動建立。如果是嚴重的錯誤例如裝置移除則 `port.readable` 會變成 `null` 。

```js
while (port.readable) {
  const reader = port.readable.getReader();

  try {
    while(true) {
      const { value, done } = await reader.read();
      if (done) {
        reader.releaseLock();
        break;
      }
      if (value) {
        console.log(value)
      }
    }
  } caatch (error) {
    // 處理非關鍵錯誤
  }
}
```

如果裝置傳回文字，你可以將 `port.readable` Pipe 給 `TextDecoderStream` 如下。一個 `TextDecoderStream` 是一個[轉換的串流](https://developer.mozilla.org/docs/Web/API/TransformStream)可以讀取 `Unit8Array` 並將它們轉換位字串。

```js
const textDecoder = new TextDecoderStream();
const readableStreamClosed = port.readable.pipeTo(textDecoder.writable);
const reader = textDecoder.readable.getReader();

// 監聽資料
while (true) {
  const { value, done } = await reader.read();

  if (done) {
    reader.releaseLock();
    break;
  }

  console.log(value);
}
```

## 寫入資料

要傳資料給裝置，需傳入資料到 `port.writable.getWriter().write()`。當需要關閉的時候可以執行 `port.writable.getWriter()` 的 `releaseLock()` 。

```js
const writer = port.writable.getWriter();

const data = new Unit8Array([104, 101, 108, 108, 111]); // hello
await writer.write(data);

// 允許埠稍後關閉
writer.releaseLock();
```

下面範例將 `TextEncoderStream` Pipe 給 `port.writable`

```js
const textEncoder = new TextEncoderStream();
const writableStreamClosed = textEncoder.readable.pipeTo(port.writable);

const writer = textEncoder.writable.getWriter();
await writer.write('hello');
```

## 關閉埠

執行 `port.close()` 若 `readable` 和 `writable` 已經解鎖 `releaseLock()` ，則關閉序列埠。

```js
await port.close();
```

然而，黨持續使用迴圈讀取裝置資料時，`port.readable` 會一直鎖住知道出錯。這種情況執行 `reader.cancel()` 會強制 `reader.read()` 立刻 resolve 回 `{ value: undefined, done: true }` 然後允許迴圈執行 `reader.releaseLock()`

```js
// 沒有搭配 transform stream
let keepReading = true;
let reader;

async function readUntilClosed() {
  while (port.readable && keepReading) {
    reader = port.readable.getReader();

    try {
      while (true) {
        const { value, done } = await reader.read();
        if (done) {
          break;
        }
      }
      console.log(value);
    } catch (error) {
      // 處理錯誤
    } finally {
      reader.releaseLock();
    }
  }

  await port.close();
}

const closedPromise = readUntilClosed();

document.querySelector('button').addEventListener('click', async () => {
  keepReading = false;
  reader.cancel();
  await closedPromise;
});
```

如果使用 Transform Streams (`TextDecoderStream`, `TextEncoderStream`)則關閉會更複雜一點。`reader.cancel()`，`writer.close()`，`port.close()` 要依序調用。錯誤會通過 Transform Stream 到序列埠。因此傳遞不會立刻發生，您需要 `readableStreamClosed` 和 `writableStreamClosed` ; 稍早利用 `port.readable` 和 `port.writable` 建立的 Promise 。

取消讀取會使串流中止; 這也是為何需要 catch 並忽略錯誤

```js
// 使用 Transform stream

const textDecoder = new TextDecoderStream();
const readableStreamClosed = port.readable.pipeTo(textDecoder.writable);
const reader = textDecoder.readable.getReader();

while (true) {
  const { value, done } = await reader.read();
  if (done) {
    reader.releaseLock();
    break;
  }
  console.log(value);
}

const textEncoder = new TextEncoderStream();
const writableStreamClosed = textEncoder.readable.pipeTo(port.writable);

reader.cancel();
await readableStreamClosed.catch(() => {
  /* Ignore the error */
});
writer.close();
await writableStreamClosed;

await port.close();
```

## 監聽連線與中斷

如果序列埠是由 USB 裝置提供，那麼裝置可能隨時和系統連線或中斷。當網頁獲得存取權限應可以監聽 `connect` 和 `disconnect` 事件。

```js
navigator.serial.addEventListener('connect', (event) => {
  // TODO: Automatically open event.target or warn user a port is available.
});

navigator.serial.addEventListener('disconnect', (event) => {
  // TODO: Remove |event.target| from the UI.
  // If the serial port was opened, a stream error would be observed as well.
});
```

> Chrome 89 之前 `connect` 和 `disconnect` 事件會觸發一個自訂的 `SerialConnectionEvent` 物件和一個 `SerialPort` 介面的 `port` 屬性。您可以使用 `event.port || event.target` 來處理過度狀況。

## 處理訊號

在建立連線之後，您可以利用虛列埠查詢和設定訊號來檢查裝置或控制。這些訊號為布林值。例如一些裝置如 Arduino 如果進入編程模式 DTR 訊號會啟動。

設定 [輸出訊號](https://wicg.github.io/serial/#serialoutputsignals-dictionary) 和取得 [輸入訊號](https://wicg.github.io/serial/#serialinputsignals-dictionary) 分別由 `port.setSignals()` 和 `port.getSignals()` 完成

```js
// 關閉中斷訊號
await port.setSignals({ break: false });

// 啟動 DTR 訊號
await port.setSignals({ dataTerminalReady: true });

// 關閉 RTS 序號
await port.setSignals({ requestToSend: false });
```

```js
const signals = await port.getSignals();
console.log(`Clear To Send:       ${signals.clearToSend}`);
console.log(`Data Carrier Detect: ${signals.dataCarrierDetect}`);
console.log(`Data Set Ready:      ${signals.dataSetReady}`);
console.log(`Ring Indicator:      ${signals.ringIndicator}`);
```

## 轉換串流 Transforming Streams

當您從序列埠接收資料時，不一定會一次取得全部資料。它可能是任意片段。更多資訊可以參考 [Streams API 概念](https://developer.mozilla.org/docs/Web/API/Streams_API/Concepts)。

要處理這個問題，您可以使用一些內建的串流轉換，他們可以支援解析進來的串流資料並回傳解析後的資料。串流轉換位於裝置和具體讀取使用資料的迴圈之間。在資料使用之前可以套用任意轉換。可以把它想成一條產線，一個配件進來，每一步驟都可以調整。

例如建立一個將串流根據斷行分組的轉換類別。其 `transform()` 每次有資料進來就會被調用。它可以將資料送到佇列或存下稍後在處理。`flush()` 在串流關閉時會呼叫，並處理還沒處理的資料。

要使用串流轉換類別，您需要將串流 pipe 給它。在我們上面的範例我們將原來的輸入 pipe 給 `TextDecoderStream`，我們需要使用 `pipeThrough()` 給新的 `LineBreakTransformer`

```js
class LineBreakTransformer {
  constructor() {
    this.chunks = '';
  }

  transform(chunk, controller) {
    this.chunks = chunk;

    const lines = this.chunks.split('\r\n');
    this.chunks = lines.pop();
    lines.forEach((line) => controller.enqueue(line));
  }

  flush(controller) {
    controller.enqueue(this.chunks);
  }
}
```

```js
const textDecoder = new TextDecoderStream();
const readableStreamClosed = port.readable.pipeTo(textDecoder.writable);
const reader = textDecoder.readable
  .pipeThrough(new TransformStream(new LinkBreakTransformer()))
  .getReader();
```

要替裝置 debug ，可以使用 `tee()` 方法來分開串流。兩個串流的建立和使用各自獨立，我們可以將其中一個導到 Console 檢查。

```js
const [appReadable, devReadable] = port.readable.tee();
```

## 開發小技巧

在 Chrome 使用內建頁面對 Web Serial API 除錯也很簡單，開啟 `about://device-log` 您可以看到所有序列裝置相關的事件

![](https://web-dev.imgix.net/image/admin/p2T9gxxLsDWsS1GaqoXj.jpg?auto=format&w=1098)

## Codelab

在 [Google Developer Codelab](https://codelabs.developers.google.com/codelabs/web-serial) 您可以使用 Web Serial API 和 [BBC micro:bit](https://microbit.org/) 互動

## 瀏覽器支援

Web Serial API 在桌機 (Chrome OS, Linux, macOS, Windows) 版本的 Chrome 支援。

## Polyfill

在 Android 上可以使用 [Serial API Polyfill](https://github.com/google/web-serial-polyfill) 搭配 WebUSB API 和 USB 的序列埠溝通。該 polyfill 僅限於可使用 WebUSB API 存取的設備硬體和作業系統，因為它尚未被內置設備驅動程式支援。

## 安全與隱私

規格作者設計與實作 API 的核心原則定義在 [Controlling Access to Powerful Web Platform Features](https://chromium.googlesource.com/chromium/src/+/lkgr/docs/security/permissions-for-powerful-web-platform-features.md) 包含控制，透明度，人體工程。使用此 API 主要由權限模型控制，該權限模型一次僅授予對單個設備的訪問權限。為響應用戶提示，用戶必須採取主動步驟來選擇特定的串行設備。

## 反饋

Chrom 團隊很歡迎聽到關於 Web Serial API 您的想法和經驗。

### 關於 API 設計

如果有關於 API 無法如預期運作? 或其他未支援的方法或屬性需求? 可在 [Web Serial API Github repo](https://github.com/wicg/serial/issues) 新增您的想法。

### 問題回報

是否發現相關實作 Bug? 可至 [https://new.crbug.com](https://bugs.chromium.org/p/chromium/issues/entry?components=Blink>Serial) 。確認提供足夠資訊和提供簡易重現問題的步驟

## 資源

- [Read from and write to a serial port](https://web.dev/serial/)

## 效能工具
React 通常效能出乎意料的快，然而在某些情況下您還是需要盡可能的擠出效能，而官方也提供 `shouldComponentUpdate` 程式預置區塊給您優化 React 的差異比對邏輯。
除了給你程式整體性能的概述，也提供 ReactPerf 這個分析工具以準確的分析關於您寫在這些 Hook 中的程式碼。

 > 開發版的 React 相對于發行版是比較慢的，因為包含了一些額外的邏輯，舉例: React 提供了一些友善的 console 提醒(這些東西會在發佈時被移除)。
   關於分析工具只是用來找出在程式中那些特別吃效能的地方。

## 一般 API
這份文件中的 `Perf` 即 `React.addons.Perf` 我們可以在開發模式下透過 `react-with-addons.js` 匯入。

### Perf.start() 和 Perf.stop()
開始/停止測量。測量開始與結束之間的 React 操作時間。無關緊要的操作花費時間將被忽略。

```js
React.addons.Perf.start();

React.renderComponent(
  <iPhone />,
  document.getElementById('example')
)

React.addons.Perf.stop();
React.addons.Perf.printWasted();
```

### Perf.printInclusive(measurements)
輸出概略的花費時間。如果沒傳入任何參數，預設為列出所有包含在內部的量測記錄。

![](http://i.imgur.com/iqto1PI.png)

### Perf.printExclusive(measurements)
"獨佔"時間不包含元件處理 `getInitialState`, `componentWillMount` 等等的時間。

> Firefox 的檢測器可能目前無法使用，因其不支援 cosole.table 請改用 Firebug 或 Chrome
![](http://i.imgur.com/AVpr1n5.png)

### Perf.printWasted(measurements)
實務上最常使用的方法，"耗費"時間指的是處理元件輸出需耗費時間，但實際上並不是真的輸出。

比較三種時間格式
![](http://i.imgur.com/1mqb5Kg.png)


### Perf.printDOM(measurements)
列出底層 DOM 的操作過程
![](http://facebook.github.io/react/img/docs/perf-dom.png)

## 進階 API
上面用來輸出的方法都可以透過 `Perf.getLastMeasurements()` 來更改輸出的結果。

### Perf.getLastMeasurements()
取得量測單位參數設定陣列，根據上一次發動 start 和 stop 階段，這個陣列包含了一些物件看起來如下:
```js
{
  // The term "inclusive" and "exclusive" are explained below
  "exclusive": {},
  // '.0.0' is the React ID of the node
  "inclusive": {".0.0": 0.0670000008540228, ".0": 0.3259999939473346},
  "render": {".0": 0.036999990697950125, ".0.0": 0.010000003385357559},
  // Number of instances
  "counts": {".0": 1, ".0.0": 1},
  // DOM touches
  "writes": {},
  // Extra debugging info
  "displayNames": {
    ".0": {"current": "App", "owner": "<root>"},
    ".0.0": {"current": "Box", "owner": "App"}
  },
  "totalTime": 0.48499999684281647
}
```

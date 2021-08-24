---
title: 新世代建置工具解析(esbuild、Snowpack、Vite、wmr)
date: 2021-04-25 19:09:22
tags:
  - javascript
categories: Program
---

> 本文為翻譯自 Comparing the New Generation of Build Tools 一文加上更新補充。因建置工具更新速度關係，如遇部分內容無法實作或不一致請參考各官方文件。

過去的一年出現了一堆新的開發工具，它們緊跟著那些目前佔據前端世界的建置工具例如：webpack，Babel，Rollup，Parcel，create-react-app。

這些新工具的目的並不是實現一模一樣功能，它們各自有不同想嘗試解決的問題或新增的功能。儘管有所不同，但這些工具都有著共同的目標：改善開發者體驗。

具體來說，本文希望試著分析每個新工具，概述它們的功能，以及為什麼我們需要它們和適用的情境。我知道項目之間的比較很難非常公平，再一次強調這裡比較的項目，它們的關係並非直接的競品。實際上例如 Snowpack 底層就有使用 esbuild 來處理某些事情。

本文的目的是更進一步了解這些工具的前景和它們提供什麼功能可以讓我們簡化開發流程。如此一來我們就知道能有什麼選擇，以及它們是如何組成的，然後在需要的時候我們可以做出比較好的決定。

當然這些論述有受到我使用 React 和 Preact 經驗的影響。個人比較熟悉這些函式庫，但我們也會探討對其他前端函式庫的支援和使用。

關於這些新工具網路上有許多不錯的文章，影片，廣播等。推薦 [ShopTalk Show Episode 454](https://shoptalkshow.com/454/) 探討關於 Vite 以及 [Episode 448](https://shoptalkshow.com/448/) Snowpack 和 wmr 作者的介紹。從這些影片介紹我們知道這些工具如何加速優化了我們的開發工作。

<!-- more -->

## 為什麼這些工具會出現？

某部分來說我認為這些工具反應了 JavaScript 工具鏈的問題 - [關於在 2016 學習 JavaScript](https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f) 一文很好的詮釋了問題。這些工具填補了撰寫一個原始 JavaScript 檔案到下載相依函式庫之際的環節。這些新工具內建功能，不用額外再安裝其他相依套件，部分符合了 JavaScript 生態圈的趨勢 - [Collapsing Layers](https://www.swyx.io/js-third-age/)。

Snowpack，Vite 和 wmr 已經可以在瀏覽器提供原生的 JavaScript 模組功能。回到 2018，Firefox 60 釋出預設提供 ECMAScript 2015 模組。之後，其他主流瀏覽器也都支援。Node.js 也在 2019 年 11 月搭載原生模組功能。直到 2021 年的今天我們依然在解開 JavaScript 原生模組的可能性。

## 新工具與當前工具的差異

無論我們使用 webpack，Rollup，Parcel 哪一個，這些工具都在建置的過程中使用像是 Babel，TypeScript 等處理 `node_modules` 並從原始碼封裝了整個程式，然後產出封裝檔（Bundle）給瀏覽器使用。即便它們支援暫存和優化，這個過程還是降低了開發伺服器的速度。

Snowpack ，Vite 和 wmr 開發伺服器則不使用這種模式。相反，它們會等到瀏覽器遭遇 `import` 語句時才發出模組的請求。只有在請求發生之後，工具才會轉譯請求的模組和模組中匯入的東西，然後提供給瀏覽器執行。由於開發伺服器的工作量減少，因此加快了不少速度。

還有 esbuild 。首先，基本上它是個封裝打包工具，但它跟其他工具不一樣。透過避免大量轉譯和使用 Go 語言並行的特性，讓  esbuild 可非常快速的處理程式碼。

## 實驗

這裡使用一個 React 的範例然後使用每個工具重新建置。這個專案是 [Yogita Verma](https://yog9.github.io/portfolio/) 的 [Snap Shot](https://github.com/Yog9/SnapShot)

* [原版檔案庫](https://github.com/Yog9/SnapShot)
* [4個不同工具版本](https://github.com/Elliotclyde/build-tool-test)

後續我們會比較編譯的結果。重製並包含一些開發常用的套件如 React Router 和 axios 讓我們可以多了解測試其開發體驗。



## 功能比較

在我們深入介紹每一個工具之前，先提一下，它們都支援下列功能但支援的程度不同：

* 支援原生 JavaScript 模組
* TypeScript 編譯（不進行型別檢查）
* JSX
* 擴展套件 API
* 內建開發伺服器
* 支援 CSS-in-JS 函式庫

這些新工具都可以編譯 TypeScript 為 JavaScript，但即便型別錯誤還是會繼續編譯。針對型別檢查您可以安裝 TypeScript 然後執行 `tsc --noEmit` 或者在您的編輯器安裝套件來檢查。

## esbuild

esbuild 是 [Evan Wallace](https://github.com/evanw) （ Figma 的 CTO）開發的。其主要目的為提升建置速度，比起基於 Nodejs 的工具可達到  10 到 100 倍快。跟像是 create-react-app 這類型的工具比起來和開發體驗沒什麼太大關係，但後續出現了很多基於 esbuild 建置的工具，它們補上了其他功能例如 create-react-app-esbuild，estrella，Snowpack。它們都是使用 esbuild 來編譯程式碼。

esbuild 目前還沒 1.0 算是蠻新的專案，不建議直接在正式專案使用，但離 1.0 應該不遠了。同時 esbuild 搭配智能預設值提供了直觀的指令介面 API 。

### 適用情境

esbuild 顛覆了前端工具的世界。在大型專案中增加了幾倍的編譯速度是非常實用的。將來 1.0 版本的時候，估計可以為大型專案提供不少幫助，大量減少等待編譯的時間。

目前還是等待穩定版本再使用在專案的正式版本，但您的一些非正式專案可以開始使用。

esbuild 的超快速度無論您在開發什麼類型的專案都能有所幫助。減少等待建置的時間對於開發體驗肯定是加分的。如果考慮到加速應用程式的開發您也許想從其他比 esbuild 高階的工具起手。不然您還是需要花時間安裝相依套件和設定專案。另外一提如果想要盡可能最小化編譯檔案的大小，使用 Rollup 和 terser，它們產出的檔案稍微小一點。

### 設定

我們從一個比較單純的方式使用 esbuild 建置一個 React 專案開始；單純使用 npm 安裝 esbuild，react，react-dom 不使用其他相依套件。然後建立 `src/app.jsx` 和 `dist/index.html` 檔案。接著使用下面指令產生 `dist/bundle.js`

```sh
$ npx esbuild src/app.jsx --bundle --platform=browser --outfile=dist/bundle.js
```

> 注意：如果您檔案中包含 JSX 則檔名須為 `.jsx` 或加入參數 `esbuild app.js --bundle --loader:.js=jsx`

如果您使用瀏覽器開啟 `index.html` 時畫面是空白的且發生錯誤 `Uncaught ReferenceError: process is not defnied`。可以參考[官方文件](https://esbuild.github.io/getting-started/#bundling-for-the-browser)設定。該錯誤解決方式是加入

```
--define:process.env.NODE_ENV=\"production\"
```

封裝任何函式庫給瀏覽器使用時，如果需要 Nodejs 環境變數則需要定義 `define` 參數。Vue 2.0 也是相同的。Preact 則不會遇到這個問題，因為它沒有使用任何環境變數且，預設就可以在瀏覽器使用。

`.jsx` 檔案中預設就支援 JSX 語法，但需要匯入 React ，不過 esbuild 也支援自動匯入，設定請參考 [auto imports in JSX and/or configure JSX for Preact](https://esbuild.github.io/content-types/#jsx)

### 使用方式

esbuild 提供 `--serve` 參數支援開發伺服器，讓我們可以直接從記憶體直接存取 JavaScript 模組，確保瀏覽器不會讀取到舊版的模組。不過它不支援 Live、Hot Reload，因此在存檔之後您需要手動讓瀏覽器重新載入，開發體驗不是非常理想。

```sh
# 依據目前的專案結構您也可使用下列指令測試開發伺服器
$ npx esbuild src/app.jsx --bundle --outfile=dist/bundle.js --servedir=dist
```

除了內建的開發伺服器，我們也可以使用 [watch](https://esbuild.github.io/api/#watch) 功能搭配其他伺服器；每當檔案儲存的時候它會通知 esbuild 重新編譯。但我們需要伺服器的支援來觀察內容的變更；這裡使用 [servor](https://www.npmjs.com/package/servor)。

```sh
$ npm i servor -D
```

然後我們就可以在啟動伺服器的同時執行 esbuild 的 watch 模式；讓我們來建立 `watch.js`

```js
const esbuild = require('esbuild');
const servor = require('servor');

esbuild.build({
  entryPoints: ['src/app.jsx'],
  outdir: 'dist',
  bundle: true,
  define: {
    'process.env.NODE_ENV': '"production"',
  },
  watch: true,
});

async function serve() {
  console.log('running server on: http://localhost:8000');
  await servor({
    browser: true,
    root: 'dist',
    port: 8000,
  });
}

serve();
```

接著可以在指令介面執行 `node watch.js` 。`servor` 是不錯的開發伺服器，但沒有支援 Hot Module Replacement，只是對於我們的測試來說已經足夠了。

雖然每次我們存檔的時候都會重建整個應用程式，也需要重新載入。但在這個設定之後，即使電腦配備不高速度也蠻順暢的。想要支援 Live Reload 的話，可以參考這個[檔案庫](https://github.com/Elliotclyde/esbuild-react-starter)。

### 支援檔案

 esbuild 可以在 JavaScript 匯入 CSS。可以編譯 CSS 並輸出一個跟產出 JavaScript 同檔名的檔案。預設支援打包 CSS `@import` 語句。不支援 [CSS Modules](https://css-tricks.com/css-modules-part-1-need/)，但未來是有計畫支援的。

esbuild 套件社群持續在成長。例如有 [Vue 單檔元件](https://github.com/few-far/esbuild-vue-plugin) 和 [Svelte 元件](https://github.com/EMH333/esbuild-svelte)。

esbuild 可以處理 JSON 檔案，無須進行任何設定即可使用。

可以在 JavaScript 匯入圖片，並設定轉換成 Data URL 或者把它們複製到輸出目錄。但這個功能預設不提供，您可以加入下面的設定：

```js
loader: {'.png': 'dataurl'}
loader: {'.png': 'file'}
```

處理過程中可能會顯示 Code Splitting 的訊息，多數會以 esm 格式輸出。另外值得一提的是內建支援 Tree-shaking （刪除沒用到的程式碼）而且不能關閉。

### 正式版本建置

使用 `minify` 和 `bundle` 參數無法建置出和 Rollup / Terser 一樣小的檔案。這是因為 esbuild 犧牲了一些優化的部分來減少讀取傳入的程式碼，但差異應該不大。加速編譯時間是值得的。在上面提到的 Sanp Shot 專案，esbuild 封裝檔是 177KB，而使用 Rollup 和 terser 的 Vite 產生的是 165KB 。

### 分析

| esbuild 功能                                                 | 支援   |
| ------------------------------------------------------------ | ------ |
| 各種前端框架專案建置樣版                                     | X      |
| 開發伺服器支援 Hot Module Replacement                        | X      |
| [Streaming Imports](https://www.snowpack.dev/guides/streaming-imports) | X      |
| 預先配置正式環境建置                                         | X      |
| 自動支援 PostCSS 和 Preprocessor 轉換                        | X      |
| 轉譯產生 HTML                                                | X      |
| 支援 Rollup 套件                                             | X      |
| 檔案大小                                                     | 7.34MB |

esbuild 是一個非常強大的工具，但需要自行進行設定配置。如果您希望預設提供更多功能則可以參考下一個工具 Snowpack 其底層就是使用 esbuild。

> 定位：An extremely fast JavaScript bundler

## Snowpack

Snowapck 是由 [Skypack](https://www.skypack.dev/) 和 [Pika](https://www.pika.dev/) 的作者開發的建置工具。核心功能是開發時期支援 Unbundled Development ，其概念是在開發時提供瀏覽器個別的檔案。檔案依舊可以使用 Babel，TypeScript，Sass 編譯然後由瀏覽器個別載入，也就是當您變更檔案時 Snowpack 只會重新編譯該檔，然後只重新載入該檔。節錄官方文件的說法：[使用封裝工具應該是您想要使用，而不是必須要使用。](https://www.snowpack.dev/concepts/build-pipeline#bundle-for-production)

其中一個賣點就是加速開發。

![](https://www.snowpack.dev/img/snowpack-unbundled-example-3.png)

預設，Snowpack 不會將所有程式碼封裝打包成一個檔案，瀏覽器載入個別檔案。雖然 esbuild 確實是其中一個相依套件，但 Snowpack 的想法是使用原生 JavaScript 模組，直到你需要封裝成一個檔案的時候才使用 esbuild。

Snowpack 擁有美觀的官方文件包含[搭配其他框架的設定說明](https://www.snowpack.dev/guides)和[專案樣版](https://github.com/snowpackjs/snowpack/tree/main/create-snowpack-app)。一些教學還處於編寫中，已完成的像 [React 教學](https://www.snowpack.dev/tutorials/react) 就非常清楚。另外 Snowpack 似乎以 [Svelete 為第一優先](https://www.snowpack.dev/tutorials/svelte/)。事實上，我第一次聽說 Snowpack 就是在 Svelte Submit 2020， Rich Harris 的 [未來的網頁開發](https://www.youtube.com/watch?v=qSfdtmcZ4d0&t=636s)。當時提到即將推出的 [SvelteKit](https://svelte.dev/blog/whats-the-deal-with-sveltekit) 應該會使用 Snowpack；後來選擇了 Vite - [SvelteKit is in public beta 說明](https://svelte.dev/blog/sveltekit-beta)

### 適用情境

想嘗試 Unbundle Deployment  不想增加封裝檔的複雜度， Snowpack 是不錯的選擇。如果您正在開發的專案模組量不大，意味著不會產生大量個別編譯的需求。一個不錯的使用情境是；您正逐步採用 SSR 或靜態應用程式。您可以只使用一點點 Node 生態圈的工具，但仍保留前端框架的好處。

第二，我認為 Snowpack 是一個不錯的 esbuild 強化版。如果您想使用 esbuild 又想要好用的開發伺服器和專案樣版，那麼選 Snowpack 不會錯。

就目前為止的情況，我認為 Snowpack 並不是零設定方案最好的選擇（例如 create-react-app），因為如果您面對的是大型專案，還是需要為了優化安裝套件並自行設定。

### 設定

讓我們直接使用指令搭配 Snowpack 來開啟一個專案

```sh
$ mkdir demo
$ cd demo
$ npm init -y
$ npm i snowpack
```

然後調整 `package.json` 加入 `scripts`

```json
"scripts": {
  "start": "snowpack dev",
  "build": "snowpack build"
},
```

接著，建立設定檔

```sh
$ touch snowpack.config.js
```

我認為 Snowpack 神奇的地方就是設定看起來相對單純，例如

```json
// snowpack.config.js
module.exports = {
  packageOptions: {
    source: 'remote',
  },
};
```

`source: remote` 啟動了 Streaming Imports 的功能。Streaming Imports 讓 Snowpack 將我們的模組匯入例如 `import React from 'react'` 轉換成從 [Skypack](https://www.skypack.dev/) 讀取（即不用先 `npm install`）。

下一步，建立 `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Hello, Snowpack</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="app.js"></script>
</body>
</html>
```

最後建立 `app.jsx`

```jsx
import React from 'react';
import { render } from 'react-dom';

const App = () => {
  return (
    <div>
      Hello, Snowpack!
    </div>
  );
};

render(
  <App />,
  document.getElementById('app'),
);
```

神奇的地方來了！現在我們不需要使用 `npm` 安裝 `react` 和 `react-dom` 直接使用下面的指令啟動開發伺服器：

```sh
$ npm start
```

我們的範例就可以正常執行了！

Snowpack 從 Skypack CDN 讀取 npm 套件，而不是 `node_modules` 。模組還是預先為瀏覽器優化過的版本，後續 Snowpack 會使用 `./_snowpack/pkg` URL 來讀取。

![](https://i.imgur.com/4Q8jfcL.png)



![](https://i.imgur.com/RYDqioS.png)

### 使用方式

光是改變從 npm 讀取模組的流程就是一個很大的變動。實際上有部分開發者正在研究新的，基於 CDN 載入的流程。

如果現在我們直接使用一樣的方式建置正式環境的檔案（ `npm run build`），則 Snowpack 會拋出錯誤。這是因為 Snowpack 從 3.0 版本開始支援 Streaming Imports，但在 3.1 版之前您需要在建置正式版本檔案之前進行  `snowpack.deps.json` 設定相依版本。可以使用下面指令來設定：

```sh
$ npx snowpack add react
$ npx snowpack add react-dom
```

> 其他細節可以參考[相依性管理](https://www.snowpack.dev/guides/streaming-imports#snowpack-managed-dependencies)

上面指令不會從 npm 安裝套件，但會紀錄使用的套件版本。

如果我們不用 Streaming Imports，Snowpack 開發伺服器也可以從 `node_modules` 讀取模組，並將這些檔案專案為原生 JavaScript 模組給瀏覽器使用。這表示瀏覽器可以暫存這些檔案直到檔案有變更。開發伺服器會在檔案變更時自動重新載入，但不會保持客戶端的狀態。全部相依的套件無論使用傳統模組格式或 Nodejs API 的格式內建都支援。例如我們在上面 esbuild 的 `process.env` 問題都會自動處理。

至於像是要在 React 中保留狀態則需要 [react-refresh](https://www.skypack.dev/view/@snowpack/plugin-react-refresh)，和一些 Babel 的套件處理。這功能預設不支援，但在很多 React 的專案樣版中都有使用。

```sh
# https://www.snowpack.dev/concepts/hot-module-replacement#enabling-hmr-%2B-fast-refresh
# https://github.com/snowpackjs/snowpack/tree/main/create-snowpack-app/cli

$ npx create-snowpack-app demo --template @snowpack/app-tempate-react
```

### 支援檔案

預設只支援 `.jsx`。Snowpack 會自動偵測是使用 React 還是 Preact，並以此決定 `render` 函式。不過如果您希望自訂 JSX 的功能您可以利用[套件](https://www.npmjs.com/package/@snowpack/plugin-babel)來安裝設定 Babel。當然 Snowpack 還支援 Vue ，Svelte，編譯 TypeScript（不會檢查型別，須使用額外 [TypeScript 套件](https://www.npmjs.com/package/@snowpack/plugin-typescript)）。

CSS 可以在 JavaScript 匯入會在執行時期置放到 `<head>`。CSS Module 也預設支援，只須使用 `.module.css` 副檔名。

JSON 檔案會直接編譯成一個物件。圖片則是會被複製到目錄下，因為 Unbundled 的設計原則，Snowpack 不會把圖片編譯成 Data URL。

### 正式版本建置

基本上預設 `snowpack build` 指令會把編譯結果放到輸出目錄下。例如 TypeScript，JSX，JSON，`.vue`，`.svelte` 檔案會被編譯成 JavaScript，檔案各自被轉換成 JavaScript 模組。

但這不是那麼適合正式環境，因為當檔案很多的時候，會導致產生大量的網路請求。例如在 Snap Shot 專案中我們檔案總共只有 184KB ，但後續卻向 Skypack 發出 105KB 相依套件的請求。

不過 Snowpack 內部使用 esbuild，因此我們可以使用 bundle，minify 等參數功能來編譯程式，只要在 `snowpack.config.js` 使用 `optimize` 物件參數即可

```js
module.exports = {
  optimize: {
    bundle: true,
    minify: true,
    target: 'es2018'
  },
};
```

如此就可以使用 esbuild 提供的優化功能。

由於 esbuild 還沒 1.0，針對正式版本建置， Snowpack 建議使用 webpack 或 Rollup 套件，當然這些都需要額外設定。

### 分析

通過完整功能的開發伺服器，詳細的文件，輕鬆安裝的專案樣版，這些功能讓 Snowpack 提供了輕量級的開發體驗。您可以自行決定封裝應用程式的方式。但如果您希望一個工具同時提供開發伺服器和更多建置功能，您可以參考下一個工具 Vite

| Snowpack 功能                                                | 支援               |
| ------------------------------------------------------------ | ------------------ |
| 各種前端框架專案建置樣版                                     | V                  |
| 開發伺服器支援 Hot Module Replacement                        | V （使用專案樣版） |
| [Streaming Imports](https://www.snowpack.dev/guides/streaming-imports) | V                  |
| 預先配置正式環境建置                                         | X                  |
| 自動支援 PostCSS 和 Preprocessor 轉換                        | X                  |
| 轉譯產生 HTML                                                | X                  |
| 支援 Rollup 套件                                             | V                  |
| 檔案大小                                                     | 16MB               |

## Vite

Vite 是由 Vue 作者 Evan You 和 Hades speedruns 開發的。在 esbuild 專注在編譯速度，Snowpack 專注開發伺服器。Vite 則提供兩者；完整的開發伺服器和使用 Rollup 進行優化編譯。

### 適用情境

如果您在尋找的是像 create-react-app 或 Vue CLI 的競品，Vite 是最接近的一個，因為它內建包含這些功能。輕量快速的開發伺服器，零設定即支援正式版本優化。Vite  可以適用於小型的個人專案 Side-Project 或大型正式專案。任何 SPA 應用程式都很適合。

為什麼不使用 Vite？Vite 是一個堅持己見的工具，可能您不同意其中的一些觀點。比如您不想使用 Rollup，或想使用上面提到非常快的 esbuild，或希望預設能提供完整 Babel ，eslint ，和 webpack loaders 生態圈的功能。還有如果您想使用無須額外設定的 Meta-frameworks，那麼您最好繼續使用基於 webpack 的框架，例如 Nuxt.js，Next.js 直到 Vite 的伺服器端渲染功能更完整。

> Meta Framework: 一網頁應用程式框架具備常見功能的核心介面，可整合服務和元件的高度擴展性。開放式的結構可和其他元件或框架合併使用。例如 Next.js，Nuxt.js

### 設定

跟 esbuild 和 Snowpack 相比，Vite 包含更多主觀意見的預設值。因為作者的關係， Vue 得到了完全支援，對於 Vue 開發者來說無疑是非常友善的。當然  Vite 可以被使用在任何前端框架而且也提供了[一系列樣版](https://github.com/vitejs/vite/tree/main/packages/create-app)。

### 使用方式

Vite 的開發伺服器功能非常強大。Vite 預先使用 esbuild 封裝專案全部的相依套件成一個原生 JavaScript 模組，然後使用 HTTP Header 快取。這表示一旦頁面載入，我們就不需要浪費時間編譯，發送請求，讀取相依模組。

Vite 也提供清楚的錯誤資訊，顯示確切錯誤程式碼段落。使用 Vite 我不曾遇到任何關於讀取相依套件的問題，即便使用 Node API 或其他較舊的格式。它們全部都被轉換成瀏覽器相容的 ES Module。

Vite 的 React 和 Vue 專案樣版都有使用套件來支援 Hot Module Replacement 。Vue 使用 [@vitejs/plugin-vue](https://github.com/vitejs/vite/tree/main/packages/plugin-vue) 和 [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx) 。而 React 使用 [@vitejs/plugin-react-refresh](https://github.com/vitejs/vite/tree/main/packages/plugin-react-refresh) 。

無論哪種方式，它們都讓我們得到 HMR 和保留客戶端狀態的功能。當然它們使用了更多相依套件包含 Babel。實際上在 Vite 中使用 JSX 並不需要 Babel 。預設情況下 JSX 的處理方式和 esbuild 一樣；就是先轉換成 `React.createElement` 。

不會自動匯入 React ，但可以設定。

另外值得提到 Vite 也[支援伺服器端渲染](https://vitejs.dev/guide/ssr.html)（處於實驗階段）。目前我們需要自行組織架構，但看起來基於 Vite 構建框架是不錯的機會。作者確實已經著手進行一個叫 [VitePress](https://vitepress.vuejs.org/) 的專案，一個包含 Vite 優點，預計取代[VuePress](https://vuepress.vuejs.org/) 的專案。

還有 SvelteKit 也採用 Vite。看起來 CSS 拆分功能是 SvelteKit 後來選擇 Vite 的原因之一。

```sh
$ npm init @vitejs/app demo
$ cd demo
$ npm i
$ npm run dev
```

### 支援檔案

針對 CSS，Vite 提供幾乎所有工具的功能，打包 CSS ，CSS Module，也可以安裝 PostCSS 套件並設定 `postcss.config.js`，Vite 會自動套用這些轉譯功能。

其他 CSS Preprocessors 也可以使用 npm 安裝，例如 SCSS。只要副檔名正確，Vite 就會自動套用對應的處理工具。

圖片匯入預設會複製到輸出目錄並提供存取的 URL，但您可以使用 `?raw` 參數將它們轉換成字串存到 JavaScript 中。

JSON 檔案會轉換為 ES Module 並以單一物件形式匯出。還可以提供具名匯入，Vite 會根據 JSON 根節點的欄位搜尋，並移除其他不用的資料（Tree-shaking）。

### 正式版本建置

Vite 使用 Rollup 並預先提供設定，為正式版本建置進行優化。不用額外設定即可適用於大多數使用情境。

輸出的結果包含我們預期 Rollup 提供的功能；封裝，最小化，Tree-shaking。其他還有拆分程式碼，動態載入，非同步程式碼片段載入。舉例來說如果我們請求載入一個 JavaScript 模組，其匯入另一個模組時會預先對該模組進行優化讓兩個模組可以同一時間載入（非同步）。

使用 Vite 的預設值建置 Snap Shot 應用程式最終會得到一個 5KB 和一個 160KB 的 JavaScript 檔案，全部 CSS 自動最小化 2.71KB。

### 分析

為了使開發人員體驗真正無縫順暢的開發流程，預設就支援正式環境版本的建置，這個部分 Vite 做了很多的努力。Vite 已成為很多前端工具的競爭對手。

| Vite 功能                                                    | 支援               |
| ------------------------------------------------------------ | ------------------ |
| 各種前端框架專案建置樣版                                     | V                  |
| 開發伺服器支援 Hot Module Replacement                        | V （使用專案樣版） |
| [Streaming Imports](https://www.snowpack.dev/guides/streaming-imports) | X                  |
| 預先配置正式環境建置                                         | V                  |
| 自動支援 PostCSS 和 Preprocessor 轉換                        | V                  |
| 轉譯產生 HTML                                                | X                  |
| 支援 Rollup 套件                                             | V                  |
| 檔案大小                                                     | 17.1MB             |

## wmr

wmr 是另一個類似 Vite 的建置工具，也是同時提供開發伺服器和整合建置流程。是由 Preact 作者 Jason Miller 開發的。因此肯定對於 Preact 開發者友善。Jason Miller 在 [JS party podcast](https://changelog.com/jsparty/158) 當來賓的時候曾解釋背後的想法：

“當您在開發一個輕量的專案時 Preact 是不錯的選擇。但關於它的建置工具呢？它使用基於 webpack 的建置工具，外面很多專案也都使用 webpack ，但它的確是一個相對笨重的工具。那關於 Prototyping 的建置工具呢？其實那是一個原因，另外一直以來包含我自己在內和 Preact 團隊都沒有特別深入建置工具的生態圈，這些因素促使我們嘗試思考進一步的方向並在”編寫和交付符合主流規範的程式碼” 這個方向達成共識

意思是 wmr 希望輕量化的工具能協助“編寫和交付符合主流規範的程式碼”。

您可能會問 wmr 代表什麼意思？ Web Module Runtime 和 Web Module Replacement 名稱自然浮現，但其實不是它們的縮寫，就像 npm 其實也不是 Node Package Manager 一樣。

wmr 跟 Preact 一樣非常小，只有 2.6MB - 沒有任何 npm 相依套件。就算如此，它還是包含很多很棒的功能包含支援 HMR 的開發伺服器以及正式環境建置的優化。

### 適用情境

如果您要使用 Preact 來開發原型產品的話可使用 wmr 。不需要設定，下載也很快，感覺就像是開加速的靜態檔案伺服器。搭配 TypeScript ，優化建置，靜態 HTML 渲染，wmr 提供了小至中型專案所需要的功能。

如果您不是使用 Preact，React 或完全原生 JavaScript 那麼 wmr 就不太適合您。Preact 團隊有為其他框架提供專案樣版，但文件看來和其他工具相比不是那麼詳細，意味著您可能會踩雷，偶而需要深入原始碼。因此假如您需要大量客製化設定的話不推薦使用。

### 設定

如果您使用的是 Preact ，那如我們所預期不需要額外設定。如果是 React 搭配 wmr 則有兩個步驟，須將 `package.json` 中的 `alias` ， `htm/preact` 換成 `htm/react` 以及 `react` 換成 `es-react`

```json
"alias": {
  "htm/preact": "htm/react",
  "react": "es-react"
},
```

然後在元件中從 `es-react` 匯入 React

```js
import { React, ReactDOM } from 'es-react';
```

上面程式碼代表我們不是從一般的 React 套件中匯入，而是從 `es-react` 。因為 wmr 需要依賴相容原生 JavaScript 模組的套件。原廠 React 預設不使用原生模組，而是使用 UMD 模組格式，`es-react` 套件基本上只是把原廠套件使用 JavaScript 原生模組的方式匯出。

這說明了 wmr 的核心理念就是使用原生模組。另外也可以使用 Skypack 來匯入模組。

```js
import React from 'https://cdn.skypack.dev/react';
import ReactDOM from 'https://cdn.skypack.dev/react-dom';
```

wmr 預期您會以現代化標準的程式碼進行開發，它們可以直接在瀏覽器執行，同時如果您有使用一些傳統 Node API 格式的模組您可能需要額外的設定。還是以 Snap Shot 專案舉例，您就需要深入那些 Node 模組並將它們轉換成相容的原生模組。如果您使用了很多舊的函式庫，可能導致開發效率變差。而 Preact 生態圈相關的東西都已經處理了，也就是如果您使用 Preact ，wmr 肯定是比較友善的。

wmr 套件部分也支援 Rollup 建置，還有很多 wmr 的範例包含最小化 HTML，使用檔案系統的路由。另外，wmr 也支援其他框架，但沒有提供預先建置的專案樣版。且設定 JSX 轉換就相當困難。

話雖如此，Jason 也已經確定計畫讓 JSX 設定更加方便，還有 wmr 將不針對特定框架，預設支援 JSX 等。

### 使用方式

欲使用 wmr 您可以執行下面指令

```sh
$ npm init wmr demo
```

或者手動在既有目錄下

```sh
$ npm init -y
$ npm install wmr
$ mkdir public
$ touch public/index.html
$ touch public/index.js
```

然後在 `index.html` 中加入

```html
<script type="module" src="./index.js"></script>  
```

接著就可以直接在 `index.js` 中使用 Preact

```js
import { render } from 'preact';
render(<h1>Hello World!</h1>, document.body);
```

最後執行下面指令啟動支援 HMR 的開發伺服器

```sh
$ npx wmr
```

wmr 使用 [htm](https://github.com/developit/htm) 工具來編譯 JSX 可以提供一些不錯的功能。例如您使用 Preact 撰寫一個計數器然後在使用 wmr 的情況下寫錯了：

```js
import { render } from 'preact';
import { useState } from 'preact/hooks';

function App() {
  const [count, setCount] = useState(0);
  return (
  	<>
    	<button onClick={setCount(cout + 5)}>Click to add 5 to count</button>
			<p>count: {count}</p>
    </>
  );
}
render(<App />, document.body);
```

上面 `onClick` 中的 `count` 拼錯了，點擊按鈕執行的時候一定會發生錯誤。通常我們需要依靠 Source Map 的功能來取得錯在哪，但 wmr 不一樣。wmr 則使用 JS 樣版字串的方式，讓您在瀏覽器下盡可能以**接近 JSX 的格式**得到錯誤訊息。例如我們寫了一段 JSX

```jsx
<MyComponent>I am JSX. I am not actually valid Javascript</MyComponent>
```

htm 輸出的結果看起來像：

```
html`<${MyComponent}>I am about as close as it gets to JSX as you can get while being able to run in the browser</MyComponent>`
```

看不懂什麼意思？現在如果我們在瀏覽器的開發工具切換到 "Sources" 面板，您會看到如下的資訊：

![](https://i1.wp.com/css-tricks.com/wp-content/uploads/2021/03/s_864795E8DFCEBDF45A714B0A68D25726C4192DC77A541292A7CD86FFCB3E5238_1614545243859_wmr-in-browser.png?w=655&ssl=1)

當然這個範例有點故意，但如此一來您在除錯的時候看到的程式碼是不是更明確了。

wmr 預設支援 Streaming Imports ，因此可以直接從 npm registry 上直接下載。該流程非常複雜，會檢查 npm 套件，移除測試相關的部分，轉換成原生模組，類似於 Snowpack 可以不用 npm 安裝任何東西就建置複雜的專案。

其實 wmr 是第一個支援這個概念的工具。

### 支援檔案

至於 wmr 支援的檔案類型，CSS 可以在 JavaScript 中匯入，CSS Module 也支援。預設不支援 Vue 單檔元件或 Svelte 元件。然而 wmr 內部的 Rollup 和開發伺服器可以搭配 [Polka](https://github.com/lukeed/polka) / [Express](https://expressjs.com/) 的中介層來設定，因此它是可以支援其他框架元件的。實際上例如 [wmr-vue-plugin](https://github.com/Elliotclyde/wmr-vue-plugin) 就可以協助編譯 Vue 單檔元件。

“匯入圖片” 轉換 URL 的部分則額外的套件處理。一般則需要使用標準的 JavaScript 語法匯入圖片，例如我們要在 Preact 元件使用圖片：

```jsx
function Dog() {
  return (
  	<img src={new URL('./dog.jpg', import.meta.url)} alt="Dog handing out" />
  );
}
```

利用這種方式執行建置的時候，圖片會被複製到輸出的目錄。HMR 不支援圖片，所以變更圖片不會自動反應在瀏覽器上。

還有 JSON 是可以被匯入的，也是轉換成一個物件。但在建置的時候需要 [Rollup JSON 套件](https://github.com/rollup/plugins/tree/master/packages/json)。

### 正式版本建置

wmr 支援正式環境版本建置包含最小化，Tree-Shaking 。觀察原始碼似乎是使用 Rollup 和 terser，以 Snap Shot 專案來說結果是 164KB 比 Vite 小一點點。

另外還有一種搭配 [preact-iso](https://www.npmjs.com/package/preact-iso) 的方式也可以輸出靜態 HTML ，意味著 wmr 也可以協助 Preact 實作 Meta Framework。

### 分析

對於小型專案搭配 React 或 Preact 的話 wmr 的使用體驗還不錯。

| wmr 功能                                                     | 支援   |
| ------------------------------------------------------------ | ------ |
| 各種前端框架專案建置樣版                                     | V      |
| 開發伺服器支援 Hot Module Replacement                        | V      |
| [Streaming Imports](https://www.snowpack.dev/guides/streaming-imports) | V      |
| 預先配置正式環境建置                                         | V      |
| 自動支援 PostCSS 和 Preprocessor 轉換                        | X      |
| 轉譯產生 HTML                                                | V      |
| 支援 Rollup 套件                                             | V      |
| 檔案大小                                                     | 2.57MB |

## 功能比較

上面涵蓋了不少東西，實作比較和內容整理讓我們概略了解這些工具的特性，組成。下面附註一些上面可能有遺漏的比較：

### 適用情境

| 工具     |                                                              |
| -------- | ------------------------------------------------------------ |
| esbuild  | 適用於專案包含大量原始碼。目前不適用於正式專案               |
| Snowpack | 小型專案，可自行選擇是否需要封裝打包，也適用於逐步採用 JavaScript 框架，目前還是伺服器端渲染的專案 |
| Vite     | Vue Cli/create-react-app 的競品，特別適合 Vue 專案           |
| wmr      | 原型，小型到中型專案，特別適合 Preact 專案                   |

### 設定

|                              | esbuild | Snowpack | Vite   | wmr    |
| ---------------------------- | ------- | -------- | ------ | ------ |
| 各種前端框架專案建置樣版     | X       | V        | V      | X      |
| 檔案大小                     | 7.34MB  | 16MB     | 17.1MB | 2.57MB |
| 無須設定，支援正式版本建置   | X       | X        | V      | V      |
| 無須設定，支援HMR 開發伺服器 | X       | V        | V      | V      |
| 處理 Node 套件               | X       | V        | V      | V      |

### 開發伺服器

|                       | esbuild | Snowpack | Vite | wmr  |
| --------------------- | ------- | -------- | ---- | ---- |
| HMR                   | X       | V        | V    | V    |
| CSS HMR               | X       | V        | V    | V    |
| 預先建置 npm 相依套件 | X       | V        | V    | X    |
| 瀏覽器顯示錯誤訊息    | X       | V        | V    | X    |
| 轉譯產生 HTML         | X       | X        | X    | V    |

### 正式版本建置

|                        | esbuild | Snowpack                                   | Vite                    | wmr   |
| ---------------------- | ------- | ------------------------------------------ | ----------------------- | ----- |
| Snap Shot 專案建置大小 | 177KB   | 多檔總和 184KB 外加 Skypack CDN 相依 105KB | 165KB(5KB + 160KB 檔案) | 164KB |
| 使用 Go 編譯           | V       | V 當使用 esbuild 時                        | X                       | X     |
| 預先設定正式環境建置   | X       | X                                          | V                       | V     |
| 非同步 chunk 載入      | X       | X                                          | V                       | V     |
| 支援 Rollup            | X       | V                                          | V                       | V     |

### 其他

|                                   | esbuild | Snowpack | Vite | wmr  |
| --------------------------------- | ------- | -------- | ---- | ---- |
| Streaming imports                 | X       | V        | X    | V    |
| Server-side rendering             | X       | X        | V    | V    |
| CSS Modules                       | X       | X        | V    | V    |
| 自動 PostCSS 和 Preprocessor 轉換 | V       | V        | V    | X    |

## 結論

對於使用上面介紹的工具建置 JavaScript 專案，無論是小型個人專案或大型專案這些工具都提升開發效率。它讓我們知道 JavaScript 生態圈需要什麼，我們是否能免受傳統瀏覽器和模組帶來的困擾。這些工具將通過提供一個更精簡，更快速的開發環境，減少”撰寫的程式碼“和”瀏覽器執行版本“之間的抽象，進一步降低新進人員的門檻。

如果您正因為建置工具而困擾，建議您可以嘗試一下這些新工具。

## 參考

* [Comparing the New Generation of Build Tools](https://css-tricks.com/comparing-the-new-generation-of-build-tools/)
* [Comparisons with Other No-Bundler Solutions](https://vitejs.dev/guide/comparisons.html) (Vite)
* [Let’s Learn esbuild! (with Sunil Pai)](https://www.youtube.com/watch?v=KLdF1yu_bmI) (Jason Lengstorf)
* [Through the pipeline: an exploration of frontend bundlers](https://dev.to/walpolea/through-the-pipeline-an-exploration-of-front-end-bundlers-ea1) (Andrew Walpole)

## 其他新世代工具

- [Rome](https://rome.tools/) – 完整型工具包含 linting, compiling, bundling, test-running and formatting
- [SWC](https://swc.rs/) – 基於 Rust 語言 JavaScript/TypeScript 編譯器
- [Deno](https://deno.land/) – JavaScript and TypeScript 執行環境 (類似 Node.js)
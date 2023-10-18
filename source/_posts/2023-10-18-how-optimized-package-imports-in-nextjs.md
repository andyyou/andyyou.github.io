---
title: Next.js 如何優化匯入函式庫
date: 2023-10-18 14:33:32
tags:
  - javascript
  - nextjs
categories: Program
---

> 加速 40% 冷啟動和 28% 建置速度

在 Next.js 最新版本，官方優化了套件匯入，改善了本地開發的效能和正式環境冷啟動的速度。特別適用於使用大型的 Icon ，元件庫，或其他重複 Export 大量模組的相依套件。

<!-- more -->



## 何謂 Barrel File?

"Barrel File" 集合檔案，在 JavaScript 中是一種將多個模組彙整組合並從單一檔案匯出的方式。通過提供一個集中的位置來存取是導入模組更為簡單。

例如，我們有 3 個模組 `module1.js` `module2.js` `module3.js` 都放在 `utils/` 目錄下。我們可以在同一個目錄下建立一個 Barrel File 叫 `index.js` 

```js
export { default as module1 } from './module1';
export { default as module2 } from './module2';
export { default as module3 } from './module3';
```



現在，你不用再像下面範例這樣從不同路徑匯入

```js
import module1 from './utils/module1';
import module2 from './utils/module2';
import module3 from './utils/module3';
```



我們可以從 Barrel File 來匯入所有的模組

```js
import { module1, module2, module3 } from './utils';
```



Barrel File 通過提供單純的存取介面以及分類相關模組改善程式的組織性以及維護性。因此廣泛的被用在 JavaScript 套件，特別是 Icon 和元件函式庫。

一些廣泛被使用的 Icon 套件甚至有超過 10000 個 re-export 的 Barrel File。



## Barrel File 集合檔案有什麼問題？

在每個 JavaScript 執行環境中的 `require()` 和 `import ...` 都含有一些隱形成本。如果你希望使用單一檔案來匯出上千的東西，你仍然要付出其他不需要的模組匯入的代價。

對於主流的一些 React 套件，光是匯入平均花費 200~800ms 。在一些極端的案例甚至要幾秒。

> 例如 Material UI 10K 的匯出光是載入可能就要耗費 3.5 秒。

這些因素造成無論本地或產品的效能被影響，特別是 Serverless 的環境。每一次 App 啟動我們就必須要再次匯入全部的東西。



## 不能使用 Tree-shake 嗎？

> Tree-shaking 是一種程式碼優化手法，主要用在 JavaScript。簡單來說，它會幫你「搖掉」不需要的程式碼。假如你的專案裡引入了一個大型的函式庫，但其實只用到其中一小部分功能，Tree-shaking 會幫你刪除那些沒有用到的多餘程式碼，讓最終的檔案大小更小，加快加載速度。

Tree-shaking 是建置工具的功能（Webpack，Rollup，Parcel，esbuild），並不是 JavaScript 執行環境的功能。如果函式庫標記為 `external` ，那麼它會維持一個黑箱的狀態。建置工具不可以優化內部的東西，因為在執行環境會需要相依的東西。

也就是說，如果我們選擇也一起打包函式庫，而且在 `package.json` 沒有設定 `sideEffects` 那麼 Tree-shaking 確實可以使用。

然而，這會需要更多的建置時間。

![](https://vercel.com/_next/image?url=https%3A%2F%2Fimages.ctfassets.net%2Fe5382hct74si%2F51xcJ5VwnREoAkuI6JhUZ7%2F3cc76e57dac2c7a1c16652713966542e%2Fbundle__6_.png&w=1920&q=75&dpl=dpl_7ZJLWwLncYZA2K9u4vAC7TZySfzz)



## 第一個嘗試解法 - `modularizeImports`

我們第一個嘗試的方法叫做 `modularizeImports` 。這個方法讓我們可以設定對應 Barrel File “匯出名稱”和它們模組的原始路徑。換句話說，這個功能就是讓你自訂分散的模組會從哪一個集合檔案被匯入。

舉例來說如果 `my-lib` 的集合檔案叫 `index.js`

```js
export { default as module1 } from './module1';
export { default as module2 } from './module2';
export { default as module3 } from './module3';
```

我們可以設定一個編譯 `my-lib/{{member}}` 的轉換告訴 Next.js 把 `import { module2 } from 'my-lib'` 改成 `import module2 from './my-lib/module2'` 。這意味著我們可以跳過 Barrel File 直接匯入目標，防止載入不必要的模組。

這個改變同時讓建置時間以及執行環境變更更快。

![](https://vercel.com/_next/image?url=https%3A%2F%2Fimages.ctfassets.net%2Fe5382hct74si%2F2tpTkpBkvN8yj6T5olIgqc%2Fc29d6c982c990166a521a21443bdf167%2Fbundle__8_.png&w=1920&q=75&dpl=dpl_7ZJLWwLncYZA2K9u4vAC7TZySfzz)

然而，這個設定是基於函式庫內部目錄結構以及你需要繁重的手動設定。成千上萬的 `npm` 套件還有不同版本，這個方法無法有效率的擴展。

如果建置工具針對流行的函式庫包含一個預設的設定而不鎖定版本的話，那麼當該函式庫未來的內部結構發生變化時，這會使轉換無效。因此需要一個更好的解決方案。



## 新的解決方案 - `optimizePackageImports`

為了解決設定 `modularizeImports` 這個困難的問題，我們引進了新的 `optimizePackageImports` 來自動完成這個任務。 Next.js 13.5 開始支援。



要使用這個功能，你可以在 `next.config.js` 設定

```js
module.exports = {
  experimental: {
    optimizePackageImports: ['my-lib']
  }
}
```

當你啟用這個設定，Next.js 會分析 `my-lib` 的進入點檔案（Entry File）判斷是否有 Barrel File。如果有責會動態分析檔案並自動映射所有的匯入，類似於 `modularizeImports` 的機制。

這個過程相較於 Tree-shakeing 省效能，因爲它只會掃描 Barrel File ，遞迴的處理其他內部引入的 Barrel File 然後全部匯出 `export * from`。



由於這個新的設定不依賴套件內部的實作，我們已經預先設定了一些[常見的函式庫](https://github.com/vercel/next.js/blob/12e888126ccf968193e7570a68db1bc35f90d52d/packages/next/src/server/config.ts#L710-L765) 如 `lucide-react` 和 `2headlissui/react`。

未來，官方將會探索自動化判斷是否需要分析某個套件的想法。目前，隨著社群和我們的團隊發現新的套件需要優化，該預設清單還在持續擴大。



## 測量改善效能

### 本地開發

本地的 benchmark 是在 M2 MacBook Air 運行，當使用其中一個主流的 Icon 函式庫的時候觀察到縮短 15% ~ 70% 根據套件有所不同。

- `@mui/material` 7.1s (2225 modules) -> 2.9s (735 modules) (-4.2s)
- `recharts`: 5.1s (1485 modules) -> 3.9s (1317 modules) (-1.2s)
- `@material-ui/core`: 6.3s (1304 modules) -> 4.4s (596 modules) (-1.9s)
- `react-use`: 5.3s (607 modules) -> 4.4s (337 modules) (-0.9s)
- `lucide-react`: 5.8s (1583 modules) -> 3s (333 modules) (-2.8s)
- `@material-ui/icons`: 10.2s (11738 modules) -> 2.9s (632 modules) (-7.3s)
- `@tabler/icons-react`: 4.5s (4998 modules) -> 3.9s (349 modules) (-0.6s)
- `rxjs`: 4.3s (770 modules) -> 3.3s (359 modules) (-1.0s)

這些節省的時間不僅適用於本地開發環境的初始啟動，也包含了熱模組替換（HMR）的速度，使得本地開發的即時反饋感覺更快。如果你使用了多個擁有許多子模組的函式庫，這些時間節省會迅速累積。



### 正式環境建置

在 M2 MacBook Air 建立一個 Next.js 使用 App Router 的頁面中，該頁面使用了 `lucide-react` 和 `@headlessui/react`，執行 `npm run build` 節省 ~28% 的時間，因為不用再處理模組的 Tree-shaking。



### 冷啟動加速

在本地環境，我們已經知道 Node.js server 渲染一個使用 `lucide-react` 和 `@headlessui/react` 的頁面大約可以提升 10% 的速度。

在 Serverless 的環境例如 Vercel，優化減少了部署程式碼的大小和 `require` 的調用，加速 13.5 的其他優化官方測量大約提升 40% 的冷啟動速度。



### 遞迴解析 Barrel File

最後一個改變就是處理遞迴的 Barrel File 的部分，它們被優化成一個單一模組。測試方面，該模組處理了一個具有 4 層、每層有10個 `export *` 的模組，總共相當於10,000個模組。

優化之前遞迴的套件大約需要 30 秒才能編譯完成。現在，它只需要大約 7 秒。我們看到有些擁有超過 100,000 模組的客戶，重載速度快了90%。
---
title: 'Rollup.js 入門'
categories: Program
tags: [javascript, bundle]
---

# 為什麼都有了 webpack 還使用 rollup

寫在前面的不是新聞...大約在 2017 4 月人們突然發現 Facebook 在 React 專案使用了 rollup.js。這個舉動的確使得很多人想問為什麼？

原本使用的 webpack 一直是 javascript 開發社群中最被推薦的封裝工具，webpack 成功的故事，每個月數以百萬的下載，無數的網站與應用程式使用了它，擁有廣大的社群和協同開發者，同時在財務上也得到許多人的贊助。

相較之下 rollup 就顯得渺小。但，React 並不是唯一轉向的人，Vue，Ember，Preact，D3，Three.js，Moment 等一些知名的函式庫或框架都使用了 rollup。所以我們真的想知道到底怎麼回事？為什麼我們不能就用一套大家都認同的模組封裝工具就好了呢？

# 關於兩者的淵源

webpack 是 2012 年由 Tobias Koppers 所開始的專案，目的是解決當時其他工具沒有解決的問題 - 建置複雜的單一頁面應用程式（SPA）。
主要的兩個核心功能：

1. Code-splitting 拆分程式碼片段讓整個應用程式可以被切成可管理的程式碼片段，然後依據需求載入。這意味的是使用者不必在每次都下載完整的程式，只須依需求下載需要的程式碼，得到更短的下載時間。你可能會說我們可以手動處理這個問題，不過有工具協助當然可以減少出錯的機會。

2. 靜態資源檔例如：圖片，CSS 可以像其他相依的函式庫一樣在程式中載入，我們省去了配置檔案，處理檔案路徑等瑣碎問題，甚至是使用 script 幫 URL 加上 hash。webpack 會幫助我們處理這些問題。

rollup 的出現完全是為了不同的動機 - 利用 ES2015 新的模組特性盡可能有效率的建置 易於分拆匯入／匯出的 javascript 函式庫。其他封裝工具包括 webpack 都是將模組包進 function 然後封裝成一個


## 概觀

Rollup 是個 Javascript 的模組封裝工具（module bundler），用來協助我們將分散的程式片段編譯整合成一個大型的函式庫或應用程式。它採用 ES6 新的模組規範，而不是既有的替代方案像是 CommonJS，AMD。ES6 模組標準讓我們可以從第三方函式庫中無縫的整合我們需要的 function 或局部功能。最終這個規範將會得到瀏覽器原生的支援，Rollup 的核心功能就是在瀏覽器支援之前，讓我們提前在現在就可以使用這個功能。

# 快速入門指南

透過 `npm i -g rollup` 我們即可安裝 `rollup`。rollup 具備兩種使用方式，一是使用[指令介面](https://github.com/rollup/rollup/wiki/Command-Line-Interface)搭配參數或設定檔。二是使用[Javascript API](https://github.com/rollup/rollup/wiki/JavaScript-API)。執行 `rollup --help` 我們可以查閱相關參數的用法。


# webpack 

* webpack `3.5.x` 預設只會處理 `import` 和 `export` 其他的新語法需要使用 Babel loader
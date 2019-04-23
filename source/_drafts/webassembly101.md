---
title: '[譯] WebAssembly 101 - 開發者的第一步'
categories: Program
tags: javascript
---

本文會透過將 Javascript 版本的 `康威生命遊戲` 轉換成使用 WebAssembly 的過程中指出一些重要的部分。這會是個簡單的練習比起 Hello World 也是較好的起點。

最近，我開始對 WebAssembly 感到興趣並且決定在週末認真的理解一下。WebAssembly 是一個新的標準，目的是使 Web 應用程式貼近原生程式的效能。基本上 [asm.js done right](https://kripken.github.io/talks/wasm.html#/) 這份簡報解釋了大部分關於 WebAssembly 的演進。不過伴隨著大量的開發任務 WebAssembly 仍處在探索發展的階段，而且許多資訊一下就過期了。

大略瀏覽了 [awesome-wasm list](https://github.com/mbasso/awesome-wasm/blob/master/README.md) 這份整理資訊的確是個好的起點。




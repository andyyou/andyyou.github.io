## 工具整合
一般來說，每一個專案使用不同的建置方式，系統環境，以及部署方式。React 試著盡可能的讓一切與系統環境沒有任何相依性。
意味著官方盡可能讓所有平台都能用一樣的方式運行 React。

## React

### React CDN
在[下載頁面](http://facebook.github.io/react/downloads.html)官方提供了 React 的 CDN。這些預先編譯的檔案使用的是[UMD 模組格式](https://github.com/umdjs/umd)。
將它們放置到 `<script>` 標簽中就可以注入 `React` 到您的全域環境中。在 `CommonJS` 與 `AMD` 的環境下只要引入就能馬上使用。

> UMD 提供兼容多種環境的特性。為了達到這個目的，一般狀況，但不是全部，UMD 模組格式會把程式碼包進一個立即調用的函式(Immediately Invoked Function Expression，IIFE)中。

### 使用 Git 中的 master
React 提供了一份說明，協助您在取得 GitHub(https://github.com/facebook/react) 後執行編譯建置。
同時也提供了 `CommonJS` 方式的模組在 `build/modules` 目錄底下，您可以將它們放到任何環境或者支援 CommonJS 的封裝工具。

## JSX

### 瀏覽器下的 JSX 編譯
如果您偏好使用 JSX ，您也可以在[官方下載頁面](http://facebook.github.io/react/downloads.html)找到這個可以在瀏覽器執行時期編譯 JSX 的 `js` 檔案 `JSTransformer`，以讓您可以在開發時方便的使用。
單純的使用 `<script type='text/jsx'>` 嵌入您的程式碼，搭配 JSX Transformer 即可。不過請注意要加入 `/** @jsx React.DOM` */ 在開始的地方，否則 JSX Transformer 將不會執行轉譯的工作。

> 注意:
  瀏覽器版的 JSX 編譯檔案相當大，且必須耗費多餘的與要呈現的結果無關運算，這應該要避免。千萬別在發佈的產品上使用。

## 發佈: 預先編譯 JSX
如果您已經有安裝 `npm` 那麼您可以很輕鬆的直接執行 `npm install -g react-tools` 來安裝 jsx 的指令工具。這個工具將會先幫你把 JSX 格式的檔案轉換為原始的 Javascript，然後您就可以直接使用。
當然您也可以使用自動編譯的方式，透過 `jsx --watch src/ build/`，如此一來當目錄有任何變更的時候，指令工具就會自動幫您編譯。
需要更多詳細的用法資訊請執行 `jsx --help`。

## 實用的開源專案
開源碼社群已經有許多整合 JSX 的工具，查閱 [JSX 整合工具](https://github.com/facebook/react/wiki/Complementary-Tools#jsx-integrations) 可以找到適合您的開源專案或工具。
  

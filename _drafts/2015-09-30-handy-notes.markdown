* css modules 的核心重點是將 css 匯入在 js 中, 類似 js import 模組那樣的行為
* 在大型專案開發 css 會遇到的問題
  - 全域的命名空間
  - 相依性 dependencies
  - Dead code elimination 意味著編譯器將那些沒作用的 code 移除 例如 if(false) { /* do tings */ }
  - 壓縮最佳化
  - 共用常數(constant) https://www.reddit.com/r/javascript/comments/3bo42p/sharing_constants_in_es6_modules/
  - 無法偵測解析度
  - isolation 意味著執行不需要依賴 context
* Christopher 指出把 css 搬移到 js 中處理，這些問題都可以被解決，但同時也產生了一些問題, 複雜性和 css 特質，舉例來說處理 `:hover` 狀態就是問題之一
  那些本來在 css 被處理掉的問題會再度呈現在我們面前
* css modules 認為我們應該要保留那些我們喜歡的 css 特性
* 在 css modules 中每一個檔案各自分開編譯, 所以第一你不需要再煩惱那些通用的命名已經被用過的問題, 因為它不會污染全域的命名

在使用 css module 之前我們可能使用 BEM 來命名

{% highlight css %}
.Button {/* all styles for Normal */}
.Button--disabled { /* overrides for Disabled */ }
.Button--error { /* overrides for Error */ }
.Button--in-progress { /* overides for In progress */ }
{% endhighlight %}

{% highlight html %}
<button class="Button Button--in-progress">Processing...</button>
{% endhighlight %}


換成 css modules 之後我們不再需要 prefix 來區分, 名稱可以是那些很常用的命名，但是注意我們是靠檔案來區分的
這些 css 都放在 submit-button.css

{% highlight css %}
/* components/submit-button.css */
.normal { /* all styles for Normal */ }
.disabled { /* all styles for Disabled */ }
.error { /* all styles for Error */ }
.inProgress { /* all styles for In Progress */
{% endhighlight %}

{% highlight js %}
import styles from './submit-button.css';

buttonElement.outerHTML = `<button class=${styles.normal}>Submit</button>`
{% endhighlight %}

這些名稱會自動加上一些編碼以保證他是唯一的名稱，匯入之後 css module 會幫你處理剩下的事情 css 會被編譯成一種叫做 ICSS 的格式
這也是 CSS 和 JS 溝通的方式


* 在 css module 每一個 class 應該要具備所有需要的樣式, 而不是靠覆寫繼承，這麼做是因為我們在撰寫 js 的時候就不用一直反覆套用不同的名稱

{% highlight js %}
/* Don't do this */
`class=${[styles.normal, styles['in-progress']].join(" ")}`

/* Using a single name makes a big difference */
`class=${styles['in-progress']}`

/* camelCase makes it even better */
`class=${styles.inProgress}`
{% endhighlight %}

* 在 css 語法中使用 composes 來實作類似 scss 的 extend 功能
* 在 Sass 或 LESS 任何 @import 的檔案會存在全域的命名空間
* `composes` 還可以指定來源檔案

{% highlight css %}
/* submit-button.css */
.common { /* font-sizes, padding, border-radius */ }
.normal {
  composes: common;
  composes: primary from "../shared/colors.css";
}
{% endhighlight %}



///=========步驟

1. npm install webpack -g
2 .npm install webpack[@1.1.1] --save-dev  # @1.2.0 specify version
3. webpack entry.js bundle.js               # excute compile 預設會在執行指令的目錄
4. webpack ./entry.js ./bundle.js
5. webpack require 解析路徑分成三種類型：絕對路徑, 相對路徑, 模組名稱

解析路徑的流程: 
* 絕對路徑: 首先檢查該路徑是否為目錄, 如果是目錄接著就要確認 main file 誰是主要進入點, 也因此會去查 package.json 的 main 欄位。如果都沒有 package.json 或者是沒有 main 欄位設定則使用 index 當作預設檔名, 得到檔名後接著會嘗試用設定在 `resolve.extensions` 的副檔名加在尾巴去搜尋檔案

* 相對路徑: 相對路徑是根據使用 require 的那個檔案的目錄開始尋找, 如果沒有任何資源檔使用 require 則設定檔或產生設定的檔案所在的那個目錄會被拿來當作搜尋的路徑基準, 這個目錄稱為 context directory, context directory 路徑 + 相對路徑 = 最後尋找的路徑
其他的規則則遵循絕對路徑尋找檔案的流程。

* 模組名稱: 為了要根據模組名稱來載入, 首先需要收集所有存放模組的目錄, 這個流程類似為 node.js 載入模組的流程,  不過搜尋的目錄是透過 `resolve.modulesDirectories` 來設定, 並且這些目錄也會被加在 resolve.root 和 resolve.fallback。接著按照名稱去尋找模組的目錄名稱, 後續的流程則跟解析絕對路徑一樣, 如果第一個被找到的目錄解析失敗則找試圖找尋第二個目錄.

只要路徑不是 require('/module') 或 require('./module') 使用 `/`, `./` 就是使用模組名稱 webpack 採用 node.js 解析模組的流程
所以舉例來說 `require('bar.js')` 就會去下面目錄尋找檔案
  
    /home/ry/projects/node_modules/bar.js
    /home/ry/node_modules/bar.js
    /home/node_modules/bar.js
    /node_modules/bar.js


6. 使用 require 來載入模組
7. 透過 loader 來編譯其他類型的檔案
8. 透過 css-loader 處理 css 檔案, style-loader 套用樣式到 DOM
  
  - 起手式 npm install webpack webpack-dev-server react-hot-loader babel-loader css-loader style-loader sass-loader url-loader --save-dev

9. loader 的使用方式分成
  - 直接用 webpack.config.js 針對檔案設定使用的 laoder

  - 指令繫結 webpack entry.js bundle.js --module-bind 'css=style!css'
    + 有一點要注意的是因為 ! 在 bash 裡面有特殊意義所以當您想用 " 替代 ' 請記得跳脫 webpack ./entry.js bundle.js --module-bind "css=style\!css"

  - require() 裡面使用 chain 的語法 require('style!css!./style.css'); // ! + loader prefix
    + 這種方式路徑是可以用相對路徑, 不一定要用 loader 的模組名稱 require("./loaders/m-loader!./m-awesome-module");
    + loader 可以用 ? 加入參數

10. webpack --progress --colors --watch # 常用的編譯

11. 單純使用 webpack 監視模式，bundle.js 檔案是會被產生的，但如果是 webpack-dev-server 則不會產生 bundle.js 那隻檔案

12. 
    webpack 建置編譯開發版的檔案，只會運行一次
    webpack -p 執行一次建置的任務，產生正式版(具有壓縮)
    webpack --watch 持續性編譯，即開發時期，每次一變更檔案就重新編譯(快速)
    webpack -d 包含產出 source maps，即 .js.map 檔案

13. 使用 webpack-dev-server
  
    webpack-dev-server 會在 localhost:8080 建立起專案的 server
    --devtool eval 會顯示出發生錯誤的行數與檔案名稱
    --progress 會顯示出打包的過程
    --colors 會幫 webpack 顯示的訊息加入顏色
    --content-based build 指向專案最終輸出的資料夾

14. 每次在 require/import 的時候都要輸入附檔名也是挺麻煩的，所以 webpack 也提供您 require 不加副檔名的機制，為了開啟這個功能，我們必須要加入 resolve.extensions
參數告訴 webpack 該處理哪些副檔名。


```
resolve: {
  // 現在您可以把那些 require 中的副檔名去掉了
  extensions: ['', '.js', '.json', '.coffee'] 
}
```

webpack-dev-server 是 express 使用 webpack-dev-middleware 取得 webpack 封裝的結果

webpack-dev-server 分成 inline mode / hot mode
   1. html 加上 http://localhost:8080/webpack-dev-server.js
   2. webpack.config.js
    - entry 加上 webpack/hot/dev-server | webpack/hot/only-dev-server
    - js module loader 加上 react-hot, 還有 include: path.join(__dirname, 'app') 
   3. webpack-dev-server --devtool eval --progress --colors --hot --content-base build # 啟用 server 指令
   4. 如果瀏覽 http://localhost:8080/web-dev-server 則不用加上第一步的 js

  - 啟用 hot mode 要加上參數 webpack-dev-server --hot

--content-base 參數用來設定 webpack-dev-server 監視和載入 server root 的目錄
output.publicPath 等於是設定 url path, 如果是設定 /foo -> http://localhost:8080/foo/, 如果有設定網址則會用該網址路徑
publicPath 和 --content-base 可以達到一樣的效果, 不過路徑不同 

browserify 和 webpack 都是處理模組載入的封裝工具 白話文即處理那些用 require 的檔案 > bundle.js
Grunt, gulp 和 webpack 這類工具處理問題的角度不同，一個是讓你組織各種任務，一個是針對檔案類型進行載入解析 > 編譯，壓縮 > 封裝，


webpack 在實務上就是一個打包的指令 針對那些有用 require 的檔案 流程上會需要一個 entrypoint 進入點的檔案 webpack entry.js bundle.js

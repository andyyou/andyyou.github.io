---
layout: post
title: 'grunt-init 使用教學與 grunt-init-simple-server'
date: 2014-01-26 12:13:00
categories: Javascript F2E
---

在開始如何製作自己的 `grunt-init` 樣板的教學之前，先分享[grunt-init-simple-server](https://github.com/AndyYou/grunt-init-simple-server)。由於小弟玩性太重三天兩頭就喜歡換 framework 所以才寫了這個簡單的 grunt task。可以方便在學習新的前端技術的時候快速地幫專案加上一個簡易地 server。

`Grunt-init` 是一個用來自動建立專案架構的工具，它協助我們根據目前的環境以及建立過程中
詢問一些設定項目來建立整個專案的架構。就像我們使用 IDE 如 Visual Studio 建立專案
一樣，透過詢問你一些選項後建立該專案。

### 安裝

{% highlight bash %}
$ npm install -g grunt-init
{% endhighlight %}

### 使用方式

* `grunt-init --help` 列出已安裝可以使用的樣板。
* `grunt-init [TEMPLATE]` 建立指定類型的專案。
* `grunt-init /path/to/TEMPLATE` 建立指定路徑樣板的專案。

上面所謂的樣板（template）其實就是『專案類型』，更白話文的說就是指定的『目錄架構』
注意：大部份的樣板都是產生檔案或目錄到目前所在的目錄，所以在使用的時候請確認這是一個新目錄
或者目錄裡的檔案不會被複寫。


### 安裝樣板（專案類型）

一般來說樣板會安裝在你的 `~/.grunt-init` 目錄裡，在 Window 中是 `%USERPROFILE%\.grunt-init\`
一旦安裝完成，接著你會透過 `grunt-init [TEMPLATE]` 來建立專案。
這邊舉個例子安裝 jquery 專案類型的話就是執行

{% highlight bash %}
$ git clone https://github.com/gruntjs/grunt-init-jquery.git ~/.grunt-init/jquery
{% endhighlight %}

通常我們會透過 git 來安裝。整體來說 template 只不過是事先將一些樣板產生的 script 放置在
`~/grunt-init` 目錄底下，透過 git 來安裝的話，之後可以輕鬆的更新升級。

注意：如果你想要將樣板安裝到本地，並且可以透過 `grunt-init --help` 來列出已安裝的樣板的話。
例如：你想指定專案類型為 `foobarbaz` ，那你只要把樣板目錄安裝到指定目錄 `~/grunt-init/foobarbaz` 即可。

下面是一些官方維護的樣板專案，您可以試著安裝並使用看看：

* [grunt-init-commonjs](https://github.com/gruntjs/grunt-init-commonjs) 建立 CommonJS 模組，包含 Nodeunit unit test。
* [grunt-init-gruntfile](https://github.com/gruntjs/grunt-init-gruntfile) 單純透過詢問選項建立 Gruntfile，最單純也最實用的。
* [grunt-init-gruntplugin](https://github.com/gruntjs/grunt-init-gruntplugin) 建立一個 Grunt 套件。
* [grunt-init-jquery](https://github.com/gruntjs/grunt-init-jquery) 建立 jQuery 套件專案，包含 QUnit unit test。
* [grunt-init-node](https://github.com/gruntjs/grunt-init-node)

大部份的狀況下 `grunt-init` 幫我們處理掉許多初始專案的東西，如果沒有你可能要使用
`npm init`，`bower init`，`建立 Gruntfile.js` ，當然如果你是使用 framework 如：Sails
那它可能都幫你處理完這些項目。

### 自定樣板
您當然可以自己建立客製化的樣板，但是你的樣板必須遵循上述一樣的結構。
舉例來說一個基本的樣板叫 `my-template` 就必須遵循下面的檔案結構：

* `my-template/template.js` 主要的樣板檔案，所有的選項提問，處理複製哪些檔案都在這隻程式處理。
* `my-template/rename.js` 更改一些檔案的名稱設定，所有的命名規則在這邊設定。
* `my-template/root/` 所有要被複製檔案都存放在這，就是在產生專案時大部份預設檔案都是從這邊來的。

假設上面說的這些檔案和目錄都已經在 `/path/to/my-template` 那你就可以使用 `grunt-init /path/to/my-template` 來建立專案了。
此外，只要你這個目錄放在 `~/.grunt-init` 目錄底下，那你可以直接使用 `grunt-init my-template` 來建立。

#### 複製檔案 Copying files
只要樣板使用 `init.filesToCopy` 和 `init.copyAndProcess` 方法，任何在 `root/` 目錄底下的
檔案都會被複製到目前的目錄。

注意：所有被複製的檔案都會被當作模板來處理，裏面有 {{ "{%= " }}%} 符號的變數都會被處理。
例如： `grunt-init-jquery` 專案裡 `/root/src/name.js` 檔案中的第五行

{% highlight html %}
{% raw %}
 Copyright (c) {%= grunt.template.today('yyyy') %} {%= author_name %}
{% endraw %}
{% endhighlight %}

除非你使用了 `noProcess` 設定，不然所有檔案都會經過上面的處理。

#### 修改檔案名稱或排除檔案
關於 `rename.json` 是用來描述來源檔案路徑和目的檔案路徑變更檔名的對應規則。
來源檔案名稱必須要對應于 root 目錄。你可以參考 `[grunt-init-jqueryplugin](https://github.com/gruntjs/grunt-init-gruntplugin/blob/master/rename.json)` 的 rename.json。

#### 設定預設詢問項目答案
每一個 prompt 詢問你的問題都會有一個預設值，它通常是設死的或者根據環境設定取得的值。
如果你要設定這些預設值你可以在 `~/.grunt-init/defaults.json` 裡面去設定。
下面有一個簡單的範例：

{% highlight json %}
{
  "author_name": "\"Cowboy\" Andy You",
  "author_email": "none",
  "author_url": "http://passer.cc/"
}
{% endhighlight %}

### 定義初始化的資訊

#### exports.description 
這是一個簡短的樣板描述，當使用者執行 `grunt-init` 會顯示在列表。

{% highlight js %}
exports.description = descriptionString;
{% endhighlight %}

注意：任何一個樣板有錯誤的時候會導致執行 `grunt-init` 產生錯誤訊息。

#### exports.note
如果啟用此設定，當開始執行項目詢問時，將會增加額外的訊息描速。這裡是你可以給使用者一些
有用的訊息的地方，例如：哪些欄位是必要的選項。

{% highlight js %}
exports.notes = notesString;
{% endhighlight %}

#### exports.warnOn
警告功能，利用萬用字元表示式或者陣列去比對目錄中已存在的檔案，一旦符合規則發現檔案存在就會中斷專案的建立。
不過一旦使用者使用 `--force` 參數，將會繼續執行，這可能會覆蓋掉以存在的檔案。
這個功能非常實用，可以協助避免覆蓋掉一些已存在的檔案。

{% highlight js %}
exports.warnOn = wildcarPattern;
{% endhighlight %}

最常見的方式是使用 `*` 去匹配大部分名稱的檔案。舉例來說 `*.js` 可以匹配目錄底下所有的 js 檔案。
下面是一些常用的範例：

{% highlight js %}
exports.warnOn = 'Gruntfile.js';    // Warn on a Gruntfile.js file.
exports.warnOn = '*.js';            // Warn on any .js file.
exports.warnOn = '*';               // Warn on any non-dotfile or non-dotdir.
exports.warnOn = '.*';              // Warn on any dotfile or dotdir.
exports.warnOn = '{.*,*}';          // Warn on any file or dir (dot or non-dot).
exports.warnOn = '!*/**';           // Warn on any file (ignoring dirs).
exports.warnOn = '*.{png,gif,jpg}'; // Warn on any image file.

// This is another way of writing the last example.
exports.warnOn = ['*.png', '*.gif', '*.jpg'];
{% endhighlight %}

#### exports.template
exports 最重要的便是爲 `template` 這個屬性定義一個 `function` ，所有實際執行的初始化動作都寫在
這個 function 裡面 。有三個你可以使用的參數分別是 `grunt`，`init`，`done`。`grunt` 參數參考至 `grunt`。
包含了所有 grunt 你能使用的函式庫，例如：`grunt.file.read()`。 `init` 參數是一個物件，主要包含關於 template 相關的屬性和方法。
`done` 是一個 function 當樣板建立的步驟執行完成的時候便呼叫他。

### 初始化樣板內部
接下來這些 `init.method` 都是在 `exports.template = function (grunt, init, done)` 函式中執行的使用的。
如果有些 method 文件寫得不清楚的話請直接參考[原始碼](https://github.com/gruntjs/grunt-init/blob/master/tasks/init.js)
#### init.addLicenseFiles
自動產生 License 檔案。如果你的專案不是 MIT 其他授權可以[參考這邊](https://spdx.org/licenses/)的縮寫。

{% highlight js %}
var files = {};
var licenses = ['MIT'];
init.addLicenseFiles(files, licenses);
// files === {'LICENSE-MIT': 'licenses/LICENSE-MIT'}
{% endhighlight %}

#### init.availableLicenses
真正可用的 Licenses 可以使用這個 method 列表。

{% highlight js %}
var licenses = init.availableLicenses();
// licenses === [ 'Apache-2.0', 'GPL-2.0', 'MIT', 'MPL-2.0' ]
{% endhighlight %}

#### init.copy
給定一個來源路徑（絕對/相對）和可選的相對目標路徑，就可以複製到目前的目錄下。
至於 options 參數的用法直接參考 `grunt.file.copy()`。

{% highlight js %}
var options = {
  // If an encoding is not specified, default to grunt.file.defaultEncoding.
  // If null, the `process` function will receive a Buffer instead of String.
  encoding: encodingName,
  // The source file contents and file path are passed into this function,
  // whose return value will be used as the destination file's contents. If
  // this function returns `false`, the file copy will be aborted.
  process: processFunction,
  // These optional globbing patterns will be matched against the filepath
  // (not the filename) using grunt.file.isMatch. If any specified globbing
  // pattern matches, the file won't be processed via the `process` function.
  // If `true` is specified, processing will be prevented.
  noProcess: globbingPatterns
};
{% endhighlight %}

#### init.copyAndProcess
遍歷 files 物件然後一個一個傳遞處理，作用就是複製來源檔案到目的，並且處理內容。

{% highlight js %}
init.copyAndProcess(files, props[, options])
{% endhighlight %}

#### init.defaults
使用者在 defaults.json 中設定的預設值。

#### init.destpath
當前目錄的絕對路徑，也就是所謂目的地目錄的路徑。

#### init.expand
用法和 `grunt.file.expand` 相同。
傳回一個唯一的陣列，包含所有檔案，目錄的路徑，這些路徑會根據之前文章提到 grunt 處理檔案的匹配方式一樣。
這個方法允許你使用 `,` 分隔萬用字元匹配表示式，或者使用一個陣列來存放這些匹配模式的表示式。
另外如果使用 `!` 會排除匹配成功的路徑。最後這些匹配模式是會照順序處理的，如果先被排除了後面就不會再被匹配到。

#### init.filesToCopy
傳回一個包含欲複製檔案的絕對路徑和相對路徑的物件，並依照 rename.json 的規則重新命名。

{% highlight js %}
var files = init.filesToCopy(props);
/* files === { '.gitignore': 'template/root/.gitignore,
   'jshintrc': 'template/root/.jshintrc',
   'Gruntfile.js': 'template/root/Gruntfile.js',
   'README.md': 'template/root/README.md',
   'test/test_test.js': 'template/root/test/name_test.js'
   } */    
{% endhighlight %}

#### init.getFile
取得一個任務檔案路徑。[原始碼](https://github.com/gruntjs/grunt-init/blob/master/tasks/lib/helpers.js)

#### init.getTemplates
傳回一個包含現有可用的樣板資訊的物件。

#### init.initSearchDirs
在初始化目錄中搜尋初始化樣板， `template` 指的是本機的樣板，通常包含 `~/.grunt-init` 目錄和 `grunt-init` 中的核心初始化的任務。

####  init.process
啟動初始化並開始輸入提示選項。

{% highlight js %}
init.process(options, prompts, done);

init.process({}, [
    //Prompt for these values
    init.prompt('name'),
    init.prompt('description'),
    init.prompt('version')
], function(err, props){
    // All finished, do something with the properties
});
{% endhighlight %}

#### init.prompt
顯示一道問答項目選擇。

{% highlight js %}
init.prompt(name[, default])
{% endhighlight %}

#### init.prompts
傳回一個包含所有 `prompt` 提示問答值的物件。

#### init.readDefaults
從任務檔案中讀取預設 JSON ，整合他們至 data object。

#### init.renames
rename.json 的物件。

#### init.searchDirs
搜尋樣板路徑目錄的陣列。

#### init.srcpath
根據檔案名稱搜尋初始化模板並回傳一個絕對路徑。

#### init.userDir
傳回用戶樣板目錄的絕對路徑

{% highlight js %}
var dir = init.userDir();
// dir === '/Users/shama/.grunt-init'
{% endhighlight %}

#### init.writePackageJSON
在目的地目錄寫入一個 `package.json`。 callback 函式可以用於後來在處理屬性新增/刪除/其他操作。

### 內建提示

#### author_email
`package.json` 中作者的 Email，預設情況下會嘗試從用戶的 git config 中找到預設值。

#### author_name
`package.json` 作者全名。

#### authro_url
作者相關的網址

#### bin
專案根目錄中 cli script 的相對路徑。

#### bugs
追蹤專案 bug 的公開 URL

#### description
專案描速

#### grunt_version
項目有效的 grunt 版本

#### homepage
專案的首頁網址

#### jquery_version
如果是 jQuery 專案的話，表示所需要的 jquery 版本。

#### licenses
授權協議，內建的有 `MIT`，`MPL-2.0`，`GPL-2.0`，`Apache-2.0`。

#### main
專案的 entry point

#### name
專案名稱

#### node_version
專案所需的 node 版本

#### npm_test
專案執行測試的命令。

#### repository
專案的 git

#### title
專案名稱

#### version
版本



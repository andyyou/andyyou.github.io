---
layout: post
title: 'React 快速概覽'
date: 2014-02-05 11:17:00
categories: Program
tags: [reactjs]
---

# 入門
開始學習了解 React 的方式就是使用 `JSFiddle` 來觀察實作的範例： Hello Worlds。

* [React JSFiddle](http://jsfiddle.net/vjeux/kb3gN/)
* [React JSFiddle without JSX](http://jsfiddle.net/vjeux/VkebS/)

![](http://i.imgur.com/Q4kFTzL.png)

<!--more-->

# 下載入門套件
下載入門範例與套件。
* [v0.8.0](http://facebook.github.io/react/downloads/react-0.8.0.zip)

解開壓縮檔後，在根目錄建立一個 `helloworld.html` 然後輸入下面的例子。

~~~html
<!DOCTYPE html>
<html>
  <head>
    <script src="build/react.js"></script>
    <script src="build/JSXTransformer.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/jsx">
      /** @jsx React.DOM */
      React.renderComponent(
        <h1>Hello, world!</h1>,
        document.getElementById('example')
      );
    </script>
  </body>
</html>
~~~

在 Javascript 裡面使用 XML 格式的語法叫做 `JSX`，這種語法是官方推薦的寫法。可以[參考](http://facebook.github.io/react/docs/jsx-in-depth.html)學習更多關於 JSX 的用法。為了使 JSX 可以正確的轉換為 Javascript 我們會使用 `<script type='text/jsx'>` 標簽，以及記得要載入 `JSXTransformer.js` 以確保程式正確執行。在這邊我們會先提醒那些有程式開發經驗的學習者。
不要忘記在一開始加入 `/** @jsx React.DOM */`，這不是一般註解，它是 React 用來定義要處理的 JSX 。如果你沒有加入這個片段，你的程式碼將不會被轉換。

# 獨立的檔案
你的 React JSX 檔案可以是分開的獨立檔案。接著讓我們建立 `src/helloworld.js`

~~~js
/** @jsx React.DOM */
React.renderComponent(
  <h1>Hello, world!</h1>,
  document.getElementById('example')
);
~~~

然後在 `helloworld.html` 加入

~~~html
<script type="text/jsx" src="src/helloworld.js"></script>
~~~

即使是分開檔案功能仍然可以運行，這在大型專案中將有助于 DRY 原則，同樣功能的元件應該抽離獨立。

# 離線轉換 JSX
安裝這個指令轉換工具（command-line）需要先安裝 `npm`。

~~~bash
$ npm install -g react-tools
~~~

然後轉換 `src/helloworld.js` 檔案為原生 Javascript。

~~~bash
$ jsx --watch src/ build/
~~~

看看自動產生的 `build/helloworld.js` 如下，因為使用了 `--watch` 參數，你可以直接修改 JSX ，然後工具就會自動更新。指令的語法是 `jsx --watch <source directory> <output directory>` ，所以請不要指定到檔案。

~~~js
/** @jsx React.DOM */
React.renderComponent(
  React.DOM.h1(null, 'Hello, world!'),
  document.getElementById('example')
);
~~~

> 注意
> 註解的解析器是非常嚴格的;為了能夠提取 `@jsx` 修飾子，兩件事情必須遵守：
> 1. `@jsx` 註解區塊必須要在檔案或程式碼的開頭，也必須是第一段註解。
> 2. 註解的開頭必須是 `/**` （`/*` 和 `//` 將會不正常）。
> 如果解析器找不到 `@jsx註解區塊` 輸出時就不會執行轉換。

讓我們接著更新 HTML 如下：

~~~html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello React!</title>
    <script src="build/react.js"></script>
    <!-- No need for JSXTransformer! -->
  </head>
  <body>
    <div id="example"></div>
    <script src="build/helloworld.js"></script>
  </body>
</html>
~~~

> 注意： `type='text/jsx'` 要拿掉，否則無法運作。

# 使用 CommonJS
如果你想要使用模組化的 React ， 請 [fork 官方專案](http://github.com/facebook/react)，然後執行 `npm install` 和 `grunt`，便可以產出遵循 CommonJS 規則模組化的程式碼。官方提供的 jsx 編譯工具可以整合可以簡單地整合到大部份的封裝系統。
[CommonJS 補充](http://www.grati.org/?p=165)

# 下一步
查閱[官方教學](http://facebook.github.io/react/docs/tutorial.html)和其他`/examples`範例目錄學習更多。

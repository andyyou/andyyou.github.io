## 手把手教您學會 Flux
關於 Facebook 前一陣子發表了一個新的方式組織應用程式，一個新的程式架構他們命名為 Flux，根據他們的說明: 隨著現代的應用程式成長越趨複雜，這個模式減少了開發時要維修的機會或維護的事項，同時降低認知負載(http://entangle-jack.blogspot.tw/2006/09/cognitive-load-theory.html)
不過由於官方的文件不夠親民，所以這邊我將記錄目前為止學習的認知並且總結一些關於 Flux 的東西。(本篇文章將會略過 React 細節的部分)
我們將會從頭建立一個 Flux 的應用程式

我們將會根據以下步驟完成這麼範例
1. 設定環境包含相依性函式庫與 Dispatcher
2. 建立 App 和主要構成元件
3. 模擬從 REST API 接收資料
4. 加入 Store 並使用 Dispatcher 註冊
5. 使用 React 元件顯示測試資料
6. 撰寫第一個 View 行為(action) ，透過點擊來選擇節點

對於那些很心急的人我們先在這裡展示完成的[DEMO](http://durdn.bitbucket.org/flux-demo-1/)

## Flux 維持一個單向的資料流
什麼是 Flux? Flux 的核心概念其實就是一個定向的循環，當事件或者處理資料時總是遵循一個單向的資料流。這使得每一件事情都可以簡單的被管理即便您將處理的機制非常負責，並且除去許多元件之間模糊的關係。

![](http://atlassian.wpengine.netdna-cdn.com/wp-content/uploads/flux-chart.png)

## 從頭開始
這個範例目的是讓我們熟悉 Model 的運作，所以我們從一個大綱編輯器著手。

## Step 1: 設定環境包含相依性函式庫與 Dispatcher
顯然的首先我們需要 Node 和 npm 環境所以在開始前您必須先安裝這些東西。
所有必須要安裝的項目我們列在 `package.json`:

```js
{
  "name": "out-of-line",
  "version": "0.0.1",
  "description": "Outline Editor that uses Reactjs and the Flux architecture",
  "main": "js/app.js",
  "dependencies": {
    "react": "~0.11"
  },
  "devDependencies": {
    "browserify": "~4.2.2",
    "envify": "~2.0.1",
    "jest-cli": "~0.1.17",
    "reactify": "~0.14.0",
    "statics": "~0.1.0",
    "uglify-js": "~2.4.15",
    "watchify": "~0.10.2"
  },
  "scripts": {
    "start": "STATIC_ROOT=./static watchify -o js/bundle.js -v -d .",
    "build": "STATIC_ROOT=./static NODE_ENV=production browserify . | uglifyjs -cm > js/bundle.min.js",
    "collect-static": "collect-static . ./static",
    "test": "jest"
  },
  "author": "Nicola Paolucci",
  "license": "Apache 2",
  "browserify": {
    "transform": [
      "reactify",
      "envify"
    ]
  },
  "jest": {
    "rootDir": "./js"
  }
}
```

> 這一步您可以先透過 `npm init` 建立 `package.json` 在編輯檔案。

關於 `devDependencies` 的設定是讓我們可以簡單的觀察我們的工作目錄(透過 npm start)，建置應用程式，執行 jest 來跑測試。這次我們只是照著官方 Flux 的範例包，所以也跟官方一樣使用 browserify。
接著我們透過 `npm install` 來安裝這些相依的套件，接著當我們編輯任何檔案時就能透過 npm start 來觀察專案的建置。

在我們努力撰寫程式碼之前我們需要一個最單純的結構，一個 `index.html` 頁面，載入壓縮打包過的 bundle.js 這東西是靠 browserify 建立的，此時我們甚至先不管任何 CSS。

這個 `index.html` 異常的簡單:
```js
<!doctype html>
<html lang='zh-tw'>
  <head>
    <meta charset='utf-8'>
    <title>Out of Line</title>
  </head>
  <body>
    <section id='react'></section>
    <script src='js/bundle.js'></script>
  </body>
</html>
```

### 加入 Flux 的 Dispatcher
在開始之前我們需要先介紹關於 Flux 的基本目錄架構如下:

```
myapp
  |
  + ...
  + js
    |
    + actions
    + components // all React components, both views and controller-views
    + constants
    + dispatcher
    + stores
    + app.js
    + bundle.js
  + index.html
  + ...
```
首先是我們把 `Dispatcher.js` 和 `invariant.js` 加入我們的專案(您可以在[這裡](https://github.com/facebook/flux/tree/master/src)找到這些檔案)
又或者直接安裝 flux 使用 `var Dispatcher = require('flux').Dispatcher`
接著我們繼承通用的 Dispatcher 來自訂我們自己的行為(action)分類，在 dispatcher 目錄下建立 `AppDispatcher.js`:
```js
var Dispatcher = require('./Dispatcher');

var copyProperties = require('react/lib/copyProperties');

var AppDispatcher = copyProperties(new Dispatcher(), {
  handleServerAction: function (action) {
    var payload = {
      source: 'SERVER_ACTION',
      action: action
    };
    this.dispatch(payload);
  },

  handleViewAction: function (action) {
    var payload = {
      source: 'VIEW_ACTION',
      action: action
    };
    this.dispatch(payload);
  }
});

module.exports = AppDispatcher;
```
Dispatcher 將會關聯所有的操作: View 會觸發 `action` 就是透過 Emitter 的 `emit` - Dispatcher 將會重新分配行為(action)給那些有訂閱的 Stores。
反過來說 `Stores` 從 View 獲取數據後觸發變更事件(Change Evnet)- 因此他們可以自己更新。

## Step 2: 建立 App 和主要構成元件
現在我們來建立我們第一個 React 元件。在 `js/components` 目錄建立 `App.react.js`(這邊一樣我們遵循官方的命名規則):
```js
/**
 * @jsx React.DOM
 */
var React = require('react');

var App = React.createClass({
  render: function () {
    return (
      <div className='outlineapp'> I am statisifed
      </div>
    );
  }
});

module.exports = App;
```
接著我們就可以建立程式的進入點了
```js
/**
 * @jsx React.DOM
 */
var App = require('./components/App.react');
var React = require('react');

React.renderComponent(
  <App />,
  document.getElementById('react')
);
```
執行 `npm start` 後可以觀察一下目前元件的運作。

## Step 3: 模擬從 REST API 接收資料
每當使用者在 UI 上做了變更，UI 就會透過一個 action (放置在 js/actions) 傳達給 Dispatcher 。
同樣也適用於伺服器端的事件，任何從 REST API 取得的資料也可以觸發 action 然後傳給 Dispatcher。
在我們可以呈現大綱之前我們需要先建立一些簡單的資料，然後當我們從伺服器取得資料時發動一個 action (模擬我們有一個 REST API 可以調用後取得資料)。
為了保持這個範例單純一點，我們先把資料放在 `Localstorage`:

`js/actions/OutlineServerActionCreators.js`

```js js/actions/OutlineServerActionCreators.js
var AppDispatcher = require('../dispatcher/AppDispatcher');

module.exports = {
  receiveAll: function (rawNodes) {
    AppDispatcher.handleServerAction({
      type: 'RECEIVE_RAW_NODES',
      rawNodes: rawNodes
    });
  }
};
```
然後我們模擬一個 REST API `js/utils/OutlineWebAPIUtils.js`
```js
var OutlineServerActionCreators = require('../actions/OutlineServerActionCreators');

module.exports = {
  getAllNodes: function () {
    var rawNodes = JSON.parse(localStorage.getItem('nodes'));

    OutlineServerActionCreators.receiveAll(rawNodes);
  }
}
```
修改 `js/app.js` 讓我們建立一個暫時的資料，假裝我們可以立刻收到存 REST API 來的資料:
```js
var App = require('./components/App.react');
var OutlineStartingData = require('./OutlineStartingData');
var OutlineWebAPIUtils = require('./utils/OutlineWebAPIUtils');
var React = require('react');

OutlineStartingData.init();
OutlineWebAPIUtils.getAllNodes();
React.renderComponent(
  〈App /〉,
  document.getElementById('react')
);
```

我們透過 `js/OutlineStartingData.js` 快速地在 localStorage 建立一些假資料:

```js
module.exports = {

  init: function() {
    localStorage.clear();
    localStorage.setItem('nodes', JSON.stringify({
      "key": 100, "meta": {},
      "content": "Home",
      "children": [
        {
          "key": 101, "meta": {},
          "content": "Animals",
          "type": "base",
          "children": [
            {
          "key": 102, "meta": {},
              "content": "Monkeys",
              "type": "base"
            },
            {
          "key": 103, "meta": {},
              "content": "Marmalutes",
              "type": "base"
            },
            {
          "key": 104, "meta": {},
              "content": "Marsupials",
              "type": "base"
            },
            {
          "key": 105, "meta": {},
              "content": "Meercats",
              "type": "base"
            },
            {
          "key": 106, "meta": {},
              "content": "Moose",
              "type": "base"
            },
            {
          "key": 107, "meta": {},
              "content": "Mice",
              "type": "base"
            }
          ],
          "collapsed": false
        },
        {
          "key": 108, "meta": {},
          "content": "Vegetables",
          "type": "base",
          "children": [
            {
          "key": 109, "meta": {},
              "content": "Trees",
              "type": "base"
            },
            {
          "key": 110, "meta": {},
              "content": "Ferns",
              "type": "base"
            },
            {
          "key": 111, "meta": {},
              "content": "Flowers",
              "type": "base"
            },
            {
          "key": 112, "meta": {},
              "content": "Grass",
              "type": "base"
            },
            {
          "key": 113, "meta": {},
              "content": "Water Lilies",
              "type": "base"
            },
            {
          "key": 114, "meta": {},
              "content": "Plums",
              "type": "base"
            },
            {
          "key": 115, "meta": {},
              "content": "Canteloup",
              "type": "base"
            },
            {
          "key": 116, "meta": {},
              "content": "Cabbage",
              "type": "base"
            },
            {
          "key": 117, "meta": {},
              "content": "Capers",
              "type": "base"
            },
            {
          "key": 118, "meta": {},
              "content": "Carrots",
              "type": "base"
            },
            {
          "key": 119, "meta": {},
              "content": "Camomile",
              "type": "base"
            }
          ],
          "collapsed": false
        },
        {
          "key": 120, "meta": {},
          "content": "Minerals",
          "type": "base",
          "children": [
            {
          "key": 121, "meta": {},
              "content": "Granite",
              "type": "base"
            },
            {
          "key": 122, "meta": {},
              "content": "Pummus",
              "type": "base"
            }
          ],
          "collapsed": false
        },
        {
          "key": 123, "meta": {},
          "content": "Planets",
          "type": "base",
          "children": [
            {
          "key": 124, "meta": {},
              "content": "Mercury",
              "type": "base"
            },
            {
          "key": 125, "meta": {},
              "content": "Venus",
              "type": "base"
            },
            {
          "key": 126, "meta": {},
              "content": "Earth",
              "type": "base"
            },
            {
          "key": 127, "meta": {},
              "content": "Mars",
              "type": "base"
            },
            {
          "key": 128, "meta": {},
              "content": "Jupiter",
              "type": "base"
            },
            {
          "key": 129, "meta": {},
              "content": "Saturn",
              "type": "base"
            },
            {
          "key": 130, "meta": {},
              "content": "Uranus",
              "type": "base"
            },
            {
          "key": 131, "meta": {},
              "content": "Neptune",
              "type": "base"
            }
          ],
          "collapsed": false
        }
      ],
      "collapsed": false
    }));
  }
};
```

### Step 4: 加入 Store 並使用 Dispatcher 註冊
到這一步我們先透過 `js/OutlineStartingData.js` 在本地端的 localStorage 建立了一些模擬資料，接著我們也透過 `js/utils/OutlineWebAPIUtils.js` 當作呼叫的 API 來取得模擬資料。
來看看關於 Flux 架構了，當一個 action 觸發了 Dispatcher，然後 `Stores` 監聽這些 action 來更新他們的內部結構以及觸發事件去更新 View。
是時候讓我們來建立 `Store` 它將用來管理記憶體中最新的資料，我們的 Store 將會有三個獨立的部分:
* 隱私的命名空間變數來保存記憶體中的資料
* 提供 View 一個方式來監聽 Store 是否發生改變以及存取資料
* 最後一個部分是在 dispatcher 中註冊 store ，回傳一組識別以用來同步其他 store。

 讓我們來看看這部分的細節:
 開始撰寫一個 `js/stores/OutlineStore.js` 這部分延伸自 `EventEmitter`
 因為我們只從模組中匯出  `OutlineStore`  不過變數 `_nodes` 卻掌控著我們的資料，這使得資料俱有隱蔽性，且不能從模組外部存取
```js
var AppDispatcher = require('../dispatcher/AppDispatcher');
var EventEmitter = require('events').EventEmitter;
var merge = require('react/lib/merge');
var CHANGE_EVENT = 'change';
var _nodes = {};
```
當 Store 發生改變時通知 View

```js
var OutlineStore = merge(EventEmitter.prototype, {
  emitChange: function () {
    this.emit(CHANGE_EVENT);
  },

  addChangeListener: function (callback) {
    this.on(CHANGE_EVENT, callback);
  },

  get: function (id) {
    return _nodes[id];
  },

  getAll: function () {
    return _nodes;
  },

  getSelected: function () {
    return _selected;
  }
});
```

還有註冊 OutlineStore 關注的 action 到 Dispatcher ，稍早我們只有在 action 加入 `RECEIVE_RAW_NODES`:

```js
OutlineStore.dispatchToken = AppDispatcher.register(function(payload) {
  var action = payload.action;

  switch (action.type) {
    case "RECEIVE_RAW_NODES":
      OutlineStore.emitChange();
      break;
    case "SELECT_NODE":
      _selected = action.key;
      OutlineStore.emitChange();
      break;
    default:
      /* do nothing */
  }
})
```

我們透過發動 emitChange 來回應一個 action，反過來說這將會通知 View

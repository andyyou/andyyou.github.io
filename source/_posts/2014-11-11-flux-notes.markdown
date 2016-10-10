---
layout: post
title: 'Flux 概念詳述篇'
date: 2014-11-11 12:25:00
categories: Program
tags: reactjs, flux
---

# 介紹
還記得之前小弟很認真的想跟大家分享 Flux 不過老實說在當時自己只能夠"模仿"，Dispatcher 和 Store 的觀念也有點模糊。
由於今天看了[這篇文章](http://scotch.io/tutorials/javascript/getting-to-know-flux-the-react-js-architecture)之後，覺得很不錯所以來補貼一下。
不過在這之前強烈推薦您還是先閱讀關於 Reactjs 的部分。
<!--more-->

# 什麼是 Flux
再一次我們說 Flux 是 Facebook 內部搭配 React 使用的一種架構，一種設計模式。它不是一個 Framework 或 Library 。
它單純是一種新的架構用來搭配補充 React 以及其單向資料流的概念。
然而我們也知道 Facebook 有提供一個 Dispatcher 的函式庫。這個 Dispatcher 是一個全域的發佈/訂閱處理函式庫，他可以廣播 payload (實際資料) 到被註冊的 callback 回呼函式。
一個典型的 Flux 架構將會使用這個 Dispatcher 函式庫配合 Nodejs 的 EventEmitter 模組來達成設置一個事件系統以協助管理應用程式的狀態。
如果用另一個角度來說那就是 Dispatcher 其實就只是處理事件的註冊與廣播，概念上我們可以先理解為一個 Pub/Sub 機制，各自把自己需求的事件註冊到這個管理中心，接著如果廣播觸發某一事件的時候
所有對應的事件都要被執行，而實際實作面就是透過 EventEmitter 來完成，想看實際的程式碼大概就是如下:

~~~js
var EventEmitter = require('events').EventEmitter,
    person = new EventEmitter();

person.on('speak', function() {
  console.log('I am here');
});

person.emit('speak');
~~~

要解釋 Flux 比較好的方式可能是透過說明每一個組成的局部:

* Actions - 輔助函式 Helper methods, 單純只是便利我們將資料傳給 Dispatcher
* Dispatcher - 接收 Actions 以及廣播 payloads 到被註冊的回呼函式(callback)
* Stores - 應用程式狀態和處理邏輯(即那些被註冊到 Dispatcher 的回呼函式)的容器，
* Controller Views - 一般來說就是那些負責管理 State ，把狀態透過 props 往下傳遞到子元件的 React 元件

看起來就會像下圖:

![](http://i.imgur.com/V70cSEC.png)

怎麼 API 也有關聯？

當你需要處理來自外部的資料，我們發現透過 Actions 來讓資料進入 Flux 流程接著進入 Stores 後續各個流程都將比較方便，是最無痛的方式。


# Dispatcher
所以到底什麼是 Dispatcher ?
基本上 Dispatcher 管理著整個流程。他就像是您應用程式的中央集線器，或者要把它比喻成電話總機。Dispatcher 收到 Actions 來的執行動作，接著分派這個動作和資料去給那些註冊的回呼函式。

所以本質上這就只是一個 發佈/訂閱 系統？
不全然是， Dispatcher 會廣播 payload 就是實際要傳遞的資料到所有被註冊的回呼函式包含讓你以特定順序執行回呼的功能，甚至是在執行之前先等更新完成。
記住在您的程式中永遠只有一個 Dispatcher ，只有一個總機小姐XD

接著讓我們來看看程式碼的部分:

~~~js
var Dispatcher = require('flux').Dispatcher;
var AppDispatcher = new Dispatcher();

AppDispatcher.handleViewAction = function(action) {
  this.dispatch({
    source: 'VIEW_ACTION',
    action: action
  });
}

module.exports = AppDispatcher;
~~~

在上面的範例，我們建立了一個 Dispatcher 的實例物件，建立了一個 `handleViewAction` 的方法。這樣的抽象化有助於區分是 View 觸發 Action 或者 Server/API 觸發 Action。
dispatch method 就是用來廣播 action 和 payload 到回呼函式。這個 action 接著就會觸發 Stores 內實際的行為函式然後更新狀態。
圖解如下
![](http://i.imgur.com/hKbN2q6.png)

# 相依性
Dispatcher 模組提供的其中一個最酷的部分就是可以替回呼函式定義相依性和序列化。所以當其中一個函式需要相依於另外一個時，為了適當的輸出，您可以使用 Dispatcher 的 waitFor 方法。
為了利用這個特性，我們需要在 Store 回傳並儲存 Dispatcher 的識別索引，而方式就是透過儲存 register 方法回傳的值。
程式碼範例如下:

~~~js
ShoeStore.dispatcherIndex = AppDispatcher.register(function(payload) {

});
~~~

然後在我們的 Store 裡，當處理一個被分派的 action 時我們就可以使用 waitFor 來確保上面那個回呼函式已經被執行

~~~js
case 'BUY_SHOES':
  AppDispatcher.waitFor([
    ShoeStore.dispatcherIndex
  ], function() {
    CheckoutStore.purchaseShoes(ShoeStore.getSelectedShoes());
  });
  break;
~~~

# Stores
在 Flux 中，Stores 針對特定需求管理應用程式的狀態。這基本上是指針對每一個應用程式，stores 管理著資料，接收資料的方法和 dispatcher 的回呼函式。
讓我們來看看 Store 的程式碼:

~~~js
var AppDispatcher = require('../dispatcher/AppDispatcher');
var ShoeConstants = require('../constants/ShoeConstants');
var EventEmitter = require('events').EventEmitter;
var merge = require('react/lib/merge');

// shoes 物件，用在內部處理資料
var _shoes = {};

// 從 action 載入 shoes 資料
function loadShoes(shoes) {
  _shoes = data.shoes;
}

// 將我們的 store 物件與 Node 的 Event Emitter 合體
var ShoeStore = merge(EventEmitter.prototype, {

  // 取得所有鞋子資料
  getShoes: function() {
    return _shoes;
  },

  emitChange: function() {
    this.emit('change');
  },

  addChangeListener: function(callback) {
    this.on('change', callback);
  },

  removeChangeListener: function(callback) {
    this.removeListener('change', callback);
  }

});

// 註冊 dispatcher callback
AppDispatcher.register(function(payload) {
  var action = payload.action;
  var text;
  // 定義如何處理特定動作 action
  switch(action.actionType) {
    case ShoeConstants.LOAD_SHOES:
      loadShoes(action.data);
      break;

    default:
      return true;
  }

  // 當 action 執行完畢則觸發 change event
  ShoeStore.emitChange();

  return true;

});

module.exports = ShoeStore;
~~~

上面程式碼中最重要的一件事就是我們擴充了 store 的功能加入了 EventEmitter。之後我們的 Stores 就可以監聽和廣播事件。
而我們的 View 和元件就可以根據事件來更新，因為 Contriller View 監聽 Stores，利用觸發 change event 就可以讓 Controller View 知道程式的狀態已經變更了該更新狀態。
我們同時也透過 register 方法註冊了一個 Callback 到 AppDispatcher ，這麼一來 Store 就開始監聽 AppDispatcher 的廣播。
上面的 switch 敘述式會決定處理的方式針對對應的 action 然後發出一個廣播，接著觸發 change event 因為 view 監聽著 change event 所以就可以根據事件來更新狀態。

![](http://i.imgur.com/rHwGUog.png)

Controller View 就可以透過 `getShoes` 來檢索 `_shoes` 裡面的資料，當然這只是一個單純的範例，複雜的邏輯都可以放在這邊。


# Action Creators 和 Actions
Action Creators 是一系列方法的集合，我們可以在 View 呼叫他們，透過他們把 action 發送給 Dispatcher. Actions 實際上只是透過 dispatcher 來分派實際資料。
關於 Facebook 如何使用它們 - action 類型常數被用來定義該執行什麼樣的行為，並且包含著資料一起被送出。在註冊的回呼函式裡這些 action 可以根據"類型"被處理，且方法可把 action 中的 data 當作參數。

讓我們來看看關於常數是如何定義的:

~~~js
var keyMirror = require('react/lib/keyMirror');

module.exports = keyMirror({
  LOAD_SHOES: null
});
~~~

上面程式碼，我們使用了 React 的 `keyMirror` 函式庫，沒錯！您已經猜到了。鏡射我們的 keys 所以我們的值就會直接等於我們的 key.

~~~js
{ LOAD_SHOES: 'LOAD_SHOES' }
~~~

之後我們只要透過觀察這支檔案，我們就能得知關於 `ShoeStore` 的行為，透過常數幫助我們讓事情更具有組織性，也能讓我們得知關於這個程式實際上在做什麼。

現在我們可以來看看對應的 Action Creator

~~~js
var AppDispatcher = require('../dispatcher/AppDispatcher');
var ShoeStoreConstants = require('../constants/ShoeStoreConstants');

var ShoeStoreActions = {

  loadShoes: function(data) {
    AppDispatcher.handleAction({
      actionType: ShoeStoreConstants.LOAD_SHOES,
      data: data
    })
  }

};

module.exports = ShoeStoreActions;
~~~

在上面的範例中，我們在 `ShoeStoreActions` 建立了一個方法，它可以呼叫 dispatcher 並傳入我們提供的資料。現在我們就能夠在我們的 view 或者 API 匯入這隻 actions 檔案
透過 `ShoeStoreActions.loadShoes(ourData)` 把資料和 actionType 傳給 Dispatcher 並廣播。接著 ShoeStore 將會聽到該事件被呼叫而去執行對應的方法來載入鞋子的資料。

# Controller Views
Controller Views 就只是 React 元件且該元件正監聽著 change event ，從 Stores 接受應用程式的狀態和資料。當然它可以把資料透過 props 傳遞給子元件。
![](http://i.imgur.com/4tBnC0e.png)

程式碼看起來便會如下

~~~js
/** @jsx React.DOM */

var React = require('react');
var ShoesStore = require('../stores/ShoeStore');

// Method to retrieve application state from store
function getAppState() {
  return {
    shoes: ShoeStore.getShoes()
  };
}

// Create our component class
var ShoeStoreApp = React.createClass({

  // Use getAppState method to set initial state
  getInitialState: function() {
    return getAppState();
  },

  // Listen for changes
  componentDidMount: function() {
    ShoeStore.addChangeListener(this._onChange);
  },

  // Unbind change listener
  componentWillUnmount: function() {
    ShoesStore.removeChangeListener(this._onChange);
  },

  render: function() {
    return (
      <ShoeStore shoes={this.state.shoes} />
    );
  },

  // Update view state when change event is received
  _onChange: function() {
    this.setState(getAppState());
  }

});

module.exports = ShoeStoreApp;
~~~

上面的範例中，我們透過 `addChangeListener` 把對應更新的事件註冊進去，當事件被廣播時元件就可以執行 this._onChange 函式來更新元件的狀態。
應用程式的狀態和資料都被保存在 Stores，所以我們可以使用 Stores 中任何 public 的方法來取得資料或狀態。

# 結合所有的東西
現在我們已經各別解釋關於 Flux 結構的每個部分，我們應該具備關於這個架構實際上是如何運作的觀念，讓我們更仔細地來看看下面運作的圖示
![](http://i.imgur.com/duZH2Sz.png)

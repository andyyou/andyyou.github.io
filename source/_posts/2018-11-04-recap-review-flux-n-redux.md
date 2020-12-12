---
title: 快速複習 Flux 和 Redux
tags:
  - javascript
  - reactjs
categories: Program
date: 2018-11-04 19:10:30
---

![](https://cdn-images-1.medium.com/max/949/1*3lvNEQE4SF6Z1l-680cfSQ.jpeg)

<!-- more -->

#### Flux

*Flux* 就是一個設計模式，用於管理資料傳遞流程。其核心概念為單一方向性的資料流。

*Flux* 由下列 3 個角色組成：

* Dispatcher：接收 *action* 然後發送給 *store*，每一個有使用 *dispatcher* 註冊的 *store* 都會收到 *action*。一個應用程式只會有一個 *dispatcher*。
  例如：

  1. 使用者輸入代辦項目然後點擊 Enter
  2. React 擷取事件之後發送一個 `ADD_TODO` *action* （dispatch an action） 這個 *action* 包含代辦項目的文字資料
  3. 接著，每一個 *store* 都會收到這個 *action*

* Store：負責管理資料。*strore* 需要向 *dispatcher* 註冊，才能夠收到 *action*。儲存在 *store* 中的資料只能夠依據 *action* 變更。變更之後必須要發送一個 `change` 的事件
  例如：

  1. *store* 收到 `ADD_TODO` *action*
  2. *store* 內部自行將 *action* 中的資料加入清單中
  3. *store* 更新完代辦清單之後 *emit* 一個 *change event*

* Action：定義應用程式內部行為。使用一個物件描述將產生的變更行為與資料（*payload*）
  例如：

  1. 使用者點擊刪除，則發生一個 `DELETE_TODO` *action*

     ```json
     {
       type: 'DELETE_TODO',
       id: '12'
     }
     ```


使用 3 者搭配 *View（React）* 來開發 *Client-Side* 網頁應用程式。

![](https://raw.githubusercontent.com/facebook/flux/master/examples/flux-concepts/flux-simple-f8-diagram-with-client-action-1300w.png)

<p data-height="300" data-theme-id="8540" data-slug-hash="oQNzjx" data-default-tab="js" data-user="andyyou" data-pen-title="oQNzjx" class="codepen">See the Pen <a href="https://codepen.io/andyyou/pen/oQNzjx/">oQNzjx</a> by andyyou (<a href="https://codepen.io/andyyou">@andyyou</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>



#### Redux

*Redux* 是給 JavaScript 應用程式使用的一種*狀態管理容器*。

使用 `createStore` 建立一個全應用程式唯一的 `store`，`store` 須使用 *reducer* （*Pure Function*）來變更內部的 *state*。

執行狀態變更須發送 *action* （`store.dispatch(action)`），一個 *action* 只是一個單純的物件須包含 `type` 屬性用來描述變更行為，對應的行為在 *reducer* 中實現。

> 搭配 React 使用時要將 store 注入，可使用 react-redux 的 <Provider>
> 其他子元件要取得 state 或 dispatch 則使用 High-Order Component `connect`

*Redux* 由下列幾個單元組成：

* Store：整個應用程式只有一個 *Store*

  ```js
  import { createStore } from 'redux';
  const store = createStore(reducer);
  // 取得狀態
  store.getState();
  // 訂閱
  let unsubscribe = store.subscribe(() => {
    console.log(store.getState());
  });
  // 解除
  unsubscribe();
  ```

* Action：*action* 是一個單純的物件，須包含 `type` 屬性。要變更 *state* 必須要發送 *action*。

  ```js
  const action = {
    type: 'ADD_TODO',
    text: 'Learn Redux',
  };
  // 發送 action 的方式
  store.dispatch(action);
  ```

* Reducer：發送了 *action* 之後，須變更 *state* ，使 *state* 變更的計算過程稱為 *reducer*。*reducer* 為一函式接收 *action* 和 *state* 回傳一個新的計算後的 *state*。

  ```js
  const todos = function(state, action) {
    switch(action.type) {
      case 'ADD_TODO':
        return [
          ...state,
          action.text,
        ];
      default:
        return state;
    }
  }
  ```

<p data-height="300" data-theme-id="8540" data-slug-hash="rQNRVY" data-default-tab="js" data-user="andyyou" data-pen-title="rQNRVY" class="codepen">See the Pen <a href="https://codepen.io/andyyou/pen/rQNRVY/">rQNRVY</a> by andyyou (<a href="https://codepen.io/andyyou">@andyyou</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

---
title: "React 18 - 了解 useSyncExternalStore"
date: 2022-01-05 17:27:48
tags:
  - javascript
  - react
categories: Program
---


在深入 `useSyncExternalStore` 之前，讓我們先了解一下一些新的術語。

## 併發渲染與 `startTransition`

併發指的是基於分配任務的優先順序同時執行多個任務的機制。如果您還不明白這個觀念可以[參考 Dan Abramov 的說明](https://github.com/reactwg/react-18/discussions/46#discussioncomment-846786)

<!-- more -->

> 重點節錄：
> 併發的意思是任務可以在同一段時間重疊。
> 讓我們使用打電話來比喻。不支援併發意思是，我一次只能和一個人通電話。如果我打給 Alice，然後 Bob 打給我，我必須要掛掉 Alice 的電話然後才能和 Bob 講話。
> ![](https://user-images.githubusercontent.com/810438/121394782-9be1e380-c949-11eb-87b0-40cd17a1a7b0.png)
> 併發表示我每一在同一段時間有多個通話。例如：我還是保持和 Alice 通話狀態只是將電話擺在旁邊，然後跟 Bob 講話，後續還是可以回來和 Alice 講話。
> ![](https://user-images.githubusercontent.com/810438/121394880-b4ea9480-c949-11eb-989e-06a95edb8e76.png)
> 注意：併發不是說我一次要跟兩個人對話。而在 React 這個打電話（併發任務）指的是 `setState`。

在新的 `startTransition` API 的協助下我們可以選擇在渲染期間讓應用程式保持可操作。換句話說，React 現在可以暫停渲染，這讓瀏覽器可以處理中間的事件。

> 更多介紹可以參考﹔[概覽 React 18 新功能](https://andyyou.github.io/2021/10/14/whats-new-in-react-18/)

### 外部儲存

所謂外部儲存就是一個我們可以訂閱的東西，例如 Redux store，全域變數，模組內部變數，DOM 狀態等等。

### 內部儲存

內部則是 `props`，`context`，`useState`，`useReducer` 這些。

### Tearing

"Tearing" 指的是視覺上的不一致性。表示介面會發生同一個狀態卻不同值的情況。

在 React 18 之前沒有這個問題。但 React 18 併發渲染的功能讓這個問題可能發生，因為在渲染期間會暫停。在這些暫停，更新之間會讀取新的渲染中使用的資料。這導致 UI 針對同一份資料來源顯示兩個不同的值。

讓我們來看看[關於 Tearing 討論](https://github.com/reactwg/react-18/discussions/69)中的範例。下面是一個元件需要存取外部儲存來取得顏色。在同步渲染情況下 UI 渲染的顏色一致

![](https://user-images.githubusercontent.com/2440089/124805929-23f7e080-df2a-11eb-99c7-776812e89908.png)

而在併發渲染下，一開始讀取的顏色是藍色。然後 React  暫停，此時外部儲存將顏色更新為紅色。當 React 恢復繼續渲染就會取得紅色。這就會造成介面上呈現不一致也就是 "Tearing"

![](https://user-images.githubusercontent.com/2440089/124805949-29edc180-df2a-11eb-9621-4cd9c5d0bc5c.png)

為了修復這個問題，React 團隊[加入了 `useMutableSource`](https://github.com/reactjs/rfcs/blob/main/text/0147-use-mutable-source.md)來安全且效率的讀取外部資料(Mutable External Source)。但工作群組的成員回報了[既有 API 整合使用的問題](https://github.com/reactwg/react-18/discussions/84)導致函式庫維護者很難使用 `useMutableSource` 到他們的實作。在一陣討論之後 `useMutableSource` 被重新設計並改名字為 `useSyncExternaStore`

## 了解 `useSyncExternalStore` Hook

[React 18 新的 `useSyncExternalStore` Hook](https://github.com/reactwg/react-18/discussions/86) 讓我們可以正確的訂閱儲存中的值。

為了協助簡化升級過程，向下兼容，React 提供了 [use-sync-external-store](https://www.npmjs.com/package/use-sync-external-store) 套件。這個套件包含的  "shim"  適用於 React 任何版本。

 ```jsx mark:3
 import { useSyncExternalStore } from 'react';
 // 或者
 // 向下相容
 import { useSyncExternalStore } from 'use-syncexternal-store/shim';
 
 // 基本使用方式。getSnapshot 必須回傳 cached/memoized 結果
 useSyncExternalStore(
   subscribe: (callback) => unsubscribe,
   getSnapshot: () => state
 );
 
 // 讀取特定欄位
 const selectedField = useSyncExternalStore(store.subscribe, () => store.getSnapshot().selectedField);
 ```

`useSyncExternalStore` Hook 參數為兩個函式

* `subscribe` 函式註冊了一個訂閱的 callback
* `getSnapshot` 是用來檢查已訂閱的資料從上次渲染之後是否有發生變更。它必須是數字，字串這種靜態資料或者 cached / memoized 的物件。然後 Hook 會回傳 Immutable 的資料。

其中也支援 `getSnapshot` 結果會自動快取 memoize  回傳結果的 API

```jsx
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/with-selector';

const selection = useSyncExternalStoreWithSelector(
	store.subscribe,
  store.getSnapshot,
  getServerSnapshot,
  selector,
  isEqual
);
```

要了解會發生了什麼問題請先觀看[這個影片](https://www.youtube.com/watch?v=oPfSC5bQPR8&t=453s)。影片中的範例如下您可以自行測試

<iframe src="https://codesandbox.io/embed/use-sync-external-store-demo-errors-9bu29?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="use-sync-external-store-demo-errors"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

<iframe src="https://codesandbox.io/embed/use-sync-external-store-demo-1-voq56?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="use-sync-external-store-demo-1"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

上面的例子說明，使用 `startTransition` 有可能會踩到 "Tearing" 問題。如果要修正問題就需要使用 `useSyncExternalStore`。

在 `useStore` ，使用 `useSyncExternalStore` 取代 `useEffect` 和 `useState`

```jsx
import { useSyncExternalStore, useCallback } from 'react';

const useStore = (store, selector) => {
  return useSyncExternalStore(
  	store.subscribe,
    useCallback(() => selector(store.getState()), [store, selector]);
  );
}
```

使用新的 Hook 讓程式碼更乾淨，好維護。建議使用 `useSyncExternalStore` 來處理外部儲存，更簡單也避免潛在的問題發生。

> ⚠️⚠️⚠️注意⚠️⚠️⚠️﹔實作併發模式，您必須使用 React 18 新的 `render` API 否則不會啟動功能
>
> ```jsx
> import * as ReactDOM from 'react-dom';
> import App from 'App';
> 
> const root = ReactDOM.createRoot(document.getElementById('app'));
> root.render(<App />);
> ```

### 哪些函式庫受到影響?

* 函式庫或客製化 Hook 在渲染期間不使用外部儲存，僅使用 `props`，`state`，`context` 的不受影響。
* 函式庫會處理資料讀取，狀態管理例如 Redux，MobX，Relay 會受到影響，因為他們把狀態儲存到 React 外部。在併發渲染時這些資料可以在渲染過程中更新，而 React 不知道。 

## 參考資料

* [RFC: useMutableSource](https://github.com/reactjs/rfcs/pull/147)
* [Discussion regarding useMutableSource and selector stability](https://github.com/reactwg/react-18/discussions/84)
* [Journey of useMutableSource to useSyncExternalStore](https://github.com/reactwg/react-18/discussions/86)
* [Meet the new hook useSyncExternalStore, introduced in React 18 for external stores](https://blog.saeloun.com/2021/12/30/react-18-usesyncexternalstore-api?ck_subscriber_id=887777478)
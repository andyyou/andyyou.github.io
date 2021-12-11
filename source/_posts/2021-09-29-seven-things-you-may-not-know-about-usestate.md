---
title: 您可能不知道關於 useState 的 7 件事
date: 2021-09-29 11:11:42
tags:
  - javascript
  - react
categories: Program
---

在為我們的專案（React）進行程式碼審查的時候，我常發現開發成員沒有意識到關於 `useState`  提供的一些好用功能或討厭的陷阱。雖然這些觀念不是什麼重大的啟發，但每一個使用 Hook 的人都應該要了解。

<!-- more -->

## 更新用的 setter 具有一致的參考

為了更具體的說明這點；所謂的更新 setter 就是陣列中的第二個函式，它們在每一次 render 時都會保持一致。因此您不需要將它們加入例如 `useEffect` 的相依參考。不要管 [eslint-plugin-react-hooks](https://reactjs.org/docs/hooks-rules.html#eslint-plugin) 給出什麼警告。

```js
const [count, setCount] = useState(0);
const onChange = useCallback((e) => {
    // setCount 永遠不會變
    setCount(Number(e.target.value));
}, []);
```



## 設定相同的狀態值，什麼也不會執行

`useState` 預設屬於 Pure Function (意指相同的輸入，永遠會得到相同的輸出，而且沒有任何顯著的副作用)。使用相同於當前的值執行更新函式不會有任何變化 - 不會更新 DOM，也不會刷新渲染，什麼事也沒有。

```js
const [isOpen, setOpen] = useState(props.initOpen);
const onClick = () => {
    // useState 已經為我們判斷了
    if (!isOpen) {
        setOpen(true);
    }
};
```

但是對於物件不適用：

```js
const [{ isOpen }, setState] = useState({ isOpen: true });
const onClick = () => {
  	// 會觸發更新，因為物件參考是不一樣的
    setState({ isOpen: false });
};
```

## 回傳 `undefined` 狀態

這表示 `setState` 可以直接從 `useEffect` 箭頭函式中回傳。警告：Effect 參數函式不能回傳除了清除用的函式以外的東西

```js
useLayoutEffect(() => {
    setOpen(true);
}, []);
useLayoutEffect(() => setOpen(true), []);
```

## useState 即 useReducer

事實上，`useState` 在 React 內部的實作類似於一個 `useReducer`，只是搭配一個預先定義的 reducer ，至少在 17.0 版本開始是這樣。參考[原始碼](https://github.com/facebook/react/blob/82c8fa90be86fc0afcbff2dc39486579cff1ac9a/packages/react-reconciler/src/ReactFiberHooks.new.js#L1464)。如果有人聲稱 `useReducer` 有更進階的技術優勢，他是騙人的。

## 您可以使用 callback 初始化狀態

您可以使用初始化函式來取代物件

```js
const [style, setStyle] = useState(() => ({
    transform: props.isOpen ? null : 'translateX(-100%)',
    opacity: 0
}));
```

您可以在初始化函式中使用 `props` 。坦白說，這有點過度優化。您都可以建立一堆 vDOM 了，為啥要擔心一個物件？不過對於繁重的初始化邏輯這的確有幫助。

另外，如果您想要在狀態使用函式，您可以額外在包一層函式 `useState(() => () => console.log('gotcha!'))`

## 您可以使用 callback 更新狀態

Callback 函式也可以用來更新狀態；像一個沒有 action 的 reducer。由於當前的狀態和閉包的狀態值可能不同

```js
const [clicks, setClicks] = useState(0);
const onMouseDown = () => {
  	// 這樣會跟預期不一樣，您以為總共 +2 但只有 + 1
    setClicks(clicks + 1);
    setClicks(clicks + 1);
};
const onMouseUp = () => {
    // 維持用閉包紀錄的值 + 1
    setClicks(clicks + 1);
  	// 這樣就可以讀取最新當前的值，完成我們的需求
    setClicks(clicks => clicks + 1);
};
```

比較正確的作法

```js
const [isDown, setIsDown] = useState(false);
// 不好，每次 isDown 變更都會更新
const onClick = useCallback(() => setIsDown(!isDown), [isDown]);
// 比較好
const onClick = useCallback(() => setIsDown(v => !v), []);

```

## 一個狀態更新 = 一次非同步的渲染

React 有一個功能稱為 batching，它會強制多個 `setState` 調用彙整成一次渲染，但不總是如預期的運作。我們來看一下下面的程式碼：

```js
console.log('render');
const [clicks, setClicks] = useState(0);
const [isDown, setIsDown] = useState(false);
const onClick = () => {
    setClicks(clicks + 1);
    setIsDown(!isDown);
};
```

當您呼叫 `onClick` 的時候，會渲染幾次取決於您如何調用。查看 [範例](https://codesandbox.io/s/setstate-multi-29z2u?file=/src/App.js)

* 使用 `<button onClick={onClick}>` 會正確彙整成一次渲染
* `useEffect(onClick, [])` 也會正確彙整
* `setTimeout(onClick, 100)`就會觸發額外的渲染
* `el.addEventListener('click', onClick)` 也**不會**正確彙整成一次渲染

這些在 React 18 會[更新](https://github.com/reactwg/react-18/discussions/21)，在這之前您需要使用 `unstable_batchedUpdates` 來強制彙整

```js
import {unstable_batchedUpdates} from 'react-dom';
unstable_batchedUpdates(() => {
  setClicks(clicks + 1);
  setIsDown(!isDown);
});
```

## 總結

* `[state, setState] = useState()` 的 `setState` 每次渲染參考都一樣
* `setState(currentValue)` 不會做任何事。`if (value !== currentValue)` 可以省略。
* `useEffect(() => setState(true))` 不會破壞 Effect 清除功能
* `useState` 內部的實作是一個預先定義 reducer 的 `useReducer`
* 初始化可以使用 callback；`useState(() => initialValue)`
* 當前狀態的更新建議使用 callback，尤其是搭配 `useCallback`
* React 彙整多次狀態更新成一次渲染的機制在某些情況下不會正確運作

## 參考

* [7 things you may not know about useState](https://thoughtspile.github.io/2021/09/27/usestate-tricks/?ck_subscriber_id=887777478)
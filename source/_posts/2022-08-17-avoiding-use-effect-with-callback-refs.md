---
title: '使用 Callback Refs 取代 useEffect 避免問題產生'
date: 2022-08-17 15:24:36
tags:
  - react
  - javascript
categories: Program
---

雖然 `ref` 是一個可變更 (mutable 物件在建立後可被修改) 的容器，我們理論上可以儲存任何資料。通常我們會拿來處理存取 DOM 節點:

```js
const ref = React.useRef(null);

return <input ref={ref} defaultValue='Hello world' />;
```

`ref` 是一個內建的屬性，React 會在渲染後儲存 DOM 節點，當元件卸載後也會設為 `null`。

<!-- more -->

## 使用 `refs` 互動

對於大多數操作互動，您不需要直接存取 DOM ，因為 React 會自動幫我們處理更新。一個最好的使用範例就是當您需要處理 focus 行為的時候。

[Devon Govett](https://twitter.com/devongovett) 雖然提出了相關的 [RFC](https://github.com/devongovett/rfcs-1/blob/patch-1/text/2019-focus-management.md) 旨在加入 FocusManagement 到 react-dom，但目前沒有內建的機制協助處理。

## 使用 Effect 來處理 focus

那麼目前我們要如何處理渲染後 focus 呢? 當然我們都知道 [autofocus](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/autofocus) 但如果您希望加入一些動畫或複雜的行為。

好, 我見過大多數的人都嘗試下面的做法:

```jsx
const ref = React.useRef(null);

React.useEffect(() => {
  ref.current?.focus();
}, []);

return <input ref={ref} defaultValue='Hello world' />;
```

這種做法在多數情況下沒啥問題。Effect 的第二個參數為空沒有問題，因為裡面的 `ref` 是可變更的，linter 不會警告，並且 `ref` 在渲染時期也不會被讀取(不過這對 React Concurrent 功能可能會有問題)。

Effect 只會在掛載時執行一次。此時，React 已經把 DOM 設到 `ref`，因此可以 focus。

然而這並不是最好的做法，並且在一些複雜的情況下會有問題。

具體來說，我們假設 `ref` 在 Effect 執行時設定完成。但如果不是呢? 例如你把 `ref` 傳入自訂的元件，這個元件會延後渲染或只有在某些操作之後才出現，這個情況下 `ref` 就會是 `null` 即便執行了 Effect 還是不會有 focus 效果。

```jsx
function App() {
  const ref = React.useRef(null);

  React.useEffect(() => {
    // ref.current 永遠是 null
    ref.current?.focus();
  }, []);

  return <Form ref={ref} />;
}

const Form = React.forwardRef((props, ref) => {
  const [show, setShow] = React.useState(false);

  return (
    <form>
      <button type='button' onClick={() => setShow(true)}>
        Show
      </button>
      // ref 掛載到 input 但需要符合條件才會渲染 // 因此在 Effect 執行的時候未被設定
      {show && <input ref={ref} />}
    </form>
  );
});
```

下面是流程說明:

- `Form` 渲染
- `<input>` 沒有渲染，`ref` 依舊是 null
- Effect 執行，什麼都沒發生
- `<input>` 顯示，`ref` 設定完成，但不會 focus 因為 Effect 不會執行

問題在於 Effect 是因為 Form 渲染才執行的關係，而我們實際上需要的是當 input 渲染完成執行 focus，並非當表單掛載時。

## Callback Refs

這就是 Callback Refs 派上用場的地方了。如果您曾經看過 `ref` 的型別宣告，您可以看到我們不只可以傳入 ref 物件，也可以用 function:

```ts
type Ref<T> = RefCallback<T> | RefObject<T> | null;
```

概念上，我喜歡 React 元素的 ref 當作函式，它會在元件渲染完成之後執行。這個函式會取得渲染的 DOM 節點作為參數。如果 React 元素卸載，也會再次執行並傳入 `null`。

將 `useRef` (一個 RefObject) 傳入 `ref` 到 React 元素只是語法糖:

```jsx
<input
  ref={(node) => {
    ref.current = node;
  }}
  defaultValue='Hello world'
/>
```

讓我們在強調一次

> 所有 ref props 只是函式!

並且這些函式會在渲染之後執行，執行包含副作用的程式完全是可以的。如果 `ref` 叫做 `onAfterRender` 或許更直覺。

有了這個知識，我們是否可以直接存取 `node`?

```jsx
<input
  ref={(node) => {
    node?.focus();
  }}
  defaultValue='Hello world'
/>
```

另外一個小細節: React 在每次渲染之後都會 focus。因此除非您可以接受每次都 focus ，否則我們需要告訴 React 只需要在我們希望的時候執行一次。

## 使用 useCallback 解決

幸運的是 React 會檢查參考是否相同來決定是否應該要執行。意思是如果我們傳入相同參考就會忽略執行。

此時， `useCallback` 就可以派上用場，因為這就是我們確保不會反覆重新建立函式的方法。這也是為什麼成為 Callback Refs 因為你需要將他們包在 `useCallback` 裡面。

```jsx
const ref = React.useCallback((node) => {
  node?.focus();
}, []);

return <input ref={ref} defaultValue='Hello world' />;
```

相較於第一個 Effect 版本，更少的程式，而且只用一個 Hook。同時也滿足所有情況。外加即使在開發環境 strict mode 也不會執行兩次。

如同隱藏在[舊版文件的段落](https://reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node)所介紹，您可以在其中使用任何副作用的操作，執行 `setState`。

```jsx
function MeasureExample() {
  const [height, setHeight] = React.useState(0);

  const measureRef = React.useCallback((node) => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
```

因此，如果您需要在渲染之後直接操作 DOM ，試著不要直接用 `useRef` + `useEffect`，嘗試 Callback Refs 取代。

## 參考

- [Avoiding useEffect with callback refs](https://tkdodo.eu/blog/avoiding-use-effect-with-callback-refs)

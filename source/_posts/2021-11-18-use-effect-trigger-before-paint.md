---
title: "[譯] useEffect 有時候會在瀏覽器繪製（Paint）之前觸發"
date: 2021-11-18 12:07:29
tags:
  - react
  - javascript
categories: Program
---

`useEffect` 應該在瀏覽器渲染（ `paint()` ）之後執行，以防止阻塞更新。但您知道它並沒有保證一定在渲染之後觸發？在 `useLayoutEffect` 中更新狀態（`state`）會導致同一次渲染中的 `useEffect` 在渲染之前執行，這是為了有效率的處理佈局的效果。感到困惑嗎？

<!-- more -->

來看看一個普通的流程如下：

1. React；渲染 Virtual DOM，Effects 排程，更新實際 DOM
2. 執行 `useLayoutEffect`
3. React；釋放控制，瀏覽器渲染新的 DOM
4. 執行 `useEffect`

首先，[React 文件](https://reactjs.org/docs/hooks-reference.html#useeffect)並沒有說明具體準確 `useEffect` 觸發時機；在佈局和渲染之後，在一個延遲時間期間。因此我一直都以為就是類似 `setTimeout(effect, 3)` 的方式，但其實是[使用 `MessageChannel` 的技巧](https://stackoverflow.com/a/56727837) 。

而文件上更有趣的內容：

> 雖然 useEffect 會延遲直到瀏覽器繪製（paint）完成，它保證會在下一次新的渲染之前觸發。React 會在下次更新之前刷新之前的 Effects 

這是一個不錯的保證 - 您可以確保更新不會不見。但這也意味著 Effect 是有可能在瀏覽器 `paint` 之前觸發的。

如果

* Effect 會在下一次更新開始之前會被刷新
* 更新是可以在瀏覽器 `paint` 之前觸發，例如使用 `useLayoutEffect`，然後 Effect 必然在下次更新之前刷新，即在 `paint` 之前刷新；

下面是圖示：

![](https://thoughtspile.github.io/images/forced-le-flush-chart-5dc51705d5854315a6fa5e0be1464f7d.png)

1. React 更新 1；渲染 Virtual DOM，排程 Effect，更新 DOM
2. 執行 `useLayoutEffect`
3. 更新 `state`，造成重新渲染
4. 執行 `useEffect`
5. React 更新 2；
6. 從第二次更新執行 `useLayoutEffect`
7. React 釋放控制，瀏覽器渲染新的 DOM
8. 執行第二次更新的 `useEffect`

這不是一個非常罕見的例子；避免在 `useEffect` 直接更新狀態，因為狀態更新會更新 DOM，並且在渲染之後會先得到一個舊的畫面然後更新，[導致畫面閃爍](https://blog.logrocket.com/useeffect-vs-uselayouteffect/)。

舉例來說：我們建置一個自適應的輸入欄位，如果欄位寬大於 `200px` 則多渲染一個清除按鈕。我們需要實際的 DOM 來得知寬。

```js
const ResponsiveInput = ({ onClear, ...props }) => {
  const el = useRef();
  const [w, setW] = useState(0);
  const measure = () => setW(el.current.offsetWidth);
  useLayoutEffect(() => measure(), []);
  
  useEffect(() => {
    window.addEventListener('resize', measure);
    return () => window.removeEventListener("resize", measure);
  }, []);
  return (
  	<label>
    	<input {...props} ref={el} />
    	{w > 200 && (
      	<button onClick={onClear}>Clear</button>
      )}
    </label>
  );
}
```

我們已經利用 `useEffect` 延遲了 `addEventListener` 希望它在 `paint` 之後在加入監聽事件，但由於 `useLayoutEffect` 更新了狀態導致被強制在 `paint` 之前執行。[(範例 Sandbox)](https://codesandbox.io/s/infallible-wildflower-127lv?file=/src/App.js:294-408)

![](https://thoughtspile.github.io/images/le-flush-paint-dd310ab13e4d418b82b96366577d4c70.png)



`useLayoutEffect` 不是唯一強制提早 Effect 的地方，Refs 例如 `<div ref={HERE} />`，`requestAnimationFrame` 迴圈， Microtasks 排程一樣會導致提早。

但某些情況下渲染流程沒有最佳化其實也沒那麼糟糕。誰會在乎呢？但了解工具的限制對您還是很有幫助的；

下面有 4 個可以學習的地方：

### 不要太依賴 useEffect 在更新之後觸發

即使您知道問題所在，但還是很難確保某些 `useEffect` 不受 `useLayoutEffect` 狀態更新的影響：

1. 我的元件沒有使用 `useLayoutEffect`。但您確定其他函式庫的 Hook 呢？例如 `usePopper`
2. 我的元件只使用內建的 Hook，但 `useContext` 或上層元件的 re-render 有可能造成 uLE 狀態更新
3. 我的元件只有 `useEffect` 和搭配 `memo()` 。但 Effect 有[全域刷新](https://github.com/facebook/react/blob/4ff5f5719b348d9d8db14aaa49a48532defb4ab7/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L769)的情況，因此一個在 `paint` 之前的狀態更新，其子元件還是會受到影響。

現在您可能會考慮不在 `useLayoutEffect` 更新狀態，但那有點困難。比較好的建議是不要太依賴 `useEffect` 會在渲染之後觸發，就像 `useMemo` 也沒有 100% 穩定參考。如果您希望使用者在渲染之後看到某些畫面， `useEffect` 不是最好的選擇，嘗試 `requestAnimationFrame` 或 `postMessage` 。

反過來，假設您沒有聽說在 `useEffect` 更新 DOM，然後測試看看有沒有閃爍。

## 不要浪費時間拆分 Layout Effects

遵循 `useEffect` 和 `useLayoutEffect` 的指導原則，我們可能會把一個 Side-effect 分拆到 `useLayoutEffect` 和 `useEffect` 就像上面的範例

```js
// DOM update = layout effect
useLayoutEffect(() => setWidth(el.current.offsetWidth), []);
// subscription = lazy logic
useEffect(() => {
  window.addEventListener('resize', measure);
  return () => window.removeEventListener('resize', measure);
}, []);
```

但我們知道這樣並沒有什麼差異，兩個 Effect 都會在下次渲染之前刷新。如果我們假設 `useEffect` 會在渲染之後觸發，您可以 100% 確保在兩個 Effect 之間尺寸不會被變更嗎？如果不是那就讓邏輯全部放在 `useLayoutEffect` 就好了

```js
useLayoutEffect(() => {
  setWidth(el.current.offsetWidth);
  window.addEventListener('resize', measure);
  return () => window.removeEventListener('resize', measure);
}, []);
```

### 不要在 `useLayoutEffect` 更新狀態

這是很好的建議，但說的比做的簡單。在  `useEffect` 更新狀態也是很糟糕因為閃爍會造成 UX 體驗不好。

有時候狀態更新可以被 `useRef` 完全取代。更新 `ref` 不會造成重新渲染，Effect 可以如預期的執行。這裡有篇文章可以[參考](https://thoughtspile.github.io/2021/10/18/non-react-state/)

如果可以盡量不要依賴 `useEffect`。

### 繞過狀態更新

如果您發現特定 `useLayoutEffect` 造成問題，試著繞過狀態更新，直接操作 DOM。

```js
const clearRef = useRef();
const measure = () => {
  // No worries react, I'll handle it:
  clearRef.current.display = el.current.offsetWidth > 200 ? null : none;
};
useLayoutEffect(() => measure(), []);
useEffect(() => {
  window.addEventListener("resize", measure);
  return () => window.removeEventListener("resize", measure);
}, []);
return (
  <label>
    <input {...props} ref={el} />
    <button ref={clearRef} onClick={onClear}>clear</button>
  </label>
);
```

手動管理 DOM 更新通常比較複雜且容易出錯的，因此在性能很糟的情況下再使用這個技巧。

本文探討了 `useEffect` 有時候會在 `paint` 之前執行，常見的原因是在 `useLayoutEffect` 更新狀態，因為它在 `paint` 之前重新渲染導致 Effect 必須得提早執行。這些意味著：

1. 在 `useLayoutEffect` 更新狀態不利於效能，但有時候沒有其他替代方案
2. 不要依賴 `useEffect` 會在渲染後觸發這個點
3. 在 `useEffect` 更新 DOM 會造成閃爍 - 
4. 將 `useLayoutEffect` 部分邏輯移到 `useEffect` 對效能是沒有意義的
5. 在性能問題下，手動 uLE 操控 DOM 的一個理由

## 參考

* [useEffect sometimes fires before paint](https://thoughtspile.github.io/2021/11/15/unintentional-layout-effect/?ck_subscriber_id=887777478)
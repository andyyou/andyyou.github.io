---
title: "(譯) 關於 React 併發功能與 Suspense 您所需要了解的事"
date: 2023-01-21 23:20:36
tags:
  - react
  - javascript
categories: Program
---

使用者介面（UI）是由許多不同“部分”組成，而每個部分會以不同的頻率回應使用者的操作。有些例如表單的輸入欄位（`<input>`），需要即時回應使用者的操作，而其他像是內容很長的“篩選列表”或切換頁面則稍微慢一些。

在同步渲染的情況下，也就是沒有併發功能的 React 或者大部分的 JavaScript UI 框架/函式庫。一些回應比較慢的介面操作就會阻塞執行而拖累較快回應的部分。

React 併發功能就是為了解耦/分離回應快和慢的介面，使渲染慢的部分在背景繼續執行而不阻塞較快的部分，達到每個部分可以各自回應使用者的操作。

併發渲染並不會讓應用程式提升效率變快，只是利用上敘分離的方式讓使用起來感覺體驗變快了。

本文將探討 React 併發功能（React Concurrent），了解其解決的問題以及如何利用。

<!-- more -->

## 遭遇的問題
想像一下，您正開發一個在客戶端執行篩選的篩選列表元件，雖然篩選邏輯本身沒有額外呼叫伺服器，但不管是因為什麼原因，這個篩選邏輯非常耗費 CPU 效能，需要渲染時間。
在這個假設下，我們有兩個主要的 UI 元素，一個輸入欄位，它的值會用來當作篩選條件，另一個就是列表內容。
為了示範這個假設我們提供了下面的範例

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-01-zsi0zt?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-01"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

上面範例的 `<List />` 故意使用 `sleep` 變慢，進而導致同步渲染阻塞。當我們開始輸入篩選條件時，整個 UI 介面會凍結一下下，然後直接變成最後的結果。

理想情況下我們應該可以找方法優化這個元件讓其渲染加速，但有時候優化的部分我們能做的有限，有時候即便優化了渲染還是不如預期。在那些情況下，我們希望那些回應快的部分可以維持，不要被慢的部分拖累。
這個範例只有 `<List />` 是慢的，卻因爲一個元件阻塞讓整個 UI 犧牲變慢，而罪魁禍首就是同步渲染。

## 同步渲染
沒有併發功能時（即沒有 `startTransition` , `useTransition`, `useDeferredValue`)，React 會同步渲染元件，意思是一旦開始渲染，除了例外錯誤，沒有任何事情可以阻止。它只會在完成這個渲染後才繼續其他任務。
實務上這表示不管渲染多久，任何新的事件只會發生在渲染完成之後。

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-02-sihklt?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-02"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

在這個實驗，我們有個 `input` 元件和會延遲 2 秒渲染的 `<Slow />` 元件。`<Slow />` 沒有 `props` 並且有 `memo` 快取，因此當 `input` 變更的時候並不會讓其重新渲染，然後每次我們點擊按鈕取得不同的 `key` 的時候會強制 `<Slow />` 重新渲染。

我們點擊按鈕讓 `<Slow />` 重新渲染並同時在 `input` 嘗試輸入內容。雖然確實可以輸入，但當 JavaScript 忙著渲染元件時，這些互動並不會中斷渲染，而是等 `<Slow />` 渲染完成之後才顯示結果。觀察 Console 的輸出就能證明這些操作的回應是在 `sleep` 之後發生的。
這就是所謂緩慢的元件會阻塞較快的元件，一旦開始渲染緩慢的元件，我們就只能等它們完成才能對其他元件進行更新。

## 解決方案
為了解決這個問題，React 18 引進了所謂的併發渲染。

讓我們從**發動一個更新**開始。在有併發渲染的情況下，一個更新也就是任何會造成進行渲染的行為，例如我們呼叫 `setState` 設定新值就會造成更新

```js
const Component = () => {
  const [count, setCount] = useState(0);
  // 會造成更新，重新渲染
  const handleIncrement = () => {
    setCount((count) => count + 1)
  };
  // 如果傳入一樣的值並不會造成更新
  // 實際上第一次呼叫會更新，但後續就不會了
  const handleSame = () => {
    setCount(count);
  };
  return (
    <>
      <button onClick={handleIncrement}>Increment</button>
      <button onClick={handleSame}>Same</button>
    </>
  );
}
```

接著，我們將所謂的更新區分爲兩種：
1. 高權重（緊急）更新 - High Priority(Urgent)

2. 低權重（非緊急）更新 - Low Priority(Non-Urgent)



產生高權重更新一般是因為呼叫了 `setState`，`useReducer` 的 `dispatch` 或者使用 `useSyncExternalStore` （訂閱的 store 的資料）。也就是產生一次原來一般的同步渲染，這種渲染一旦開始就無法中斷。

此外，值得一提的是當整個應用程式第一次渲染，也就是 `ReactDOM.createRoot`，這個渲染也是高權重更新。



而低權重更新通常由 `startTransition` 或 `useDeferredValue` 觸發，它們會在高權重更新完成之後執行，並且可以被任何高權重的更新中斷。
```js
const Component = () => {
  const [filter, setFilter] = useState('');
  const [delayedFilter, setDelayedFilter] = useState('');
  const [isPending, startTransition] = useTransition();

  const handleInputChanged = (e) => {
    setFilter(e.target.value);
    startTransition(() => {
      setDelayedFilter(e.target.value);
    });
  };
  return (
    <>
      <input value={filter} onChange={handleInputChanged} />
    </>
  )
}
```

當一個低權重的渲染被中斷，它會等中斷它的高權重更新渲染完畢，然後再重新開始渲染。

一個重要的特性，就是它們會和其他相同權重的更新合併，也就是全部的高權重更新會在同一個 call stack 執行，也就只會產生一次渲染。低權重更新也是相同的。

```jsx
const Component = () => {
  const [filter, setFilter] = useState('');
  const [otherFilter, setOtherFilter] = useState('');
  const [delayedFilter, setDelayedFilter] = useState('');
  const [delayedOtherFilter, setDelayedOtherFilter] = useState('');
  const [isPending, startTransition] = useTransition();

  const handleInputChange = (e) => {
    // 下面兩個更新會合併成一次高權重渲染
    setFilter(e.target.value);
    setOtherFilter(e.target.value.toUpperCase());

    startTransition(() => {
      // 下面兩個更新則會合併成一次低權重更新的渲染
      setDelayedFilter(e.target.value);
      setDelayedOtherFilter(e.target.value.toUpperCase());
    })
  };

  return (
    <input filter={filter} onChange={handleInputChange} />
  )
}
```

當回應慢的 UI 操作出現的時候，為了保持回應快的 UI 繼續運作，就把這些更新分成高權重和低權重。

通過這樣的方式，當回應慢的 UI 在處理時，出現了回應快的部分要更新，回應快的更新會中斷回應慢的部分，然後當快速的部分完成後回頭繼續渲染較慢的部分。

## 併發篩選列表
這裡有個一樣的篩選列表範例，但現在我們使用併發功能來將快速與較慢的部分分離
<iframe src="https://codesandbox.io/embed/react-concurrent-demo-03-bv34wn?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-03"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

上面範例有很多需要說明的地方，讓我們從改寫的地方開始。

第一，我們建立了新的 `delayedFilter` 狀態，然後傳入 `<List />` 變成是觸發低權重更新。

第二，當使用者操作 `input` 的時候，我們會觸發高權重更新和一個低權重更新。

然後我們有使用 `memo` 快取了 `<List />` ，現在 `<List />` 不會因為上層元件 `<App />` 渲染更新就更新，只會在 `props` 不同的時候才會重新渲染。

讓我們來進一步解析，當 `<App />` 第一次渲染的時候，這是一個高權重渲染，因為是由 `ReactDOM.createRoot` 觸發的，因此 `filter` 和 `delayedFilter` 都是空字串。然後我們在 `input` 輸入 "tasty"，每次鍵盤輸入時，高權重更新和低權重更新都會觸發。

高權重更新永遠會在低權重更新之前執行，因此在第一鍵按下的時候，第一個高權重觸發，並執行渲染，只有高權重更新的效果會被修改，`filter` 的值變成 t ，但 `delayedFilter` 還是維持空字串。在這個情況下，直到低權重的更新完成，`<List />` 是不會變的。

`delayedFilter` 不會在高權重更新時取得資料，加上 `<List />` 有被塊錢，因此收到一樣的 `delayedFilter` 資料是不會更新的。

一旦高權重更新完成，並且提交變更了 VDOM 之後（Insertion Effect > 變更DOM > Layout > Layout Effect > Paint > Effect 流程結束後）會開始低權重更新，此時 `filter` 的值是 "t"，然後 `delayedFilter` 也取得 "t" 。

但是，如果在低權重渲染的期間，我們輸入的下一個字 "a" 這時也會同時造成兩個更新，一個高權重和一個低權重。

**因為高權重更新發動了，低權重更新會被中斷。**



到此 `filter` 變成 "ta" 但 `delayedFilter` 還是 "" 空值。低權重更新的狀態只有在渲染完成並 commit 時才會發生（關於 commit 可參考資源[React 渲染](https://andyyou.github.io/2020/05/31/understand-react-rendering-in-deep/)一文），與此同時高權重更新看到的這些值還沒變化。

我們可以把低權重更新看作是一份草稿，意思是只要被高權重更新打斷，就暫時不會被提交更新。

當高權重更新渲染時，`delayedFilter` 還是保持一樣的資料，回應慢的 `<List />` 就不會被渲染。

當高權重更新完成後，會在回去處理這些低權重更新，如此循環直到我們停止操作 `input` ，當沒有其他高權重更新發生時候，低權重更新最終會完成渲染。上面這段描述您可以透過觀察 Console 證明了解。

同時您也可能注意到有時候低權重更新甚至沒有發動和中斷，這是因為輸入太快，在低權重更新都還沒開始之前就被高權重更新取消了。

整個流程乍看之下有點困惑，但有個非常適合的比喻可以說明這個概念。

## 併發渲染大略等於版控分支流程

假設我們在開發一個應用程式，通常我們會使用 git 作為版控。當然我們會有一個主線 `main` 代表正式環境版本的程式碼。開發新功能的時候，我們會建立分支，例如 `feature/awesome-feature`，然後當開發工作完成，會合併回主線 `main`。

而當正式環境有某些嚴重的問題時，通常會立刻建立像是 `hotfix/fix-nasty-bug` 的分支，完成之後也會合併回主線。



當我們已經在開發某些功能時，突然需要先處理某個問題。因為修正問題通常會比發佈新功能來的緊急，因此我們會中斷開發，然後先處理問題直到我們解決問題並合併回主線。

只有在問題修正之後，我們才會回到新功能開發，然後需要先將 `main` 的更新先合回新功能的分支。

在這個例子中，一旦我們完成問題修正，我們就可以回去繼續開發新功能直到完成，但很有可能又再一次被其他問題中斷。

您大概意會到了，功能開發就類似低權重更新，可能隨時被一些嚴重需要修正的問題打斷。

同時，在完成問題修正之後，我們回到功能開發之前會需要先 `pull` 那些修正的結果，大致上相當於回到低權重更新時需要重新開始並且把高權重更新的東西也放進去。

如此您應該對於 React 的併發模式更加理解了，後續我們將更深入的解析。

## 併發功能
併發功能發佈時，只有兩種方式可以開啟併發功能，分別是過渡 Transition 和延遲值 Deferred Value。

兩個功能都根據相同的原則運作：我們可以標記特定的更新爲低權重更新，然後就如同我們上面看到的說明。

## Transition (`useTransition`)

Transition API 提供一種命令式的方式讓我們標記更新爲低權重，透過 `startTransitoin`  接收的參數函式，任何在裡面發生的狀態變更都屬於低權重。

我們已經在上面篩選範例看過如何使用 Transition 了。

## 行為細節
還有一些 Transitions API 的行為值得討論：`startTransition` 永遠會觸發高權重更新和低權重更新。
呼叫 `startTransition` ，甚至我們在內部沒有觸發任何更新，它還是會觸發一個高權重更新，沒錯！您沒看錯是一個高權重更新和一個低權重更新。

老實說我不是真的很了解這麼做的目的，但無論如何這個知識點可能在您除錯的時候派上用場。



<iframe src="https://codesandbox.io/embed/react-concurrent-demo-04-umkr23?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-04"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>



`startTransition` 的 callback 會立刻執行，傳入 `startTransition` 的函式也會立刻同步執行。這對於偵錯來說很重要，我們才不會被搞混亂。同時這也意味著我們不應該在這個 callback 裡面執行耗費效能的操作，否則也是會阻塞渲染。

總結來說在 `startTransition` 裡面**觸發耗費效能的更新渲染是可以的**，它們會在背景執行，但不要直接在裡面執行大量計算。



在 `startTransition` 內部更新的狀態必定在同一個呼叫堆疊(call stack)。標記低權重更新，也必須要在同一個堆疊中否則不起作用。（如果這裡您不懂 call stack 可以參考）

- [理解 JavaScript 中的事件循環、堆疊、佇列和併發模式（Learn event loop, stack, queue, and concurrency mode of JavaScript in depth）](https://pjchender.blogspot.com/2017/08/javascript-learn-event-loop-stack-queue.html)
- [理解 Javascript 中的執行環境與堆疊](https://andyyou.github.io/2015/04/18/what-is-the-execution-context-in-javascript/)



意思是下面範例可能不會如您所預期一般作用：

```js
startTransition(() => {
  // 輪到 setTimeout callback 被調用時
  // 已經是在另一個呼叫堆疊了
  // 這個執行會被標記為“高權重更新”
  setTimeout(() => {
    setCount((count) => count + 1, 1000);
  });
});
```

```js
startTransition(async () => {
  await asyncWork();
  // 這裡也是在不同的 call stack
  setCount((count) => count + 1);
});
```

```js
startTransition(() => {
  asyncWork().then(() => {
    // 不同堆疊
    setCount((count) => count + 1);
  })
})
```

如果我們想使用上面這些結構或者說非同步 API ，則我們需要換個組織方式。



```js
setTimeout(() => {
  startTransition(() => {
    setCount((count) => count + 1);
  });
});
```

```js
await asyncWork();

startTransition(() => {
  setCount((count) => count + 1);
});
```

```js
asyncWork().then(() => {
  startTransition(() => {
  	setCount((count) => count + 1);
  });
});
```



**全部 Transition 都會合併在同一次渲染中。** 這其實是我們之前提到低權重更新的特性或者說新功能，多個低權重更新會被合併到一次渲染。然而，這裡我認為有必要在重提一次。

當這些低權重更新被中斷，處於待處理的狀態時，如果另一個 B 低權重更新在第一個 A 低權重更新處理之前被觸發，則所有低權重更新會在同一次渲染執行。



此外，目前還有一個全套用或不套用的規範，意思是即便元件樹狀結構下某個低權重更新已經完成，但其他非相關的元件還在處理更新，“中斷”會讓它們全部重新開始。

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-05-bjtjlb?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-05"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>



**Transition 只能用在狀態上，`ref` 是不行的.**

雖然您可以在 `startTransition` 的 callback 裡面做幾乎所有的操作包含 `ref` ，但只有 `setState` 能標記低權重更新，換句話說只有狀態變更會收到影響。

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-06-iu8xdb?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-06"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

注意到 `filter` 和 `delayedFilter` 保持同時在高權重更新發生變更。

意思是即便您在 `startTransition` 裡面變更 `ref` 但它的行為依舊和你不使用 Transition 一樣。值變了 `<List />` 就會被觸發渲染導致原本問題還是存在。



## 延遲值 `useDeferredValue`

相較於 Transition API 命令式風格的寫法，`useDeferredValue` 屬於宣告式。（Imperatively vs Declarative 可自行查詢）。



`useDeferredValue` 回傳一個低權重更新的狀態結果，且可以把值作為參數傳入。當前值和上一次的值不一樣時會觸發一個低權重更新。

在我們深入探討之前我們先來使用 `useDeferredValue` 處理一樣的篩選列表問題。

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-07-tzy1hb?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-07"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

第一次應用程式渲染時會觸發高權重更新，且第一次呼叫 `useDeferredValue` 時它不會觸發低權重更新，只會回傳初始化的值。所以在第一次渲染 `filter` 和 `deferredFilter` 都是 `""` 空值。



在第一次渲染之後，您可以輸入 "t" 然後因為 `setFilter` 會觸發一個高權重更新。



高權重更新時，`filter` 值會變成 `"t"` ，因此呼叫 `useDeferredValue` 會收到 `"t"`，但它會保持上一次的值 `""` 直到低權重更新的渲染完成。

然後在高權重更新渲染完成之後，會因為 `useDeferredValue` 收到一個不一樣的值觸發一個低權重更新。

在低權重更新渲染，`useDeferredValue` 回傳的值會被設成最後收到的值，而且只有在低權重更新渲染時期收到的值才會被接收。

從這一刻開始它跟 Transition API 範例的工作流程就很類似，您可以通過觀察 Console 證明它們非常相似。

## 行為細節

**在低權重更新渲染期間傳入新值到 `useDeferredValue` 不會再觸發另一個更新。**

如同之前提到的，`useDeferredValue` ，在低權重更新渲染期間會回傳收到最新的值，即便在低權重更新重新渲染期間收到的值和之前高權重更新中接收到的值不一樣。

換個說法，兩次不同的更新 `render` 就是執行 2 次我們元件的程式碼，在高權重更新時，如果 `useDeferredValue` 接到新值會觸發低權重更新，接著在執行低權重更新的時候收到的新值不會在觸發其他更新，而且會直接回傳這次收到的值。

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-09-bddj81?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-09"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

雖然我們在高權重和低權重更新時傳入不同值到 `useDeferredValue` ，因為 `isPending` 只有在高權重渲染時期會是 `true` ，即便如此你會看到一切幾乎沒有變化，重點是在低權重渲染期間收到什麼值，而且不會額外觸發渲染。

不同 `useDeferredValue` 觸發的更新會在一次渲染中合併。這類似於所有的 Transition 也會在一次渲染合併一樣。所以即便在不相干的元件呼叫多次 `useDeferredValue` ，它們的更新還是會合併在一次低權重渲染。



## Suspense

到此我們已經討論了如何使用併發模式處理那些會造成效能瓶頸的元件，然而我們也可以用它們來處理 IO 相關元件。由於 IO 操作在 JavaScript 中可以利用非同步的方式完成，因此即便沒有併發功能，IO 相關的元件也不會造成上面提到的阻塞問題。IO 雖然不會阻塞，但我們等待 IO 的時候通常還是會渲染一個等待的效果。

使用 Suspense 讀取資料時，卻會遇到另一個問題。

現在我們可以不用在 Effect 中讀取資料（也就是完成渲染之後讀取資料），使用 Suspense 我們可以在渲染期間讀取。

我們不再需要等渲染完成才能開始讀取資料，但也不是沒有缺點。例如，如果我們想維持顯示舊資料，直到我們讀取完新資料，而不是永遠顯示載入中。這種情況以前我們會使用 `useEffect` 達成

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-10-zizk9p?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-10"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

這個範例要呈現的效果是，當我們第一次或重新載入頁面渲染時，此時還沒有任何資料，這個時候會顯示“載入中”。然後當我們切換頁面時，我們不會顯示載入中，而是保持顯示舊資料直到新資料載入完成。

但是如果是使用 Suspense 來讀取資料，無論是第一次渲染還是切換頁面，Suspense 都會顯示 `fallback`。

這時如果您希望 Suspense 可以使用這種顯示舊資料的行為，那我們就需要併發功能。

總體的思路是使用低權重更新來變更導致重新載入的狀態，因此當元件處於待處理狀態時是在背景執行，於此同時保持了舊資料在高權重更新渲染。

這裡有兩個範例一個採用 Transition 一個採用延遲值。我們不深入 `suspenseFetchData` 的細節，因為目前沒有穩定版本的 Suspense API 可以用來讀取資料，還有這裡不希望造成讀者困惑。

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-11-9pln7g?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-11"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>
`useDeferredValue` 範例

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-12-u87sp1?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-12"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>



## 其他注意事項

現在我們了解 React 的併發功能是如何運作以及如何運用。不過下面還是有些其他注意事項希望和您分享。

### 暫停點 - Preemption

每當我們有所謂非平行的併發時（即我們的例子，當併發渲染發生在單一執行緒）就需要搶占，也就是說，為了切換任務，我們需要在切換到另一個之前停止當前正在執行的任務。

而允許我們中斷的“地方”，就是我所說的暫停點（請注意，這裡的“暫停”不是 Suspense）。

在不鎖定的多執行緒的環境下，每一行程式碼都可以是一個暫停點，意思是任何時候不管已經執行了什麼，執行緒可以為了讓讓其他執行緒執行而暫停。

而在 React ，暫停點需要在每個元件渲染之間。這段資訊很重要，因為這表示個別元件的渲染不能被中斷，一旦元件開始渲染就會路到達 `return` 。

在到下一個元件渲染之前會一直檢查是否有高權重更新。

<iframe src="https://codesandbox.io/embed/react-concurrent-demo-13-4l6937?fontsize=14&hidenavigation=1&theme=dark&view=editor"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="react-concurrent-demo-13"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

上面範例使用 Transition 支援併發功能的篩選列表，但做了些調整。

這次我們不讓 `<List />` 本身成為耗費效能的元件，而是在裡面渲染新的 `<Slow />`元件，就如其名這個元件就是現在效能的瓶頸。

這裡有兩個重要的事情需要注意：

第一，根據我們中斷低權重更新的速度，我們會在中斷發生之前有不同的 `<Slow />` 需要被渲染。

第二，`<Slow />` 產生的 `console.log` 永遠成對出現，意味著一旦開始渲染 `<Slow />` 我們無法中斷它。

如果我們可以在 `<Slow />` 渲染時期中斷它，我們應該不會看到他們成對出現。

結論就是我們應該避免在單一元件裡有耗費效能的操作，也不應該把它們亂拆成多個元件，拆成多個元件也不能解決問題。

真的無法避免這種情況發生，即便是併發模式也無能為力，因為一旦開始渲染某個回應慢的元件我們也無法觸發其他高權重更新，而之前的範例都是在低權重更新開始渲染之前就中斷它。

小結：我們應該通過低權重更新去更新那些耗效能的元件，延遲它們的渲染。

## 低權重和高權重更新都只有一個階級

雖然針對併發渲染 React 內部有很多順序權重，但只會是高權重或低權重，沒有中間值。

所有高權重更新都是相同權重的，低權重也一樣。

這表示低權重更新，不管來自沒有直接關係的元件，或者來自 Transition 或 Defered Value，他們的權重都是一樣的，也會被合併在同一次渲染。也會遵循全套用或不套用的原則。

```jsx
export default function App() {
  const [filter, setFilter] = useState('');
  const [delayedFilter, setDelayedFilter] = useState('');
  const deferredFilter = useDeferredValue(filter);
  const [isPending, startTransition] = useTransition();

  useDebug({ filter, delayedFilter, deferredFilter });

  return (
  	<div className="container">
    	<input
        value={filter}
        onChange={(e) => {
          setFilter(e.target.value);
          startTransition(() => {
            setDelayedFilter(e.target.value);
          });
        }}
      />
      {isPending && 'Recalculating...'}
      <List filter={delayedFilter} />
    </div>
  );
}

const List = memo(({ filter }) => {
  const filteredList = list.filter((entry) => (
  	entry.name.toLowerCase().includes(filter.toLowerCase())
  ));

  sleep(100);

  return (
    <ul>
      {filteredList.map((item) => (
        <li key={item.id}>
          {item.name} - ${item.price}
        </li>
      ))}
    </ul>
  );
});
```

可以注意到 `delayedFilter` 和 `deferredFilter` 永遠都是同步的，也不會觸發額外的更新。



## 併發模式除錯

替併發元件除錯是非常棘手的事，因為它們會渲染兩次，一次是高權重更新，一次是低權重更新。使用 console 的時候會變得非常混亂，特別是兩個值不一樣。

這個情況下，一個問題自然就出現了，就是我們要如何知道我們是在高權重更新還是低權重更新，又怎麼知道是低權重更新。

好在，有些指標可以協助我們識別：

第一個是觀察 `useDeferredValue` 回傳的值， `useDeferredValue` 在第一次掛載渲染會回傳跟收到一樣的值，但之後，如果我們傳入不同資料它只會在低權重更新渲染時才變更。

因此我們可以

```jsx
// 我們在每次渲染都建立一個新的參考
// 如此 probe 將會跟前一次的資料不同
const probe = {}
const deferredProbe = useDeferredValue(probe);

// 如果不是第一次渲染的話
const isLowPriority = probe === deferredProbe;
```

針對元件的第一次渲染，沒有什麼簡單的方式可以偵測是否是優先更新，因為第一次渲染也可能是低權重更新造成的。

```jsx
const App = () => {
  const [show, setShow] = useState(false);
  const deferredShow = useDeferredValue(show);

  return (
    <>
      <button onClick={() => setShow(true)}>Show</button>
      {/*
        下面元件第一次渲染是基於 deferredShow 這個低權重更新觸發的
      */}
      {deferredShow && <Component />}
    </>
  );
}
```



因為所有的 Effect `useInsertionEffect`，`useLayoutEffect`，`useEffect`只會在渲染階段之後觸發，意思是當它們開始執行的時候，不僅元件將要完成渲染也 commit 了，等於整個元件樹狀結構已經完成渲染了。

有了這些指標，我們可以建置 Hook 來協助我們為併發功能除錯

```jsx
const useDebugConcurrent = ({
  onFirstRenderStart,
  onFirstRenderEnd,
  onLowPriorityStart,
  onLowPriorityEnd,
  onHighPriorityStart,
  onHighPriorityEnd
}) => {
  const probe = {};
  const deferredProbe = useDeferredValue(probe);
  const isFirstRenderRef = useRef(true);
  const isFirstRender = isFirstRenderRef.current;

  const isLowPriority = probe === deferredProbe;

  if (isFirstRender) {
    isFirstRenderRef.current = false;
    onFirstRenderStart?.();
  } else {
    if (isLowPriority) {
      onLowPriorityStart?.();
    } else {
      onHighPriorityStart?();
    }
  }
  useLayoutEffect(() => {
    if (isFirstRender) {
      onFirstRenderEnd?.();
    } else {
      if (isLowPriority) {
        onLowPriorityEnd?.();
      } else {
        onHighPriorityEnd?.();
      }
    }
  });
}
```

上面範例是之前使用 `useDebug` 的改版，差別只是這個 Hook 更適用於一般情況下。此外我們並無法正確精準的偵測中斷，只能知道一個非優先渲染開始和結束不匹配，也無法正確的偵查第一次渲染的優先或非優先。

## 結論

React 的併發功能讓我們可以處理長久以來的問題提升更好的使用者體驗，而本文的介紹是希望您能正確的理解和使用這個功能。



## 資源

- [Everything you need to know about Concurrent React (with a little bit of Suspense)](https://blog.codeminer42.com/everything-you-need-to-know-about-concurrent-react-with-a-little-bit-of-suspense/)

- [React 渲染](https://andyyou.github.io/2020/05/31/understand-react-rendering-in-deep/)
- [理解 JavaScript 中的事件循環、堆疊、佇列和併發模式（Learn event loop, stack, queue, and concurrency mode of JavaScript in depth）](https://pjchender.blogspot.com/2017/08/javascript-learn-event-loop-stack-queue.html)
- [理解 Javascript 中的執行環境與堆疊](https://andyyou.github.io/2015/04/18/what-is-the-execution-context-in-javascript/)
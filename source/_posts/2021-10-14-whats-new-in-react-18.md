---
title: "[譯] 概覽 React 18 新功能"
date: 2021-10-14 16:31:02
tags:
  - react
  - javascript
categories: Program
---

[React 18 alpha](https://github.com/reactwg/react-18/discussions/9) 已經釋出，穩定版可能幾個月後就會跟進。是時候聊聊加入的新功能了。如果您本來就不知道 React 那您可以略過這篇文章。

介紹新功能之前，我們先來看一些您有可能不熟的概念，例如 SSR，Suspense 還有 Hydration。如果您已經知道這些，您可以直接跳到 React 18 的變更一章。

## 深入新功能之前需要了解的概念

### Server Side Rendering

伺服器端渲染 SSR 主要和改善使用者體驗和 SEO 有關，並不是針對改善應用程式的效能。當客戶端對一般 React 應用程式請求頁面時，伺服器會回應一些檔案。這種情境下有兩個很重要的檔案：第一個是一個幾乎為空的 HTML，第二個就是 `bundle.js`。應用程式依據路由在這個空的 HTML 中利用 JS 動態產生內容。這種模式叫做客戶端渲染，因為主要是由客戶端動態渲染。使用者最一開始會看到空白的頁面，接著當 `bundle.js` 下載完畢會動態渲染。

<!-- more -->

而 SSR 讓我們可以在伺服器端就產生 HTML 內容。因此當客戶端發出請求，伺服器會讀取需要的資料，渲染 HTML，接著送出回應。瀏覽器渲染的 HTML 會在伺服器端產生，而不再是幾乎沒有內容的 HTML。

### Hydration

如果應用程式內容很多或您預期使用者的網路很慢，選擇使用 SSR 是合理的。當 `bundle.js` 還在下載時，就算他們點擊的元素還不能使用，使用者至少可以先看到內容。然後當下載完成，事件掛載到 HTML 節點，一切功能就正常了。這個渲染 React 元件的流程，將事件掛載到 SSR 產生的 HTML 的過程就是 Hydration。

### Suspense

雖然上面看起來一切都很巧妙，但在使用 SSR 的時候可能會遇到瓶頸。舉例來說伺服器動態渲染 HTML 必須要等資料讀取完成，這表示如果需要從其他伺服器讀取資料就可能會變慢，而且您必須要等它完成，然後才會開始 Hydration 的處理。由於這非常沒有效率，React 工作小組在 2018 年引進了 [Suspense Component](https://reactjs.org/docs/concurrent-mode-suspense.html) ，該元件僅適用於 Lazy-Loaded 元件。讓使用者在等待非同步操作時，提供一個替代的內容。而它的行為在 React 18 會改變。

## React 18 的變更

### 全新 Root API vs 舊的 Root API

React 應用程式是透過掛載到 DOM 根元素來建立的。如果您使用框架來建置專案，通常可以在 `index.js` 找到關聯 `index.html` 和 `App.jsx` 的程式碼。即 `index.html` 載入 `index.js` ，然後 `index.js` 負責執行掛載的行為。

```jsx
import * as ReactDOM from "react-dom"
import App from "App"

// <App/> 元件會被直接掛載到 id 為 "app" 的 DOM 元素上
ReactDOM.render(<App tab="home" />, document.getElementById("app"))
```

而新的 Root API 使用 `ReactDOM.createRoot()` 

```jsx
import * as ReactDOM from 'react-dom';
import App from 'App';

const root = ReactDOM.createRoot(document.getElementById('app'));

root.render(<App tab="home" />);

// 如果有更新不用再整個 DOM 初始化
root.render(<App tab="profile" />);
```

但為什麼要這麼做呢？其最大的好處在底層。為了使用下面提到改善的功能，您必須使用新的 Root API 而不是舊的 Root API。

### 內建功能優化

這些優化是屬於被動的，意思是一旦您升級到 React 18 並使用新的 API 就會套用。如果您繼續使用舊的 API 則不會有這些新的好處。

### 自動 Batching

Batching 是 React 內部的一個處理機制，許多開發者沒有意識到它。而當您多關注開發者工具的 Console，會發現如果您在同一個事件連續的更新狀態，**React 只會渲染一次**。意思是 React 幫我們把這些操作合在一起，本來**狀態更新**加**重新渲染**要兩次會變成一次。

Batching 是一個很棒的機制，可以防止不必要的重新渲染，但 React 17 只支援在單一事件中合併它們。牽扯到 Promise，async，或其他原生事件則不會觸發這個機制。在 React 18，上述這些狀況 Batching 都會自動完成。

```js
function handleClick() {
  // React 17 會合併
  // React 18 依然保持預設行為
  setIsBirthday(b => !b);
  setAge(a => a + 1);
}

function handleClick() {
  fetchData().then(() => {
    // React 18 會合併，但 17 不會
    setIsBirthday(b => !b);
  	setAge(a => a + 1);
  })
}

setInterval(() => {
  // React 18 會合併，但 17 不會
  setIsBirthday(b => !b);
  setAge(a => a + 1);
}, 5000);

element.addEventListener("click", () => {
  // React 18 會合併，但 17 不會
  setIsBirthday(b => !b)
  setAge(a => a + 1)
})
```

另外，還可以使用 `ReactDOM.flushSync()` 來取消 Batching，但官方不建議頻繁使用它。

```js
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React 已更新 DOM
  flushSync(() => {
    setFlag(f => !f);
  });
  // React 已更新 DOM
}
```

其他更多細節可以[參考](https://github.com/reactwg/react-18/discussions/21)

### 支援元件渲染 undefined

直到 React 18，如果一個 Function 元件回傳 `undefined`，或沒有回傳任何東西。Class 元件的 `render` 方法回傳 `undefined` 或沒有回傳則會觸發錯誤警告；需回傳 JSX 元素或 `null` 。這主要是為了提醒開發者它他們忘記在元件中回傳元素。但 React 開發團隊認為這類型的檢查機制應該歸到 Linter 而不是函式庫內部。因此 React 18 您可以回傳 `undefined`

### SSR 支援 Suspense

之前的版本在伺服器端並不支援 Suspense。新的 `pipeToNodeWritable` API 提供了完整的支援，更多資訊可以[參考](https://github.com/reactwg/react-18/discussions/22)

### Uncaptured Suspense

在 React 17 如果一個元件還沒解析完成，如元件使用 `React.lazy` 載入，這時會找上層最近的 `<Suspense>`，然後渲染它的 `fallback` 直到該元件載入，如果上層沒有任何 `<Suspense>` 就會拋出錯誤。在 React 18 如果沒有 `<Suspense>`，則整個應用程式會暫停，意思是在該元件解析完成之前，什麼都不會渲染。

### Suspense `fallback` 可使用 `null` 或 `undefined`

之前的版本如果 `<Suspense>` 沒有 `fallback` 屬性，則該元件會被忽略並找尋上層下一個 `<Suspense>`。如果都沒有則拋出錯誤。在版本 18 `fallback` 可以是 `null` 或 `undefined` ，意思是不會往上找，就什麼東西都不渲染，直到該 `<Suspense>` 的元件解析完成。

## 併發功能

> 建議可以先閱讀 [併發 CONCURRENCY](https://github.com/reactwg/react-18/discussions/46) 的說明，以對這裡說的併發有些了解。
> 重點節錄：
> 併發的意思是任務可以在同一段時間重疊。
> 讓我們使用打電話來比喻。不支援併發意思是，我一次只能和一個人通電話。如果我打給 Alice，然後 Bob 打給我，我必須要掛掉 Alice 的電話然後才能和 Bob 講話。
> ![](https://user-images.githubusercontent.com/810438/121394782-9be1e380-c949-11eb-87b0-40cd17a1a7b0.png)
> 併發表示我每一在同一段時間有多個通話。例如：我還是保持和 Alice 通話狀態只是將電話擺在旁邊，然後跟 Bob 講話，後續還是可以回來和 Alice 講話。
> ![](https://user-images.githubusercontent.com/810438/121394880-b4ea9480-c949-11eb-989e-06a95edb8e76.png)
> 注意：併發不是說我一次要跟兩個人對話。而在 React 這個打電話（併發任務）指的是 `setState`。

程式中的併發指的是能同時執行多個任務的能力。但由於 React 執行在單執行緒上，因此必須決定執行順序（切換任務）。針對這個問題，React 使用一個 dispatcher 用來註冊 callback。在之前的版本，開發者完全不會碰到這些 API。版本 18 加入了併發功能使其有辦法更有效率的**渲染內容**外加**揭露部分 API**。這些新加入的併發功能支援多工協作，基於權重渲染，排程和中斷，也因此可以大大的改善使用者體驗。

在 16.3 版本加入的 `<StrictMode>` 也得到支援。可以提醒開發者使用併發功能時，如果包含不相容的程式碼可能造成錯誤。但顯然的對整個程式使用 `<StrictMode>`  很容易觸發一大堆警告。因此 React 開發團隊決定開發併發功能而不是併發模式，而您可以在使用併發功能的地方使用`<StrictMode>` 。

雖然 `createRoot` 讓整個應用程式變成了官方所謂的併發模式，但元件依舊可以渲染，除非您在元件中使用了下面提到的併發功能。如果您在某個元件使用了併發功能，那它和它的子元素結構就會套用併發渲染且 `<StrictMode>` 也會啟動相關功能。

### startTransition

在這個版本之前，React 有個非常重要的規則；沒有任何東西可以干擾渲染。一旦狀態變更，重新渲染就會被執行且沒有辦法阻止直到元件渲染完畢。在新版本，現在每個狀態會被分成；立即更新（Urgent Update）或過場更新（Transition Update）。

立即更新即使用者直覺預期會立刻產生回應，例如滑鼠點擊或按鍵盤。而過場更新則是會有一點延遲的動作例如搜尋；表示它們是可能中斷的。過場更新也是同步的，但在它們執行的時候 UI 不會被鎖住。

```js
import { startTransition } from 'react';

// "立即更新" 會直接顯示使用者輸入的資料， UI 會立刻更新渲染
setInputValue(input);

// 使用 startTransition 表示這是一個過場更新
startTransition(() => {
  // 這個變更是可以中斷的
  setSearchQuery(input);
});
```

您可以在這個[討論](https://github.com/reactwg/react-18/discussions/41)找到上面的範例。如果 `setSearchQuery(input)` 沒有被標記為過場更新，則每次 `input` 改變時 UI 會鎖起來。現在利用 `startTransition` 該狀態變更被標註為過場更新，使用者可以搜尋並隨時在介面更新之前改變想法，不用等介面更新。

>  過去這種情況，我們可能要自己使用 `debounce` 來優化介面體驗

```js
import { useState, startTransition } from 'react';

export default function App() {
  const [value, setValue] = useState('');
  
  const onChange = (e) => {
    startTransition(() => {
    	setValue(e.target.value);  
    });
  };
  
  return (
  	<div>
    	<input type="text" value={value} onChange={onChange} />
    </div>
  );
}
```

您甚至可以追蹤過場更新的待辦狀態

```js
import { useTransition } from 'react';
const [isPending, startTransition] = useTransition();

{
  isPending ? <Spinner /> : null
}
```

想更了解併發概念和 `startTransition` 可以參考這篇[圖文說明](https://github.com/reactwg/react-18/discussions/46)。還有[這篇](https://github.com/reactwg/react-18/discussions/65)實務範例

### `useDeferredValue`

`useDeferredValue` Hook 可以讓我們延遲更新部分的 UI，在指定時間內頁面持續可以操作。React 會試著盡快更新延遲狀態。如果在 `timeoutMs` 時間內未能完成就會強制更新，這時 UI 會被鎖起來。換句話說延遲狀態使用的是過場更新，而不是立即更新。

```js
import { useDeferredValue } from 'react';

cosnt deferredValue = useDeferredValue(value, {
  timeoutMs: 5000,
});
```

更多資訊可以參考[官方文件](https://reactjs.org/docs/concurrent-mode-patterns.html#deferring-a-value)

### `<SuspenseList>`

`<SuspenseList>` 支援您調整 `<Suspense>` 節點顯示的順序，就算完成資料取得的順序是不同的。一般，如果您在同一個階層有多個 `<Suspense>` ，它們會各自解析，但如果您希望這些元件照特定順序排列，而不是它們讀取資料或解析完成的時間順序。

```jsx
import { Suspense, SuspenseList } from 'react';

<SuspenseList revealOrder="forwards">
  <Suspense fallback="Loading first item...">
    <FirstItem />
  </Suspense>
  <Suspense fallback="Loading second item...">
    <SecondItem />
  </Suspense>
  <Suspense fallback="Loading third item...">
    <ThirdItem />
  </Suspense>
</SuspenseList>
```

上面的範例，即使第三個元件先處理完了，它還是會顯示 `Loading third item...`，直到前面的項目載入完畢。

`revealOrder`屬性有 `forwards`，`backwards`，`together`。`forwards` `backwards`讓裡面的  `<Suspense>` 照順向或逆向順序顯示。`together`則是等全部好在一起顯示。

另外還有一個 `tail` 屬性，其值支援 `collapsed` 和 `hidden`預設 ``<SuspenseList>` 會渲染所有元件的 `fallback`。如果您不要 `fallback`您可以使用 `hidden`，或者您希望最多渲染一個 `fallback` 可以使用 `collapsed`。

更多資訊可以參考[官方文件](https://reactjs.org/docs/concurrent-mode-patterns.html#suspenselist)

### Selective Hydration 與串流化 HTML 

在我們討論這個概念之前，伺服器端渲染 SSR 包含下面幾個步驟：

1. (伺服器)讀取全部應用程式所需的資料
2. (伺服器)渲染 HTML
3. (客戶端)載入 HTML 和程式邏輯
4. (客戶端)執行 Hydration 

上面的流程除非當前的步驟完成，不然不會進入下一步。在 4 個步驟完成之後，應用程式才可以開始操作。意思是應用程式至少有 4 個可能造成瓶頸的環節。而 React 18 提供兩個主要的功能來解決問題。

##### 在讀取所有資料之前串流化 HTML  

如果您將局部頁面包進 `<Suspense>` 元件，那麼就不會再等這個部分，只要其他元件好了就會繼續。被包起來的元件如果還沒準備好的時候會顯示 `fallback`。一旦資料讀取完成，React 會傳送補充的 HTML 和 JS 到客戶端，並將內容準確的顯示在它該呈現的地方。因此我們不用再等全部資料都讀取完成，這可以解決第一步可能產生的延遲問題。不過要使用這招您的資料讀取函式庫需要實作相關功能。React 提供的 [Server Components](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html) 內建整合 Suspense。

如果您的 `bundle.js`很大的話，第三步載入應用程式邏輯也可能耗費大量時間。為了避免這個問題，您可以實作 Code-Splitting 和 Lazy-Loading。如此就可以依據頁面的需求分批載入邏輯程式。檔案變小，您也不用在一次載入當下用不到的功能可以大大的優化。

##### 全部程式碼載入完成之前執行 Hydration

使用 React 18，將元件使用 `<Suspense>` 包起來，您可以輕易的讓客戶端不再等待該元件。即使缺少局部的 HTML ，程式可以立馬開始 Hydration 的流程。

##### 在全部元件 Hydration 完成之前即可開始操作元件

在 React 18，被包在 `<Suspense>` 的元件 Hydration 流程不會影響使用者和其他已經處理完畢的元件互動。如果被 `<Suspense>` 包起來的元件包含一些 HTML 已經載入的話，就會直接開始處理 Hydration 。但如果使用者在處理期間，操作另外一個也在處理 Hydration 的元件（例如不耐煩的一直點擊）。這個時候就會優先處理那個被使用者一直操作的元件。它也會紀錄事件並在 Hydration 處理完成後調用。這稱為 Selective Hydration。

## 參考資源

* [**what's new in react 18?**](https://yagmurcetintas.com/journal/whats-new-in-react-18)



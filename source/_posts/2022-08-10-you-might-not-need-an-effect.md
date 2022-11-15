---
title: '[譯] React: 您可能不需要一個 Effect'
date: 2022-08-10 14:08:31
tags:
  - react
  - javascript
categories: Program
---

Effect 是讓您可以脫離典型 React 設計模型的一個方式。它讓您可以跳脫到 React 之外並讓您的 React 元件同步外部系統，比如使用非 React 的套件，網路，瀏覽器 DOM 。假如沒有涉及外部，例如您想要在某些 `props` 或 `state` 發生變更時更新元件的 `state` ，**那麼您不應該使用 Effect**。移除不必要的 Effect 可以讓您的程式碼更容易維護，執行效率更好，減少錯誤的產生。

> 本文您將學習
>
> - 為什麼以及如何移除不必要的 Effect
> - 如何快取耗費效能的計算而不使用 Effect
> - 如何重置和調整元件 State 而不使用 Effect
> - 如何在多個事件處理函式共享邏輯
> - 哪些邏輯應該移到事件處理函式中
> - 如何通知上層元件狀態變更

<!-- more -->

## 為什麼以及如何移除不必要的 Effect

有兩種常見的案例，您不應該使用 Effect:

- **您不需要 Effect 來轉換整理渲染呈現需要的資料**。舉例來說，您希望在顯示之前過濾一個列表。您可能會覺得寫一個 Effect 在列表變更的時候更新一個狀態變數。然而這很沒效率。當您更新狀態時，React 會先呼叫您的元件函式來計算應該呈現在畫面的東西。接著 React 會 "commit" 變更到 DOM 更新畫面。接著 React 會執行 Effect，如果 Effect 也同時立刻更新狀態，則會重啟整個流程! 為了避免不必要的渲染，直接在元件頂部轉換資料則可。每當 `props` 或 `state` 變更，那些程式都會自動重新執行。
- **您不需要 Effect 來處理互動事件**。例如您想要發送一個 POST 請求給 `/api/buy` ，然後當完成購買時顯示提示視窗。在購買按鈕的點擊事件，你可以確實知道是使用者點擊了。而 Effect 執行時，你不會知道使用者做了什麼，例如哪個按鈕被點擊。這也是為什麼通常直接在對應的事件處理函式處理對應的互動。

您確實需要 Effect 來同步外部系統。例如您可以使用 Effect 來保持同步 jQuery 套件的資料到 React 狀態。您也可以使用 Effect 來 fetch 資料，例如同步一個查詢的搜尋結果。注意，主流的一些框架支援更有效率的做法來讀取資料，而不是單純使用 Effect。

為了協助您獲得正確的觀念和直覺想法，讓我們看一些常見的具體範例！

### 基於 props 或 state 更新狀態

假設您一個元件有兩個狀態: `firstName` 和 `lastName` 。您希望使用這兩個資料組合出 `fullName` 。同時您希望 `fullName` 會隨著 `firstName` 或 `lastName` 更新而更新。您第一個想法可能是增加一個 `fullName` 狀態然後使用 Effect 更新。

```js
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 避免多餘的狀態和 Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
}
```

這是複雜化需求。同時也沒效率; 導致先渲染了一次 `fullName` 舊資料的狀況，然後又立刻重新渲染。這種情況是不需要多一個狀態變數和 Effect

```js
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // 正確作法: 渲染時會立刻計算新值
  const fullName = firstName + ' ' + lastName;
}
```

當某些需要的資料可以用 `props` 或 `state` 計算出來的時候，不要放到 `state`，直接計算即可。這能優化您的程式碼，避免額外關聯連續的更新，程式碼會更精簡，且減少錯誤的發生。您可以避免一些因為狀態同步造成的 Bug。如果這個方式您感覺很陌生可以參考 [Thinking in React](https://beta.reactjs.org/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state)

### 快取耗效能的計算

元件使用從 `props` 取得的 `todos` 和 `filter` 計算 `visibleTodos`。您可能直覺想要把計算結果放到 `state` 然後利用 Effect 更新。

```js
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 避免多餘的狀態和 Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);
}
```

如同稍早的範例，`state` 和 Effect 都是不必要的。

```js
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // 正確作法, 如果 getFilteredTodos() 計算不複雜直接計算即可
  const visibleTodos = getFilteredTodos(todos, filter);
}
```

在大部分的狀況，上面的程式碼是 ok 的。但 `getFilteredTodos()` 計算可能非常耗效能，或者 `todos` 資料很多。

在這種情況，您不希望因為其他不相關的狀態更新重新計算 `getFilteredTodos()` ，例如 `newTodo` 更新的時候。

此時，您可以使用 `useMemo` 快取耗效能的計算

```js
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // 正確作法
  const visibleTodos = useMemo(() => {
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
}
```

或者單行的形式

```js
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(
    () => getFilteredTodos(todos, fitler),
    [todos, filter]
  );
}
```

這樣 React 就會知道 useMemo 裡面的函式只有在 `todos` 或 `filter` 變更的時候才重新計算。React 會記住 `getFilteredTodos()` 回傳的資料。後續的渲染如果 `todos` 和 `filter` 都一樣則是用上一次記住的資料，如果不同則會在執行一次計算。

包在 `useMemo` 的函式不要包含副作用的操作。

> 如何判斷是否為耗效能的計算?
>
> 一般來說除非您的計算牽扯到上千個物件，否則都在合理範圍。您可以加入 `console` 測量花費的時間。
>
> ```js
> console.time('filter array');
> const visibleTodos = getFilteredTodos(todos, filter);
> console.timeEnd('filter array');
> ```
>
> 執行測量您將會在開發者工具 Console 看到像是 **filter array: 0.15ms** 的訊息，如果紀錄的時間加總很大，例如 1ms 或更久。那麼快取就是合理的。您可以將計算的部分包在 `useMemo` 來驗證互動時的計算時間是否有減少。
>
> ```js
> console.time('filter array');
> const visibleTodos = useMemo(() => {
>   return getFilteredTodos(todos, filter); // Skipped if todos and filter haven't changed
> }, [todos, filter]);
> console.timeEnd('filter array');
> ```
>
> `useMemo` 並不會加速第一次渲染。它只能協助在後續更新時略過不必要的計算。
>
> 注意您的機器可能比使用者的快，因此建議模擬比較差的環境來測試效能。例如 Chrome 有提供 CPU Throttling 功能。
>
> 同時在開發環境測試效能並不能保證準確的結果。例如**Strict Mode 開啟的情況下您會發現每個元件渲染了兩次。**
>
> 為了盡可能準確，您應該建置正式版然後測試。

### 當 props 變更時重新設定狀態

下面 `ProfilePage` 元件可以從 props 取得 `userId`。頁面包含一個留言的輸入框，然後使用了一個 `comment` 狀態變數來儲存資料。某天您注意到一個問題: 當您從 A 個人資料頁到 B 時，`comment` 並沒有重置。其結果造成很容易發錯留言。要修正這個問題您希望當 `userId` 改變時可以清除 `comment`

```js
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 避免在 props 改變時在 Effect 重置狀態
  useEffect(() => {
    setComment('');
  }, [userId]);
}
```

這種處理方式很沒效率，因為 `ProfilePage` 和子元素會先使用舊資料渲染，才又更新(多渲染一次)。要為每個在 `ProfilePage` 有用到狀態的元件都要對應處理會讓事情複雜化。例如；如果有個內嵌的留言介面您希望它的狀態也要因為切換 `userId` 清除。

事實上您可以利用 `key` 來讓 React 知道每個使用者資料元件概念上是不一樣的。

```jsx
export default function ProfilePage({ userId }) {
  return <Profile userId={userId} key={userId} />;
}

function Profile({ userId }) {
  // 元件內的狀態會自動重置
  const [comment, setComment] = useState('');
  // ...
}
```

一般來說在渲染同一個位置的元件時，React 會保留狀態。利用傳入 `userId` 到 `key` 屬性，如果 `userId` 不同 React 會把 `Profile` 視為兩個不同的元件，並且不會共享狀態。

每當 `userId` 改變 React 會重新建立 DOM 和重置狀態。結果就是每當切換使用者資料則 `comment` 也會自動被清除。

注意到這個範例只匯出 `ProfilePage` 。渲染 `ProfilePage` 不需要使用 `key` 。`Profile` 使用 `key` 只是一個實作的細節。

### 當 props 變更時調整狀態

有時候您希望當 props 可以重置或調整局部狀態，但不是全部。

`List` 元件會通過 props 收到 `items` 列表，然後被選取的項目會存放到 `selection` 狀態變數。當 props 收到不一樣的陣列時，您希望設定 `selection` 為 `null`。

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 避免當 props 變更時使用 Effect 調整狀態
  useEffect(() => {
    setSelection(null);
  }, [items]);
}
```

這也一樣不理想。每次 `items` 改變，`List` 和子元素, 元件會用舊的 `selection` 渲染一次。然後執行 Effect 才更新。最後 `setSelection(null)` 會造成另一次重新渲染，然後 `List` 和其他內部元件再次一個循環。

移除 Effect，直接調整狀態即可。

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 比較好的做法是直接調整狀態
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
}
```

紀錄上一次渲染的資料確實有點難以理解，但這比在 Effect 更新狀態的處理方式好。上面範例 `setSelection` 在該次渲染期間直接被執行。React 會在執行到 `return` 時立刻重新渲染 `List` ，此時 React 還沒有渲染 `List` 的子元件和更新 DOM。因此 `List` 的子元件跳過了渲染 `selection` 舊資料的部分。

當您在渲染時期直接更新元件，React 會拋棄回傳的 JSX 並立刻重新嘗試渲染。為了避免緩慢的聯級重試，在渲染期間 React 只讓您更新同一個元件的 `state`。如果您在此元件渲染時期直接更新另外一個元件的狀態，將會看到錯誤訊息。而判斷條件像 `items !== prevItems` 是必要的，為了避免無限迴圈。您可以向上面那樣調整狀態，但任何副作用例如變更 DOM 或使用 timeout 應該維持放在事件處理函式或 Effect 以保證元件的可預測性。

雖然這樣的模式比起 Effect 比較有效率，但大多數的元件應該不需要這麼處理。不管您是如何處理，基於 props 或其他 state 調整狀態會讓您的資料流變的不容易理解。時時檢查看是否您可以利用 key 重置狀態或直接在渲染流程計算。例如取代儲存選取的項目，您可以直接儲存 ID

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);

  // 直接在渲染流程計算
  const selection = items.find((item) => item.id === selectedId) ?? null;
  // ...
}
```

現在我們根本不需要調整狀態。如果清單項目的 ID 被選取，`selection` 會在渲染時期計算，如果沒有選取因為沒有找到匹配的項目則為 `null` 。這個行為跟之前稍微有點不同，但可以說好很多，因為即便 `items` 變更我們還是可以維持正確的 `selection`。但你需要在使用 `selection` 的地方多一些邏輯處理，因為 `selectedId` 可能找不到項目。

### 在事件處理函式間共享邏輯

假如您有一個產品頁面有兩個按鈕 (購買和結帳) 他們都是購買產品的功能。您希望當使用者加入產品到購物車時顯示提示 [toast](https://uxdesign.cc/toasts-or-snack-bars-design-organic-system-notifications-1236f2883023)。

在兩個按鈕的 click 事件加入 `showToast()` 有點重複的感覺，因此您可能想在一個 Effect 處理:

```js
function ProductPage({ product, addToCart }) {
  useEffect(() => {
    // 避免在 Effect 處理特定事件邏輯
    if (product.isInCart) {
      showToast(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
}
```

這邊 Effect 是不必要的。同時也很容易造成 Bug。例如您的應用程式在頁面切換重載時會記住購物車內容。如果您加入一個產品到購物車然後刷新頁面，提示會再次出現。每次您刷新頁面就會出現。因為 `product.isInCart` 是 `true` 然後 Effect 會執行 `showToast()`。

當您不確定程式碼應該要放在 Effect 或事件處理函式中時，問自己這段程式碼執行是為了什麼理由。只有在程式碼是因為元件顯示所以必須對應執行的時候才是用 Effect。上面這個例子，提示視窗是因為使用者點擊按鈕，不是因為頁面載入顯示。刪除 Effect 並將共享的邏輯放到兩個事件處理函式。

```js
function ProductPage({ product, addToCart }) {
  // 特定事件使用的邏輯應從對應事件處理函式呼叫
  function buyProduct() {
    addToCart(product);
    showToast(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
}
```

移除不必要的 Effect 修復 Bug。

### 發送 POST 請求

下面 `Form` 元件會發送兩種類型的 POST 請求。當掛載的時候會發送統計分析的事件。當填完表單並點擊送出的時候，會發送 POST 到 `api/register` :

```js
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // 正確，每次元件顯示的時候需要執行
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 避免特定事件邏輯在 Effect 處理
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
}
```

讓我們套用之前的標準到這個範例。

分析統計的 POST 請求應該維持在 Effect 處理。因為送出請求的理由是因為表單載入顯示就要記錄。但這在開發時期會觸發兩次，可以參考[這裡](https://beta.reactjs.org/learn/synchronizing-with-effects#sending-analytics)解決這個問題。

然而， `/api/register` 請求並不是每次元件顯示就要送出。我們只希望在特定時間發送; 當時用者點擊按鈕時。它應該只發生在特定互動情況。請刪除第二個 Effect 並將邏輯移到事件處理函式

```js
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // 正確，每次元件顯示的時候需要執行
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  });

  function handleSubmit(e) {
    e.preventDefault();
    // 特定事件的邏輯應移至事件處理函式
    post('/api/register', { firstName, lastName });
  }
}
```

當您在抉擇否將邏輯放到事件中或 Effect，您該問的是; 從使用者的角度來看邏輯是什麼。如果是特定操作那就放在事件處理函式。如果是每當使用者看到元件出現在畫面都需要執行則放到 Effect。

### 初始化應用程式

某些邏輯只需要在程式載入的時候執行一次。您可能將它們放到 Effect

```js
function App() {
  // 避免將只執行一次的邏輯單純放到 Effect
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
}
```

然而，您很快會發現在開發時期會執行 2 次。這會造成一些問題 - 舉例來說，這可能造成驗證 Token 無效，因為該函式並沒有設計成可以被執行兩次。一般來說，您的元件應該要保持能被重新掛載的彈性。包含頂層的 `App` 元件。雖然在實務上可能不會被重新掛載，但所有元件遵循相同的規則可以比較容易的組織和重複使用。如果部分邏輯必須在應用程式載入額時候執行一次，而不是每次掛載都執行，您可以在最上層加入變數追蹤其是否被執行:

```js
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // 正確實作每當應用程式載入時只執行一次的邏輯
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
}
```

您也可以在模組初始化階段和應用程式渲染之前執行:

```js
if (typeof window !== 'undefined') {
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {}
```

頂層的程式碼在元件載入的時候會執行一次，就算該元件沒有被渲染也會執行。為了避免匯入元件導致效能變差或非預期的行為，請不要過度使用此模式。將應用程式層級的初始化邏輯應放在 `App.js` 或是進入點的模組。

### 通知上層元件狀態變更

假如您寫了一個 `Toggle` 元件內部包含一個 `isOn` 狀態，它是一個布林值。有不同的方式可以觸發，點擊或拖拉。您可能希望當這個 `Toggle` 內部狀態改變的時候通知上層元件，因此您增加了 `onChange` 事件並在 Effect 呼叫

```js
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 避免, onChange 太晚執行
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange]);

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }
}
```

如之前提到的，這不是理想的處理方式。`Toggle` 會先更新自己的狀態，然後 React 更新畫面。然後 React 執行 Effect，執行 `onChange` 把狀態值交給上層元件。現在上層元件會更新自己的狀態，開始另一回合的渲染。任何時候一次完成所有事情會比較好。

刪除 Effect 並在事件裡同時更新兩者的狀態:

```js
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // 在事件直徑更新狀態
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }
}
```

使用這個方式，`Toggle` 元件和上層元件會在同一個事件更新狀態。React 基於 [batches updates](https://beta.reactjs.org/learn/queueing-a-series-of-state-updates) 的機制一起更新，因此只會渲染一回。

您也許還可以刪除狀態，單純從上層元件接收 `isOn`

```js
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }
}
```

[將狀態往上搬](https://beta.reactjs.org/learn/sharing-state-between-components) 讓上層元件可以利用自己的狀態完整控制 `Toggle`。這意味著上層元件必須包含更多邏輯，但整體來說可以有較少的狀態。每當您需要去維持兩個不同的狀態變數同步時，這就是您應該嘗試將狀態往上搬的訊號。

### 將資料傳入上層

下面範例 `Child` 元件會讀取一些資料然後傳給 `Parent` 元件

```js
function Parent() {
  const [data, setData] = useState(null);

  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();

  // 避免在 Effect 把資料傳入 parent
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
}
```

在 React，資料流是從上層元件流向子元件。當你發現錯誤的時候，您可以沿著元件向上追蹤資料來源，直到找到某個元件傳錯 props 或錯誤的狀態。當子元件在 Effect 更新上層元件的狀態。資料流會變更非常難以追蹤。如果子元件和上層元件都需要相同的資料，建議讓上層元件來讀取資料。

```js
function Parent() {
  const data = useSomeAPI();

  // 較佳: 由上而下傳遞資料
  return <Child data={data} />;
}

function Child() {}
```

如此一來並刻意保持資料流的可預測性; 資料是由上而下傳遞。

### 訂閱外部儲存

有時候，您的元件會需要訂閱一些外部資料。這些資料可能來自第三方函式庫或瀏覽器內建 API。由於這些資料可能在 React 不知情的情況下變更，您會需要手動訂閱。這種情況很常使用 Effect 處理

```js
function useOnlineStatus() {
  // 不夠理想: 在 Effect 手動儲存訂閱
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);

    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);

  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
}
```

這裡元件訂閱一個外部資料，這個範例是瀏覽器的 API `navigator.onLine`。初始狀態為 `true`。每當瀏覽器的資料改變，元件會更新狀態。

雖然使用 Effect 處理這種狀況非常常見，但 React 有專門的 Hook ，在遇到訂閱外部資料的狀況時應優先選擇。刪除 Effect 使用 `useSyncExternalStore` 取代。

> - [useSyncExternalStore 原始碼](https://github.com/facebook/react/blob/6bce0355c3e4bf23c16e82317094230908ee7560/packages/use-sync-external-store/src/useSyncExternalStoreShimClient.js)
> - [官方文件](https://reactjs.org/docs/hooks-reference.html#usesyncexternalstore)

```js
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);

  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // 較佳; 使用內建 Hook 處理訂閱外部資料
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

比起使用 Effect 手動同步資料到 React 狀態，這種方式比較不會發生錯誤。一般來說您會寫一個自訂的 Hook 像上面的 `useOnlineStatus()` ，如此在個別的元件就可以重複使用。更多關於訂閱外部儲存的資訊可以參考官方文件。

### 讀取資料

很多應用程式會使用 Effect 實作 fetch 資料。使用這種方式讀取資料十分常見:

```js
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 避免讀取資料而沒有清除的邏輯
    fetchResults(query, page).then((json) => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
}
```

這邊不需要把 fetch 搬到事件處理函式。

不過這裡似乎和之前的範例有些矛盾，不是說需要基於用戶角度的理由，那這裡應該要放到事件處理函式。考慮到不只是單純打字輸入或換頁才讀取。搜尋條件通常可能預先由 URL 取得，使用者可能回上一頁或下頁操作而沒有打字或切換頁面。不管 `page` 和 `query` 從哪來。當這個元件顯示的時候，您會希望 `results` 會根據當前的 `page` 和 `query` 保持同步網路取得的資料。這就是為什麼這裡使用 Effect。

不過上面的程式碼有 Bug。想像您快速輸入 "hello" 然後 `query` 的變化會是 "h" 到 "he"，"hel"，"hell"，"hello"。每次變更都會觸發讀取，但回應並不會保證照順序返回。例如 "hell" 回應比 "hello" 返回的還慢。因為會執行 `setResults()` 如此將會顯示錯誤的搜尋資料。這就是所謂的 "race condition": 兩個不同的請求相互競爭並且產生跟預期不一樣的順序。

要修正 race condition 的問題，您需要加入清除的功能用來忽略過期的回應

```js
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then((json) => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
}
```

這確保當 Effect 讀取資料時，除了最後的請求，其他的會被忽略。

處理 race condition 並不是讀取資料會遭遇的唯一問題。你也需要思考如何快取回應(如此使用回上一頁會立刻看到之前的畫面，而不是重新讀取)，如何從伺服器讀取(初始化的 server-side 渲染的 HTML 包含讀取的資料)，如何避免連續階層請求(子元件讀取資料不需要等上層讀取完畢)。這些問題任何 UI 函式庫都會遇到，不是只有 React。處理這些問題非常繁瑣，這也是為什麼最新的主流框架提供更有效率內建的讀取機制，而不是直接使用 Effect。

如果您沒有使用框架，可以考慮下面的範例

```js
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
}

function useData(url) {
  const [result, setResult] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then((response) => response.json())
      .then((json) => {
        if (!ignore) {
          setResult(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return result;
}
```

您可能還想加入一些錯誤處理以及追蹤資料是否正確載入的邏輯。可以自訂 Hook 或者使用社群中許多的解決方案。雖然跟框架內建功能相比可能效能差一點，但將讀取邏輯獨立到一個自訂 Hook 後續採用其他策略或增加其他處理機制會比較方便。

一般來說，每當您必須使用 Effect ，留意是否可以將片段功能擷取到一個更具語意自訂的 Hook ，或專門針對某個 API 就像上面 `useData` 一樣。在元件中越少 `useEffect` 會更好維護。

## 概括

- 如果您能在渲染時期計算出值，就不要使用 Effect
- 快取耗費效能的計算可以使用 `useMemo` 而不是 `useEffect`
- 重置元件狀態可以使用 `key`
- 要根據 props 的變更調整狀態，請在渲染期間設定
- 如果程式碼是因為元件顯示就要執行的理由就放在 Effect，其他應該放在對應事件
- 如果您需要更新一些元件的狀態，最好在單一事件處理
- 每當你嘗試同步狀態時，請考慮將狀態往上層搬
- 你可以在 Effect 讀取資料，但記得清除的部分，避免 Race condition

## 參考

- [You Might Not Need an Effect](https://beta.reactjs.org/learn/you-might-not-need-an-effect#how-to-remove-unnecessary-effects)

---
title: "[譯] 詳解 React 渲染"
date: 2020-05-31 20:24:34
categories: Program
tags:
  - react
---

[原文出處](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/?utm_campaign=React%2BNewsletter&utm_medium=email&utm_source=React_Newsletter_213#component-render-optimization-techniques)


本文主旨於介紹 React 渲染的行為，包含使用 Context 和 React-Redux 之後的渲染行為。

我已經見了太多關於渲染的問題並且這些問題仍然持續出現 - 什麼時候，為什麼 React 會重新渲染元件，如果使用了 Context 或 React-Redux 會如何影響重新渲染的範圍和觸發的時機。經歷了多次的解釋，似乎有這個必要將這些回答彙整成一個較完整的介紹。

注意：這些資訊網路上都已經有了，而且很多人都解釋過了，但人們似乎還是蠻困擾因為資訊散落而不容易有個比較完整的了解，所以這篇文章試著將這些問題整理說明清楚，希望能對您有些幫助。

<!-- more -->

## 何謂渲染（Rendering）？

渲染是 React 取得元件“定義您想呈現的介面”並整合當下的 `props` 和 `state` 資訊的流程。

### 渲染流程

渲染流程中，React 會從元件樹狀結構的根開始往下逐一檢查被標記需要渲染的元件。然後被標記的元件 React 會執行 `classComponentInstance.render()` 或 `FunctionComponent()` 最後取得回傳的結果。

元件通常使用 JSX 撰寫，然後會被編譯成純 JavaScript 例如 `React.createElement()` ，`createElement` 會回傳 React elements 它們是純 JavaScript 物件，用來描述 UI 的結構

```jsx
// JSX
return <Component a={42} b="string">Text</Component>

// 編譯為 JS
return React.createElement(Component, {a: 42, b: 'string'}, 'Text');

// 最後回傳 React element
{type: Component, props: {a: 42, b: 'string'}, children: ['Text']}
```

在從元件取得結果之後，React 會比對這個新物件並收集需要對 DOM 產生變更的資訊。這個比對和計算的流程稱為 [Reconciliation](https://reactjs.org/docs/reconciliation.html)。

最後 React 會把計算結果同步套用到 DOM 上。

### Render 和 Commit 階段

概念上整個渲染流程被官方團隊區分為兩個階段：

- Render 包含解析元件計算差異
- Commit 則是真實變更 DOM 的流程

在 Commit 結束之後會同步執行 `componentDidMount` ， `componentDidUpdate` ， `useLayoutEffect` 。接著 React 會設定一個很短的 timeout 來執行 `useEffect` 。

您可以從[示意圖](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)看到 class 元件的生命週期不過它沒有包含 Hook 執行的時間點。

處於實驗階段的新功能 Concurrent Mode，它的功能是它能夠在 Render 階段允許瀏覽器處理事件。React 後續可以恢復繼續執行或中斷，又或者重新計算。一旦完成計算 React 會繼續執行 Commit 流程

關鍵是要理解所謂的渲染不完全等於更新 DOM ，而且元件有可能執行渲染但不對畫面產生任何變化。當 React 渲染元件時：

- 元件有可能回傳相同的結果，所以 React 並不需要變更介面
- 在 Concurrent Mode，React 可能渲染元件很多次，但是拋棄不用直到結果符合為止

## React 是如何處理渲染流程？

### 渲染佇列

初始化渲染完成之後，有幾種方式可以觸發 React 重新渲染

- Class 元件
  - `this.setState()`
  - `this.forceUpdate()`
- Function 元件
  - `useState` 的 setter
  - `useReducer` 的 dispatch
- 其他
  - `ReactDOM.render`

### 標準流程

值得注意的是 React 預設，當上層元件被渲染時，React 會遞迴的渲染所有內部的子元件！

舉例來說：我們有 `A > B > C > D` 元件結構，然後這些元件都已經呈現在介面上了。如果使用者點擊了一個在 `B` 裡面的按鈕變更了一個計數器 `counter` 的狀態：

- 在 `B` 中執行了 `this.setState()` ，會將重新渲染 `B` 的任務加入渲染佇列
- React 開始渲染流程
- 因為 `A` 沒有被標記需要渲染所以跳過
- `B` 被標記需要渲染，所以會重新渲染，然後 `B` 和之前一樣回傳了 `<C />`
- `C` 原本沒有被標記需要渲染，但因為 `B` 重新渲染，React 現在也需要渲染 `C` 以此類推 `D` 也是

渲染元件會造成所有子元件重新渲染！

另外，在預設的渲染流程中，React 不會去管 `props` 有沒有改變，它會無條件的渲染子元件只要上層元件被重新渲染。

意思是每當您在 `<App />` 調用  `setState()` 的時候，就算沒有任何變更，React 會重新渲染整個元件樹。畢竟 React 本來的賣點就是每次更新都像重新繪製整個畫面。

現在很可能元件樹中大多數的元件都回傳了跟上次一樣的結果，因此 React 不會變更 DOM，但 React 依舊必須要執行取得元件狀態，比對差異，計算的流程。這些都需要花一些時間。

記住，渲染不是不好 - 這是 React 得知如何變更 DOM 的方法

## 優化渲染效能

必須得承認的是 - 渲染可能浪費效能也是事實。如果元件渲染之後的結果是不需變更，那麼渲染流程確實是多餘的。

由於 React 元件渲染的輸出永遠都會基於當下的 `props` 和 `state` ，因此如果我們提前得知 `props` 和 `state` 沒有變更的話，我們應該也能確定該次渲染的結果應該是一樣，然後我們就可以跳過該次渲染。

通常我們在優化效能時大概有兩個方向

- 更快的完成相同的任務
- 減少需執行的任務

優化 React 渲染主要屬於第二種方式。

### 優化元件渲染的技巧

React 提供了一些 API 讓我們可以略過渲染

- `React.Component.shouldComponentUpdate`：class 類型元件中一個可選的生命週期方法，該方法會在 `render` 之前被調用。如果回傳 `false` 那麼 React 就會跳過渲染該元件。可以包含任何您想要計算的邏輯，最後回傳一個布林值，大多都是比對上一次的 `props` 和 `state` 如果沒有改變就回傳 `false`
- `React.PureComponent`：因為在 `shouldComponentUpdate` 中比對 `props` 和 `state` 是很常見的方式，`PureComponent` 實作了該行為，可以用來取代 `Component` + `shouldComponentUpdate`
- `React.memo()` ：內建的高階元件。允許我們傳入一個自己撰寫的元件作為參數，回傳一個新的元件。這個元件預設會檢查 `props` 是否有變，如果沒有就不再重新渲染。同時支援 class 和 function 類型的元件。另外它可以傳入一個比對的函式作為第二個參數，但你只能比較 `props` ，所以主要的用途是針對特定欄位。

上面的方式都是使用淺層比對。意思是簡單比對兩個物件的欄位是不是相同，用程式碼來看就是 `obj1.a === obj2.a` ， `===` 比對對 JS 引擎來說非常容易因此上述 3 種方式的比對都等於 `const shouldRender = !shallowEqual(newProps, prevProps)`

另一個比較少被提起的事：如果 React 元件在 `render` 回傳了一個和上一次相同參考（Reference）的輸出， React 會跳過渲染該子元素。上述這些技術都屬於略過渲染，也就是 React 會跳過渲染其內部的子元件，如此就可以停止遞迴去渲染而產生計算的行為。

### 新建立的 Props 參考是如何影響優化

我們知道如果該元件被標記需要重新渲染，那麼預設 React 就會重新渲染其內部的元件即使它們的 `props` 沒有變更。意思是 `props` 是否有變更根本不是重點。

```jsx
function ParentComponent() {
	// 每次渲染會是新的 Reference
	const onClick = () => {
		console.log('clicked');
	};
	
	const data = {a: 1, b: 2};
	
	return <NormalChildComponent onClick={onClick} data={data} />
}
```

每次 `ParentComponent` 重新渲染的時候就會建立新的 `onClick` 和 `data` 參考，然後傳給 `NormalChildComponent` 不管我們使用箭號函式或傳統的 `function` 定義 `onClick` 它都會得到一個新的參考。

也就是說這種時候使用 `React.memo` 是沒有意義的。使用 `React.memo` 要注意傳入 `props` 的參考

```jsx
const MemoizedChildComponent = React.memo(ChildComponent);
function ParentComponent() {
	const onClick = () => {
		console.log('clicked');
	};
	
	const data = {a: 1, b: 2};
	return <MemoizedChildComponent onClick={onClick} data={data} />
}
```

例如上面的範例 `ParentComponent` 每次渲染的時候產生新的參考會導致 `MemoizedChildComponent` 認為 `props` 改變了然後就會重新渲染。

意思是：

- `MemoizedChildComponent` 即使我們想要略過渲染但每次都會被重新渲染
- 上述情況比對新舊 `props` 只是浪費效能

同樣的例如 `<MemoizedChildComponent><OtherComponent /></MemoizedChildComponent>` 它的子元件也是每次都會每次都會重新渲染，因為 `props.children` 永遠都是新的參考。

這個小結的重點在於謹慎正確的使用 React.memo

### 優化 Props 參考

class 類型的元件不太需要擔心為 callback 函式建立新參考的問題，因為他們可以使用物件實例方法，如此一來參考都會是一樣的。然而它們還是可能會需要為列表的項目（子元件）建立獨立的 callback 或者在匿名函式中取得一個值然後傳遞給子元件。這些都會造成產生新的參考。對於這種情況 React 並沒有任何內建的方式優化。

而 function 元件，React 提供了兩個 hook 協助重複使用相同的參考：`useMemo` 可以使用在任何資料，物件（建議使用在複雜計算情境）。`useCallback` 則針對 callback 函式。

### 是否該快取所有東西？

事實上您不需要對每個往下傳遞的函式或物件使用 `useMemo` 和 `useCallback` 。具體的範例可以參考補充一節。

另一個常被提出的問題是為什麼 React 不預設把所有元件使用 `React.memo` 包起來？

Dan Abramov 已經指出快取還是會耗費效能在比對 props 上，並且很多情況下快取無法阻止重新渲染。為每個元件都套用 `React.memo` 可能導致更多問題。

### 測量 React 元件渲染效能

使用 React DevTools Profiler 可以檢查每次 Commit 重新渲染了那些元件。藉此可以找到那些非預期渲染的元件。使用開發工具可以了解為什麼它們重新渲染並且修復問題。

## Context 渲染流程

React Context API 可以協助子元件取得資料。例如任何在 `<MyContext.Provider>` 底下的元件都可以讀取它的資料而不需要逐層通過 `props` 傳遞。

Context 不是狀態管理工具，您還是要自己管理那些傳給 Context 的資料。

### Context 基礎

Context 的 Provider 可以傳入一個 `value` 屬性。例如：`<MyContext.Provider value={42}>` 

然後內部的子元件就可以使用 Consumer 取得 `value` 的資料例如：

```jsx
<MyContext.Consumer>
	{(value) => (
		<div>{value}</div>
	)}
</MyContext.Consumer>
```

或者使用 `useContext` 取得值

```jsx
const value = useContext(MyContext)
```

### 更新 Context

React 會觀察 Provider 的 `value` 是否有更新。如果 Provider 的 `value` 是新的參考 React 發現值產生變更就會讓有使用 Consumer 的元件更新

```jsx
function GrandChildComponent() {
	const value = useContext(MyContext);
	return (
		<div>{value}</div>
	);
}

function ChildComponent() {
	return <GrandChildComponent />
}

function ParentComponent() {
	const [a, setA] = useState(0);
	const [b, setB] = useState('text');
	const value = {
		a,
		b,
	};
	return (
		<MyContext.Provider value={value}>
			<ChildComponent />
		</MyContext.Provider>
	);
}
```

上面範例每次 `ParentComponent` 渲染的時候，React 會註記 `MyContext.Provider` 得到新的 `value` 然後會找到那些有使用 `MyContext` 取值的元件並強制更新。

每個 Context 只有一個 `value` 不管它是物件，陣列或其他格式的資料。目前使用 Context 取值的元件並沒有辦法跳過由 Context 造成的重新渲染。

### State，Context 和重新渲染

現在我們來整理上面學習到的東西

- 調用 `setState()` 會把一次元件重新渲染的任務加入佇列
- React 預設會遞迴的渲染內部元件
- Context Provider 會由使用它的元件來給定 `value`
- 通常 Context Provider 的 `value` 來自上層元件的 `state`

根據我們一開始提到 React 的預設行為會渲染所有子元件（沒有優化的情況下），意思是一旦上層元件的 `state` 變更那麼所有子元件都會被更新。

回到上面 `Parent/Child/GrandChild` 的範例，注意！ `GrandChildComponent` 會重新渲染是因為 `ChildComponent` 重新渲染了，而不是上面因為使用了 `MyContext` 會強制更新。

### 更新 Context 和優化渲染

讓我們修改一下上面的範例

```jsx
function GreatGrandChildComponent() {
	return (<div>Hi</div>);
}

function GrandChildComponent() {
	const value = useContext(MyContext);
	return (
		<div>
			{value.a}
			<GreatGrandChildComponent />
		</div>
	);
}

function ChildComponent() {
	return <GrandChildComponent />
}

const MemoizedChildComponent = React.memo(ChildComponent);

function ParentComponent() {
	const [a, setA] = useState(0);
	const [b, setB] = useState('text');
	const value = {a,b};
	return (
    <MyContext.Provider value={value}>
      <MemoizedChildComponent />
    </MyContext.Provider>
  );
}
```

現在如果我們執行了 `setA(42)`

- `ParentComponent` 會重新渲染
- 新的 Context `value` 參考產生
- React 發現 `MyContext.Provider` 有了新的 `value` 因此任何使用 `MyContext` 的元件都要更新
- React 會嘗試要重新渲染 `MemorizedChildComponent` 不過因為它使用了 `React.memo` 並且沒有 `props` 的變動，所以跳過重新渲染
- 然而因為 `MyContext.Provider` 更新了，也許深層還有元件需要更新
- React 繼續往下檢查，直到 `GrandChildComponent` 它發現有使用 `MyContext` 因此 `GrandChildComponent` 需要被重新渲染
- 接著因為 `GrandChildComponent` 重新渲染了所以 `GreatGrandChildComponent` 也要被重新渲染。

位於 Context Provider 的元件大概都可能需要使用 React.memo - Sophie Alpert

通過這種方式上層元件重新渲染的時候就不用強制渲染所有元件了。或者您也可以利用傳遞相同 `props.children` 參考的方式

```jsx
<MyContext.Provider>
	{props.children}
</MyContext.Provider>
```

來避免重新渲染子元件。

## React-Redux 渲染流程

前陣子最常見的問題大概是各種 Context v.s Redux。這個問題一開始就是錯誤的二分法，因為 Redux 和 Context 處理的是不同的問題。

其中人們最常提到 React-Redux 只會讓需要的元件重新渲染，所以比 Context 好。

這種說法部分是正確的，但答案不僅如此。

### React-Redux 訂閱機制

很多人提到 React-Redux 內部也是使用 Context，技術上來說 - 沒錯，但 React-Redux 使用 Context 是用來傳遞 Store 物件實例，而不是狀態。意思是傳遞給 `<ReactReduxContext.Provider>` 的 `value` 都是一樣的 `store` 參考。

還記得 Redux Store 會在我們 `dispatch` `action` 的時候執行註冊的 callback 函式（加入 subscribe 的函式）。[介面必須訂閱 `store` 然後自行讀取最新的狀態進而觸發重新渲染](https://blog.isquaredsoftware.com/2018/11/react-redux-history-implementation/) 。整個訂閱機制的流程並不屬於 React 內部的機制，只有在 React-Redux 得知特定元件需要的資料發生異動的時候 React 才會加入流程。React-Redux 主要就是基於 `mapState` 或 `useSelector` 來判斷狀態是否發生變更。

這直接導致和 Context  相較之下在效能表現上的差異。觸發重新渲染的元件看起來會比較少。不過當 `store` 的狀態變更時， React-Redux 必須要為整個元件結構執行 `mapState` 或 `useSelector` 函式。雖然大部分情況執行 `mapState` 或 `useSelector` 比起渲染流程耗費的效能比較小，不過如果這些函式需要做一些複雜的轉換或者多餘的更新也是會降低效能。

### `connect` 和 `useSelector` 的差異

`connect` 是一個高階元件。您可以把您的元件作為參數傳入然後它會回傳一個新的元件。這個新元件添加的功能就是訂閱 `store` ，執行 `mapState` 和 `mapDispatch` ，最後把狀態通過 `props` 交給您的元件。

`connect` 產生的元件基本上行為和 `PureComponent` `React.memo` 一樣，不過有些微的差異： `connect` 只有在整合的 `props` 發生變化的時候才會渲染您的元件。

通常最終整合的 `props` 包含 `{...ownProps, ...stateProps, ...dispatchProps}`，所以任何從上層產生 `props` 的新參考都會造成重新渲染，`mapState` 也會，也就是影響重新渲染的範圍變大了。

 `useSelector` 是給函式元件使用的 Hook，因為是在元件內部使用所以不像 `connect` 可以比對阻止上層元件造成的重新渲染。

這就是 `connect` 和 `useSelector` 最大的差異，使用 `connect` 的話元件基本上等於 `PureComponent` 可以停止因為上層重新渲染導致必須要跟著重新渲染的問題。由於大部分的 React-Redux 應用程式會大量使用 `connect` ，意味著其實它們已經降低了渲染次數。

React-Redux 會根據資料的更新來觸發 `connect` 產出的元件進行更新，當然內部的元件可能也會被渲染不過檢查到另外一個 `connect` 元件是不需要渲染的時候就會停止往下影響。

此外，大量使用 `connect` 意味著每個元件可能只關注小部分的資料，因此每次狀態變更，造成重新渲染的機會也會降低。

如果您只使用函式元件和 `useSelector` 。那麼每當 `store` 變更時，比起 `connect` 的方式，很大的機會會重新渲染整個元件樹。

如果因為這樣而產生效能的問題，那麼解決辦法就是使用 `React.memo` 來防止因為上層元件造成多餘的重新渲染

## 總結

- React 預設重新渲染時會遞迴式的影響子元件，所以上層元件重新渲染，內部子元件也會重新渲染
- 渲染機制是 React 得知該如何調整 DOM 的方法
- 渲染是需要耗費時間，如果介面沒改變卻產生了多餘的渲染是會累積的
- 大部分情況傳入 `props` 的函式或物件產生新的參考不會產生問題
- 如果 `props` 沒有變動，`React.memo` 可以避免不必要的渲染
- 如果每次渲染都產生新的參考並傳入 `props` ，那麼 `React.memo` 也沒用，這種情況下就要將傳入的東西暫存起來，例如 `useMemo`
- Context 主要是讓深層的元件可以存取資料，而避免逐層傳遞 `props` 的問題
- `Context.Provider` 的 `value` 比對是基於參考來判斷是否需要更新
- 一旦 `value` 發生異動，底下有讀取 Context 資料的元件會強制重新渲染
- 但大部分的情況造成重新渲染的其實是預設行為，就是上層元件更新導致的
- 所以您應該將 `Context.Provider` 內部的子元件使用 `React.memo` 包起來或使用 `{props.children}` 固定參考的方式，才不會在 Context `value` 更新時造成多餘的渲染
- 子元件如果有讀取 Context `value` ，React 會確保其正確的執行渲染
- React-Redux 通過訂閱 `store` 的方式來判斷是否需要更新，而不是將 `state` 存在 Context
- 訂閱機制會在每次 `store` 更新時運作
- React-Redux 確保了只有在元件使用的資料發生變化才會重新渲染元件
- `connect` 的行為跟 `React.memo` 很接近，因此大量使用 `connect` 可以最小化渲染次數
- `useSelector` 無法阻止因上層元件造成的渲染，所以偏好使用函式元件的開發者應更注意 `React.memo` 的時候時機

## 心得

顯然情況是很複雜的不能單純說 - Context 會造成所有東西重新渲染，Redux 不會，所以應該使用 Redux。雖然我希望大家使用 Redux 但我同時也希望大家可以清楚的明白這些運作機制，然後自己判斷該使用什麼。

由於大家老是在問“什麼時候我該使用 Context？什麼使用我該使用 Redux？那就讓我們進一步彙整一些建議

- 使用 Context
  - 您只是簡單要傳遞資料並且這些資料不會頻繁的變動
  - 如果有些狀態或函式可能貫穿整個應用程式，而且深層的元件也需要存取
  - 您只想使用內建的功能
- 使用 Redux
  - 應用程式有大量的狀態需要處理
  - 狀態很頻繁的變更
  - 狀態變更的邏輯很複雜
  - 多人維護的專案，專案屬於中大型有大量的程式碼

請注意上面提到的都不是硬性的規則，只是簡單的建議。請花點時間根據遭遇的問題，環境自行思考選擇。希望這篇說明可以幫助您對於 React 渲染的行為有更全面的理解

## 參考資源

- [A (Mostly) Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/?utm_campaign=React%2BNewsletter&utm_medium=email&utm_source=React_Newsletter_213#component-render-optimization-techniques)
- [You’re overusing useMemo: Rethinking Hooks memoization](https://blog.logrocket.com/rethinking-hooks-memoization/?from=singlemessage&isappinstalled=0)

## 補充 - 正確使用 useMemo

### 維持相同參考的問題

```jsx
// 範例想避免 value 的參考變化導致 ExpensiveComponent 重新渲染
const MyComponent = ({ page, type }) => {
	const value = useMemo(() => {
		return getValue(page, type);
	}, [page, type]);
	
	return <ExpensiveComponent value={value} />
}
```

根據上面範例，使用 `useMemo` 前請先問自己：

- `getValue` 屬於複雜計算耗費效能嗎？如果不是請不要濫用 `useMemo`
- 不同的參數例如 `page` 和 `type` 會造成 `value` 的參考不同嗎？如果是 `string` `number` `boolean` `null` `undefined` `symbol` 這些參考都不會變動因此 `<ExpensiveComponent>` 也不會重新渲染，也就不需要使用 `useMemo`

### 基於各種理由快取預設狀態

```jsx
const MyComponent = ({page, type}) => {
	const defaultState = useMemo(() => ({
		fetched: operation(),
		type,
	}), [type]);
	const [state, setState] = useState(defaultState);
	return <ExpensiveComponent />
}
```

看似沒有什麼問題的範例，但 `useMemo` 根本不重要。首先我們理解一下範例的意圖，當 `type` 改變時 `defaultState` 可以跟著更新，也可以在每次渲染的時候省去一些計算。

看似合理的想法，但這個方法是錯的並且違反 `useState` 的原則。`useState` 在第一次掛載初始化之後就不會在使用預設狀態。

雖然`type` 改變會得到新的  `defaultState` 但 `useState` 會直接忽略。

```jsx
// 比較好的方式
const MyComponent = ({page, type}) => {
	const defaultState = () => {
    // 複雜計算
    console.log("default state exec");
    return {
      name: "andyyou"
    };
  };
	const [state, setState] = useState(defaultState);
	return <ExpensiveComponent />
}

```

上面這種方式 `defaultState` 只會在第一次掛載的時候被執行一次。

### 處理 ESLint Hook 警告

沒有經過完整理解只為了解決 ESLint 的警告而照著說明調整反而可能產生 Bug 

```jsx
function Example({ tracker, a, b, c }) {
	useEffect(() => {
		tracker(a, b, c)
	}, []);
	
	return <MyComponent a={a} b={b} c={c} />
}
```

上面的範例您不在意  `props` 是否改變，您只想在第一次初始化的時候追蹤 `props` 。而此時 ESLint 會出現一些警告 `React Hook useEffect has missing dependencies: 'tracker', 'a', 'b', and 'c'. Either include them or remove the dependency array.`

這個時候您可以關閉警告

```jsx
// eslint-disable-next-line react-hooks/exhaustive-deps
```

但不是最好的解法，這裡建議您可以使用 `useRef`

```jsx
const initialTrackingValues = useRef({
	tracker,
	params: {
		a,
		b,
		c
	}
});
useEffect(() => {
	const { tracker, params } = initialTrackingValues.current;
	tracker(params)
}, []);
```

注意只有 `useRef` 可以解除這個警告，不要使用 `useMemo` 來處理這個問題，完全是錯的。

### 使用 `useMemo` 處理參考（Reference）的問題

大多人使用 `useMemo` 來快取減少複雜計算和維持參考的問題。第一點減少耗費效能的計算沒有問題。但第二點使用 `useMemo` 維持參考的一致則並不正確。請不要使用 `useMemo` 就只為了讓參考一致。

為什麼不要使用 `useMemo` 維持一致的參考？

```jsx
function Bla() {
	const baz = useMemo(() => [1, 2, 3], []);
	return <Foo baz={baz} />
}
```

上面範例 `Bla` 的值 `baz` 被快取不是因為 `[1, 2, 3]` 是耗費效能的計算，只是因為 `baz` 每次渲染的參考都會改變。

範例看起來沒什麼問題，但 `useMemo` 並不是處理這種情況最佳的 Hook。

上面我們在 `useMemo` 的第二個參數傳入空陣列，也就是 `[1, 2, 3]` 在元件掛載之後就不會改變。結論就是我們快取了一個不會耗費效能的值，這個值在掛載之後就不變了。

如果您發現您遇到類似的情境，請您在好好思考一下是否該使用 `useMemo` 。

那該怎麼做？

首先必須回到我們希望在這裡完成什麼？我們並不是想要快取資料，我們想要的是在每次重新渲染的時候維持一致的參考。畢竟我們使用了 `[]` 作為第二個參數那麼這個變數掛載之後就不會變了。

此時使用 `useRef` 是比較合理的。

```jsx
funtion Bla() {
	const { current: baz } = useRef([1, 2, 3]);
	return <Foo baz={baz} />
}
```

事實上您還可以使用 `useRef` 去保存一個耗費效能的函式計算只要該函式不需要因為 `props` 的改變而需要重新計。

記住，上面的情境是您需要維持一致的參考，但如果您需要保存的是值且會根據 `props` 等改變那您應該使用 `useMemo`
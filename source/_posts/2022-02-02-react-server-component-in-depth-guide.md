---
title: "[譯]React Server Component 如何運作深入解析"
date: 2022-02-02 19:22:32
tags:
  - javascript
  - react
categories: Program
---


> 原文: [How React server components work: an in-depth guide](https://blog.plasmic.app/posts/how-react-server-components-work)
> 推薦參考文章: [React 新概念 — Server Components](https://chentsulin.medium.com/react-%E6%96%B0%E6%A6%82%E5%BF%B5-server-components-d632f9a18463)

React Server Component (RSC)是個令人激動的新功能，不久的將來將會產生很大的影響包括頁面載入效能，打包檔案大小，和開發 React 應用程式的方式。在 Plasmic 開發 [Visual Builder for React](https://plasmic.app/) 時，我們非常關心效能的問題 - 許多客戶建立了行銷和電商相關的網站。因此就算 RSC 還在實驗階段，我們也已經深入探究。在這篇文章，我們會分享目前的研究成果!

<!-- more -->

[toc]

## 何謂 React Server Component

React Server Component 讓伺服器和客戶端(瀏覽器)可以協作渲染 React 應用程式。思考一下典型 React element 樹狀結構通常是由不同的元件組成，裡面又渲染了更多元件。而 RSC 讓我們有可能在伺服器渲染一些元件，另外一些則交給瀏覽器渲染。

下面是 React 團隊的簡易說明，呈現了最終目標；一個 React 樹狀結構，其中橘色元件是由伺服器渲染，藍色元件則是瀏覽器渲染。

![](https://blog.plasmic.app/static/images/react-server-components.png)

### 它是 Server-Side Rendering 嗎?

RSC 不是 Server-Side Rendering。因為他們名字都有 Server 而且也都是在伺服器端工作，所以確實有點讓人困惑。但最好把兩者分開來看，各自的功能相互獨立不可替代。使用 RSC 不需要使用 SSR，反之也是一樣的狀況。SSR 是模擬了環境來渲染 React 樹狀結構輸出 HTML，也就是說元件沒有區分伺服器端和客戶端，元件都是一樣的，渲染的方式也是一樣的。

另外，SSR 和 RSC 是可以一起使用的，如此一來就可以在伺服器端渲染伺服器元件，然後在瀏覽器 "hydrate" 它們的屬性。後續的文章我們會探討更多關於如何一起使用。

### 為何要使用它?

在 RSC 之前，所有的元件都是客戶端元件 - 它們必須在瀏覽器執行。當瀏覽器造訪了 React 的頁面，會下載所有 React 元件需要的程式碼，構建 React element 樹狀結構，然後渲染到 DOM 或者 "hydrate" DOM。瀏覽器擅長處理這些，並讓我們的應用程式可以互動 - 您可以使用事件處理函式(Event Handler)，管理狀態，根據事件變更 React 結構或直接更新 DOM。

如果是這樣那為什麼我們需要在伺服器渲染呢?

這裡提供一些在伺服器端渲染的優點﹔

* 伺服器可以更直接存取資料來源 - 您的資料庫，GraphQL 端點，檔案系統。伺服器可以直接讀取所需的資料而不需要依賴 API，而且通常伺服器端和資料來源會相對緊密，讀取會比瀏覽器快。
* 伺服器使用一些大型函式庫開銷比較小，例如一個將 Markdown 轉為 HTML 的 npm 套件，因為伺服器不需要反覆下載這些相依套件，瀏覽器的話每次要使用時就要下載整包程式

簡單來說，RSC 讓伺服器和瀏覽器可以各自執行擅長的任務。伺服器元件可以專注在讀取資料上，客戶端元件則關注狀態互動達成更快的載入，減少 JavaScript 檔案大小，優化使用者體驗。

## 概覽

讓我們先快速了解一下它是如何運作的。

我的小孩喜歡裝飾杯子蛋糕，但不太會烤蛋糕。讓他們從頭做一個蛋糕再裝飾大概會是一場惡夢。我需要給他們一袋麵粉，糖，奶油。讓他們使用烤箱，閱讀一些教學然後花上一整天。但如果是我來做這些前置作業的話會快很多 - 我先做好杯子蛋糕和糖霜，再交給他們裝飾。這樣他們可以得到樂趣，我也不用擔心他們使用烤箱。

![](https://blog.plasmic.app/static/images/cake.jpg)

RSC 其實就是關於實現這種分工 - 讓伺服器先處理擅長的任務，再交給瀏覽器完成剩下的任務。如此一來伺服器交給瀏覽器的東西就變少了 - 而不是麵粉，奶油，烤箱等等。直接給 12 個杯子蛋糕效率高很多。

回到頁面上的 React 樹狀結構，某些元件可以在伺服器渲染，某些在瀏覽器。這裡提供一個簡化的概念說明﹔伺服器照常只渲染伺服器元件，把元件轉換成 HTML 元素例如 `div`， `p` 等。每當遇到需要在瀏覽器渲染的客戶端元件就輸出一個"佔位"包含"指示" - 如何使用對應的客戶端元件和 `props` 來填補空缺。

然後瀏覽器取得這份"指示"和輸出，填補空缺的客戶端元件。登登! 完成。

**當然這不是完整真實的執行流程**我們很快就會進入那些棘手的細節，但這裡您可以先有個大致上概覽的概念。

## 客戶端-伺服器端元件區分

首先 - 到底什麼是伺服器元件? 到底什麼是給伺服器用的，什麼是給客戶端用的?

React 團隊定義 - 根據附檔名決定該元件屬於哪種類型。如果附檔名是 `.server.jsx` 則為伺服器元件; 如果是 `.client.jsx` 就是客戶端元件。如果都沒有則元件兩者都支援。

如此一來無論開發者還是打包工具都可以清楚的分辨，特別是打包工具，現在它們通過檢查檔名來分別進行不同的處理。很快您就會看到，打包工具對於讓 RSC 運作扮演著非常重要的角色。

因為伺服器元件是在伺服器執行的，客戶端元件在瀏覽器執行，關於各自能做些什麼有很多限制。但記住最重要的就是**客戶端元件不能 import 伺服器元件**，因為伺服器元件不能在瀏覽器執行，這會導致程式碼在瀏覽器無法運作。如果客戶端元件相依伺服器元件，那麼最終會導致我們會把那些不合規範的相依全部加入瀏覽器使用的 bundle。

最後一點可能有點難懂;  這表示像下面這樣的客戶端元件是不合規範的

```jsx
// ClientComponent.client.jsx
// 不可
import ServerComponent from './ServerComponent.server';
export default function ClientComponent() {
  return (
    <div>
      <ServerComponent />
    </div>
  );
}
```

但，如果客戶端元件不能 import 伺服器元件，那我們怎麼辦到像下面圖示那樣的樹狀結構。下圖的伺服器/客戶端元件不是交錯在一起? 我們要如何讓伺服器元件(橘色)掛在客戶端元件(藍色)底下?

![](https://blog.plasmic.app/static/images/react-server-components.png)

雖然您不能在客戶端元件直接 import 和使用伺服器元件，但您可以使用 "composition" 組合的方式，即利用 `props.children` 傳入的方式 - 客戶端元件依然可以從 `props` 取得一個暫時不知道是什麼的 `ReactNode`，而那些 `ReactNode` 可以是伺服器元件。舉例來說

```jsx
// ClientComponent.client.jsx
export default function ClientComponent({ children }) {
  return (
    <div>
      <h1>Hello from client land</h1>
      {children}
    </div>
  );
}

// ServerComponent.server.jsx
export default function ServerComponent() {
  return (
  	<span>Hello from server land</span>
  );
}

// OuterServerComponent.server.jsx
// OuterServerComponent 可以支援客戶端和伺服器元件實例化
// 我們把 <ServerComponent /> 作為 children 傳入
import ClientComponent from './ClientComponent.client';
import ServerComponent from './ServerComponent.server';
export default function OuterServerComponent() {
  return (
    <ClientComponent>
    	<ServerComponent />
    </ClientComponent>
  );
}
```

這個限制將對您如何組織元件發揮 RSC 的功能有著重大的影響。

## RSC 渲染生命週期

接著讓我們深入渲染 RSC 的細節。使用 RSC 不需要了解全部細節，但對其運作有些了解對於您後續開發與幫助。

### 1. 伺服器收到請求

因為伺服器需要處理一些渲染，因此使用 RSC 的頁面生命週期從伺服器開始，為了回應呼叫 API 而渲染 React 元件。

這個根元件一定是伺服器元件，它可以渲染其他伺服器或客戶端元件。伺服器會根據請求和傳入的資料計算伺服器元件和使用的 `props`。這種請求通常是為了取得特定 URL 頁面，Shopify Hydrogen 支援[更細的方法](https://shopify.dev/custom-storefronts/hydrogen/framework/server-state)，React 團隊則提供一個[原始實作](https://github.com/reactjs/server-components-demo/blob/main/server/api.server.js)

### 2. 伺服器序列化根元件元素為 JSON

這一步最終的目標是渲染初始化的 Root 伺服器元件到基本 HTML 標籤的結構中，包含客戶端元件的佔位。然後序列化這個樹狀結構，再送到瀏覽器，接著瀏覽器反序列化，填入客戶端元件，最後渲染結果。

照上面的例子 - 假如我們要渲染 `<OuterServerComponent />` 我們是不是可以就 `JSON.stringify(<OuterServerComponent/>)` 來取得序列化的元素結構?

差不多，但不完全是! 回想一下 React element 實際上就是一個物件有 `type` 欄位，值可以是字串例如 `"div"` 或函式 - React 元件物件實例。

```jsx
> React.createElement('div', { title: 'oh my' })
{
  $$typeof: Symbol(react.element),
  type: "div",
  props: { title: "oh my" },
  ...
}
  
> function MyComponent({ children }) {
  return <div>{children}</div>
}
> React.createElement(MyComponent, { children: 'oh my'});
{
  $$typeof: Symbol(react.element),
  type: MyComponent, // A function
  props: { children: 'oh my' }
  ...
}
```

當您的 React element 是元件時 - 不是基本的 HTML tag - `type` 欄位會是一個函式，函式不能直接序列化成 JSON!

為了正確地將所有內容序列化成 JSON 字串，React 傳入了 [replacer function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#the_replacer_parameter) 來協助 `JSON.stringify()` 正確的處理函式元件，您可以在 [`ReactFlightServer.js` 中找到 `resolveModelToJson()`](https://github.com/facebook/react/blob/42c30e8b122841d7fe72e28e36848a6de1363b0c/packages/react-server/src/ReactFlightServer.js#L368) 就是協助的 `replacer`。

```js
// replacer 函式範例
function replacer(key, value) {
  if (typeof value === 'string') {
    return undefined;
  }
  return value;
}
var foo = {foundation: 'Mozilla', model: 'box', week: 45, transport: 'car', month: 7};
JSON.stringify(foo, replacer);
// '{"week":45,"month":7}'
```

具體來說每當 `resolveModelToJson()` 看到要序列化的 "React element"

* 如果是基本的 HTML 標籤如 `div` 那就是可被序列化的東西，不做任何事
* 如果是伺服器元件，那會執行元件函式，就是存在 `type` 的那個函式，然後序列化。這裡的目標是要將伺服器元件轉成 HTML 標籤
* 如果是客戶端元件，那它也是可被序列化的! `type` 欄位已進指向**模組參考物件**，而不是元件函式。 等等!! 什麼意思!?

#### 什麼是模組參考物件?

RSC 引入了一個新的 `type` 欄位值叫做 "模組參考(Module Reference)"取代元件函式，它是可序列化的參考。

舉例來說，`ClientComponent` React element 看起來可能如下

```jsx
{
  $$typeof: Symbol(react.element),
  type: {
    $$typeof: Symbol(react.module.reference),
    name: 'default',
    filename: './src/ClientComponent.client.js'
  },
  props: { children: 'oh my' }
}
```

但這種處理是什麼時候發生的 - 我們是在哪邊將客戶端函式變成可序列化的參考的?。

是您的打包工具 (Bunlder) 負責處理的! React 團隊針對 Webpack 提供了官方 RSC 支援，發佈了以 [webpack loader](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightWebpackNodeLoader.js) 或 [node-register](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightWebpackNodeRegister.js) 形式的 `react-server-dom-webpack` 。當一個伺服器元件 "import" 了 `*.client.jsx` 檔案，取代實際取得該元件的是換成單純取得模組參考物件並包含檔案名稱和匯出名稱。任何客戶端元件函式都不會在伺服器建構的 React 樹狀結構中。

再看看一下上面的例子，在我們要試圖序列化 `<OuterServerComponent />` 時，我們最終會得到 JSON 概略如下:

```js
{
  $$typeof: Symbol(react.element),
  type: {
    $$typeof: Symbol(react.module.reference),
    name: 'default',
    filename: './src/ClientComponent.client.js'
  },
  props: {
    // 傳入 ClientComponent 的是 <ServerComponent />
    children: {
      // ServerComponent 會直接渲染成 HTML tags
      $$typeof: Symbol(react.element),
      type: 'span',
      props: {
        children: 'Hello from server land'
      }
    }
  }
}
```

#### 可序列化 React 樹狀結構

在這個流程的最後步驟，我們希望在伺服器端得到大概如下的結構然後回送給瀏覽器執行

![](https://blog.plasmic.app/static/images/react-server-components-placeholders.png)

#### 全部的 props 必須可序列化

因為我們要序列化整個 React 結構到 JSON，所有要傳入客戶端元件的 `props` 或 HTML 標籤也必須要序列化。意思是在**伺服器元件您不能傳入事件函式 (Event Handler) 作為 `props`**

```jsx
// 錯誤! 伺服器元件不能傳入函式做為屬性值
function SomeServerComponent() {
  return (
    <button onClick={() => alert('OHHAI')}>
      Click me
    </button>
  );
}
```

但須注意的一點是在 RSC 處理過程中如果遇到客戶端元件，我們是不會調用元件函式或進到下面的客戶端元件。因此如果您有一個客戶端元件裡面還有其他客戶端元件

```jsx
function SomeServerComponent() {
  return (
    <ClientComponent1>
      Hello World!
    </ClientComponent1>
  );
}

function ClientComponent1({ children }) {
  return (
    <ClientComponent2 onChange={() => {}}>
      {children}
    </ClientComponent2>
  )
}
```

`ClientComponent2` 根本不會在 RSC JSON 結構中出現; 我們只會看到 `ClientComponent1` 的模組參考和 `props`。因此在 `ClientComponent1` 裡面傳入事件處理函式給 `ClientComponent2` 完全是沒問題的。

### 3. 瀏覽器重新建構 React 樹狀結構

瀏覽器收到 JSON 並且開始重新構建 React 樹狀結構以便在瀏覽器渲染結果。當遇到 `type` 是模組參考時，我們希望將它換成實際的客戶端元件函式。

再一次這個工作需要打包工具的幫助;  之前是打包工具幫我們在伺服器端將函式替換成模組參考，現在打包工具知道該如何在瀏覽器把參考換成元件函式。

重建的 React 樹狀結構看起來入下

![](https://blog.plasmic.app/static/images/react-server-components-client.png)

然後我們就可以像往常一樣渲染和 "commit" 結構到 DOM。

### 可以搭配 Suspense 使用嗎?

Yes! "Suspense" 在上述所有步驟中都扮演者不可或缺的角色。

我們在本文中故意避開談論 Suspense，因為 Suspense 是個龐大的主題應該獨立介紹。但簡短提一下 - Suspense 讓我們可以在React 元件尚未準備好，例如讀取資料還沒好時，拋出 Promise。這些 Promise 會在 "Suspense boundary" 被攔截 - 每當一個 Promise 從 `<Suspense>` 底下被拋出，React 會暫停渲染直到 Promise 完成，然後繼續工作。

當我們在伺服器端呼叫伺服器元件函式以產生 RSC 輸出時，當這些函式需要讀取資料，可能會拋出 Promise。如果遇到這種情況就會輸出一個佔位，直到 Promise 完成，會在調用一次伺服器元件函式，如果成功則輸出完整的程式碼片段。事實上我們在建立 RSC 串流時，遇到 Promise 會暫停，解析完成則繼續補上額外的程式碼。

同樣的，在瀏覽器，我們以串流的方式 `fetch()` RSC JSON。這個流程也一樣，如果有 Suspense 佔位且串流中沒有相關內容就拋出 Promise。或者是模組參考但元件還沒下載好也一樣。

多虧了 Suspense，您可以在伺服器元件讀取資料時以串流的方式處理 RSC 輸出，然後瀏覽器可依逐步渲染取得的資料並動態讀取需要的客戶端元件。

## RSC 傳輸格式

那到底伺服器輸出了什麼? 如果您在讀到上面 "JSON" 和 "串流"的時候覺得困惑，您的疑惑是對的! 那到底伺服器是怎麼以串流的方式把資料傳到瀏覽器?

是個簡單的格式，一個 JSON blob 一行搭配標記 ID。下面是 `<OuterServerComponent />` 的 RSC 輸出範例

```
M1: {"id": "./src/ClientComponent.client.js", "chunks": ["client1"], "name":""}
J0: ["$", "@1", null, {"children": ["$", "span", null, {"children": "Hello from server land"}]}]
```

 上面片段，`M` 開頭的行，定義了客戶端元件模組參考，包含了可以在 bundle 搜尋元件函式的資訊。`J` 開頭定義的是 React 元素結構，`@1` 則指向由  `M` 定義的客戶端元件。

這個格式具備支援串流特性 - 只要客戶端讀取一行，就可以解析 JSON 執行對應更新。如果伺服器在渲染時遇到 Suspense boundary，您就會看到多個 `J` 對應 "解析完成之後，會取得的程式碼資訊"。

例如: 

```jsx
// Tweets.server.js
import { fetch } from 'react-fetch';
import Tweet from './Tweet.client';
export default function Tweets() {
  const tweets = fetch('/tweets').json();
  
  return (
    <ul>
      {tweets.slice(0, 2).map((tweet) => (
        <li>
          <Tweet tweet={tweet} />
        </li>
      ))}
    </ul>
  );
}

// Tweet.client.js
export default function Tweet({ tweet }) {
  return (
    <div
      onClick={() => alert(`Written by ${tweet.username}`)}  
    >
      {tweet.body}
    </div>
  );
}

// OuterServerComponent.server.js
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
      <Suspense fallback={'Loading tweets'}>
        <Tweets />
      </Suspense>
    </ClientComponent>
  );
}
```



RSC 串流資料看起來如下:

```
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}
S2:"react.suspense"
J0:["$","@1",null,{"children":[["$","span",null,{"children":"Hello from server land"}],["$","$2",null,{"fallback":"Loading tweets...","children":"@3"}]]}]
M4:{"id":"./src/Tweet.client.js","chunks":["client8"],"name":""}
J3:["$","ul",null,{"children":[["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}],["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}]]}]
```

`J0` 現在多了一個額外的子階層 - `Suspense` 的部分，它的 `children` 指向 `@3`。有趣的是 `@3` 現在還沒定義。當伺服器完成讀取推文(`tweets`) ，就會輸出 `M4` 其定義了模組參考 `Tweet.client.js` 元件。然後 `J3` 定義了應該要被放到 `@3` 位置的東西。接著`J3` 的 `children` 執行 `M4` 即 `Tweet` 元件。

這裡另一個要注意的是，打包工具自動將 `ClientComponent` 和 `Tweet` 分成兩個分開的檔案，讓瀏覽器可以延遲載入 `Tweet` 。

### 使用 RSC 格式資料

那瀏覽器要怎麼把這個串流格式的資料變成實際的 React element? `react-server-dom-webpack` 包含了處理 RSC 回應並重建的功能。下面是簡化版的 "Root client component"

```jsx
import { createFromFetch } from 'react-server-dom-webpack';

function ClientRootComponent() {
  const response = createFromFetch(fetch('/rsc?...'));
  return (
    <Suspense
      fallback={null}
    >
      {response.readRoot()}
    </Suspense>
  );
}
```

我們使用 `react-server-dom-webpack` 來讀取從 API 取回的 RSC 回應。然後 `response.readRoot()` 會回傳 React element 並且隨著串流的資料更新。在取得任何資料之前會拋出 Promise - 因為還沒有內容準備好。然後當它處理第一個  `J0`，會建立對應的 React element 結構，處理 Promise ，React 恢復渲染流程，但當遇到還沒準備好的 `@3` 參考，另一個 Promise 會拋出。一旦 Promise 完成，讀取 `J3` ，React 會再次恢復渲染。因此當我們使用串流形式的回應，我們可以持續更新渲染元素直到全部完成。

### 為何不直接輸出 HTML

為何要發明一個全新的傳輸格式? 客戶端的目標是重新建構 React 元素。比起解析 HTML 來建立元素使用這種方式會相對簡單，並且這樣我們對後續變更可以 "commit" 最小的 DOM 變更。

### 這真的比單純從客戶端元件讀取資料好嗎? 

反正我們都需要發送 API 請求到伺服器來讀取資料，這樣真的比我們請求只讀取資料，然後全部在客戶端渲染好嗎?

這取決於您需要渲染什麼到畫面上。使用 RSC，您可以優化讀取效能，預先處理要顯示的資料。如果您只需要渲染一小部分資料，需要下載大量的 JavaScript 又或者多種資料讀取互相依賴需要一個接一個讀取，這些情況在伺服器上直接處理會快很多。

### 那麼關於 Server-Side Rendering 呢?

我知道! 我知道! React 18 您是可以 SSR 和 RSC 一起使用，在伺服器端產生 HTML，在瀏覽器 "hydrate" RSC。後續會有更多關於這個題目的探討。

## 更新渲染的伺服器元件

那如果我需要更新伺服器元件呢? 舉例來說，如果我切換檢視頁面從 A 產品頁到 B 產品頁?

由於渲染發生在伺服器，這會發出另一個 API 請求並取得新的 RSC 傳輸資料。好消息是，一旦瀏覽器取得新內容它可以執行一般的差異比對然後只做最小的 DOM 變動，並且狀態和事件函式都會在。對客戶端元件來說，這個更新跟完全在瀏覽器發生的情況沒有不同。

目前，您必須從伺服器根元件來重新渲染整個 React 樹狀結構，但未來，是可能可以從階層下的節點渲染。

## 為什麼需要為了 RSC 採用 Meta-Framework (Next.js / Shopify Hydrogen)

React 團隊表示 RSC 最初的意思是建議通過 Meta-Framework 來使用，而不是直接使用。但為什麼要這樣? Meta-Framework 可以為我們提供什麼?

你不一定要這麼做，但使用 Meta-Framework 會輕鬆很多。通常 Meta-Framework 會提供友善的抽象，你就不用思考如何在伺服器產生 RSC 傳輸資料，還有要怎麼在瀏覽器使用。

另外，Meta-Framework 也支援 SSR，同時他們做了很多工作確保如果您使用 RSC 後伺服器產生的 HTML 可以被正確的 "hydrate"。

如您所見，您還需要處理打包工具的設定才能在瀏覽器正確的使用客戶端元件。目前已經有 webpack 的支援，Shopify 正在開發 Vite 的整合。這些套件目前都須變成 React 檔案庫的一部分，因為 RSC 需要的很多東西都還沒公開。不過一旦開發完成，後續應該是可以在不用 Meta-Framework 的情況下使用。

## RSC 可以使用了嗎?

RSC 目前您可以在 Next.js 實驗版本使用，還有 Shopify Hydrogen 開發版，但不適合在正式產品。後續的部落格文章會深入探討這些框架如何使用 RSC。

不過毫無疑問，RSC 將會成為 React 未來重要的一部分。這是 React 的答案 - 提供更快的頁面載入，減少 JavaScript bundle，縮短互動時間。至於如何使用 React 構建多頁面應用程式，更全面的探討，可能還沒有準備好，但很快就會開始關注了。 

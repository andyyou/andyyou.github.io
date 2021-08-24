---
title: React DnD 入門
tags:
  - react
  - javascript
categories: Program
date: 2021-03-07 13:56:21
---

[toc]

React DnD 是 React 一系列的單元功能，用來協助您建置介面上複雜的拖拉功能同時維持元件解耦。特別適合像是 Trello 類型的應用程式利用拖拉來轉移資料，並依據拖拉事件變更的狀態改變元件的外觀。


## 安裝

```sh
$ npm i react-dnd react-dnd-html5-backend
```

<!-- more -->

第二個套件是支援 React DnD 底層使用 [HTML 5 拖拉 API](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Drag_and_drop)。您可以選擇使用第三方的套件例如 [touch backend](https://npmjs.com/package/react-dnd-touch-backend)

## 範例概覽

```jsx
// 讓 <Card text="Write the docs" /> 可以拖拉

import React from 'react';
import { useDrag } from 'react-dnd';
import { ItemTypes } from './Constants';

// 您的元件

export default function Card({ isDragging, text }) {
  const [{ opacity }, dragRef] = useDrag(() => ({
    item: { type: ItemTypes.CARD, text },
    collect: (monitor) => ({
      opacity: monitor.isDragging() ? 0.5 : 1,
    }),
  }), []);
  
  return (
  	<div ref={dragRef} style={{ opacity }}>
    	{text}
   	</div>
  );
}
```

## 功能

### 整合您的元件

比起現成的元件，React DnD 封裝您的元件並注入相關 `props` 。如果您曾使用 React Router 或 Flummox 您已經了解這種模式。

### 單向資料流

React DnD 完全支援 React 陳述式且不直接改變 DOM 的渲染流程。透過單向資料在 Redux 以及其他架構都非常好擴充。實際上它就是用 Redux 建置的。

### 消除平台特殊行為

HTML 5 拖拉 API 處於一個尷尬的狀況，每個瀏覽器行為不一致。React DnD 底層為您處理了這個問題。您可以專心在開發您的應用而不是處理瀏覽器造成的問題。

### 可擴充性和測試性

React DnD 底層使用 HTML 5 拖拉 API 但也允許您自訂“後端”（後續我們使用後端代表處理拖拉的底層機制）。您可以客製化基於觸控事件的 DnD 後端，甚至是其他事件。

舉例來說，內建模擬的後端，可以在 Node 環境下測試元件的拖拉操作。

### 支援觸控

可以使用 [touch 後端](https://npmjs.com/package/react-dnd-touch-backend) 支援觸控拖拉。

### 非特定目標

React DnD 提供一組原始語法，但不包含任何具備完整功能的元件。比 jQuery UI 和 Interact.js 還低階，專注在正確的執行拖拉功能，視覺方面則交由您自行處理。例如，React DnD  沒有提供排序元件，而是讓您自行使用這些功能組合。



# 概覽

React DnD 不像市面上其他的拖拉功能的函式庫，而且在還沒用過之前，乍看之下使用方式可能沒那麼方便。不過一旦您了解其設計概念，就會覺得這些設計非常合理。建議在開始閱讀其他文件之前線閱讀這些觀念。

其中一些概念類似於 Flux 和 Redux 架構。不是巧合，因為 React DnD 內部就是使用 Redux。

### 項目和類型（Item & Type）

就像 Flux 或 Redux 一樣，React DnD 使用資料，而不是視圖作為唯一來源。當您拖動螢幕上某個物件，我們不說元件或 DOM 節點被拖動，而是特定類型（Type）的項目（Item）被拖動了。

那什麼是項目 Item？一個項目是一單純的 JavaScript 物件，描述被拖動的東西。舉例來說看板類型的應用程式，當我們拖動一張卡片的時候項目 `item` 大概就是 `{cardId: 42}`。在西洋棋裡，當你拿起一個棋子 `item` 可能是 `{ fromCell: 'C5', piece: 'queen' }`。使用物件的形式描述被拖移的資料，讓我們可以保持元件解耦。

Type 類型又是什麼？類型是一個字串或 [Symbol](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) ，是一個唯一識別。例如在一個看板應用程式中，您可能有一個 `card` 類型代表可以被拖動的卡片們和一個 `list` 類型可拖動的列表。在西洋棋的例子可能就只有一個 `piece` 類型。

類型的功用是，隨著應用程式持續成長，您可能會有越來越多可以拖拉的東西，但您不希望某些可拖入的區域接受非預期的項目。類型讓我們可以限制拖移來源和拖入目標。您可能有一個類型的列舉型別常數，就像 Redux 的 action types 一樣。

### Monitor

拖放本質上是有狀態的。不管是不是正處於拖移中，或者是否存在目前拖移的項目、類型，這些狀態資料必須要存在某個地方。

React DnD 使用容器的方式將內部狀態通過 `monitor` 傳給元件。`monitor` 讓我們可以更新元件的 `props`  ，讓元件呈現拖拉狀態的改變。

每個元件都需要追蹤拖放的狀態，您可以定義一個狀態收集函式並使用 `monitor` 取得相關資料。然後 React DnD  會即時調用狀態收集函式並將回傳的值合併到元件的 `props` 中。

假設我們要凸顯被拖拉的西洋棋格。`Cell` 元件使用的狀態收集函式可能長的如下：

```js
function collect(monitor) {
  return {
    highlighted: monitor.canDrop(),
    hovered: monitor.isOver(),
  };
}
```

該函式會指示 React DnD 傳入 `highlighted` 和 `hovered` 更新的資料到 `Cell` 的 `props`。

### 連接器

如果後端處理的是 DOM 事件，但元件是使用 React 來定義描述 DOM 的，那麼後端該如何得知 DOM 節點的監聽。答案是連接器。連接器讓我們預先在 `render` 函式定義分配角色（拖移來源、拖拉預覽、拖入區域）到 DOM 節點。

實際上，連接器會被當作第一個參數傳入我們上面說的狀態收集函式。讓我們來看看我們可以如何指定拖入區域：

```js
function collect (connect, monitor) {
  return {
    highlighted: monitor.canDrop(),
    hovered: monitor.isOver(),
    connectDropTarget: connect.dropTarget(),
  };
}
```

在這個元件的 `render` 方法，我們可以存取從 `monitor` 取得的資料和取得 `connector` 的函式

```jsx
render() {
  const {
    highlighted,
    hovered,
    connectDropTarget,
  } = this.props;
  
  return connectDropTarget(
  	<div className={classSet({
    	'Cell': true,
      'Cell--highlighted': highlighted,
      'Cell--hovered': hovered,
    })}>
      {this.props.children}
   	</div>
  );
} 
```

執行 `connectDropTarget` 會通知 React DnD 該元件的根節點是有效的**拖入目標**，然後它的 `hover` 和 `drop` 事件應該被後端處理。內部的運作機制是利用您給的 [callback ref](https://reactjs.org/docs/refs-and-the-dom.html#callback-refs) 附加到 React 元素來完成。`connector` 回傳的函式會被存起來，因此 `shouldComponentUpdate` 不會破壞它。

### 拖移來源和拖入目標（Drag Sources & Drop Targets）

目前為止我們已經涵蓋了後端；其用來處理 DOM 相關，使用項目和類型來代表資料，狀態收集函式包含 `monitor` 和 `connect` 讓我們可以設定一些屬性，React DnD 會幫我們注入元件。

但我們要怎麼設定元件才能取得注入的屬性呢？我們該如何對應拖拉事件執行 Side Effect 的操作？是時候來看 React DnD 主要的抽象單元；拖移來源和拖入目標。它們將類型、項目、Side Effect 操作，狀態收集函式和您的元件結合在一起。

每當您希望某個元件或元件的部分支援拖拉時。您需要將元件包進拖移來源。每個拖移來源會註冊為特定類型，並且從元件的 `props` 實作產生項目的方法。還可以選擇性的指定一些方法來處理拖拉的事件。這個拖移來源宣告也可以為元件設定狀態收集函式。

拖入目標跟拖移來源非常類似。唯一的差別是一個拖入目標可以註冊多個類型，然後它可以處理 `hover` 或 `drop` 事件。

### 後端

React DnD 使用 HTML 拖拉 API。很合理使用它作為預設，因為它內建可以為拖移的 DOM 節點截圖作為拖移時預覽的效果。您不用在滑鼠移動時額外繪製圖像。也是唯一提供處理拖入檔案的 API 。

不幸的是，HTML5 的拖拉 API 有些缺陷。它不支援觸控螢幕，並且 IE 瀏覽器的支援度比不上其他瀏覽器。

這也是為什麼 React DnD 使用外掛套件的方式實作 HTML5 拖拉 API 。您不一定要使用，您完全可以自訂自己的實作，基於觸控事件 ，滑鼠事件或其他。這些外掛的實作在 React DnD 中稱為後端（Backend）。

函式庫目前搭載 HTML 後端，它應該可以滿足大部分網頁應用程式。[觸控後端](https://react-dnd.github.io/react-dnd/docs/backend/touch) 可以支援行動裝置上的網頁程式。

後端會執行類似 React 的合成事件；抽離瀏覽器的差異並處理原生 DOM 事件。儘管有類似的地方，React DnD 後端並不相依於 React 或其合成事件系統。底層，所有的後段會轉換 DOM 事件到內部的 Redux actions 然後 React DnD 接著後續的處理。

### Hooks 和高階元件

現在您已經了解 React DnD 各個方面

* 項目物件和類型
* Flux 架構的 DnD 狀態
* `monitor` 觀察 DnD 狀態
* 狀態收集函式可以將 `monitor` 的資訊合併進 `props`
* 連接器把 DnD 狀態掛到 DOM 節點

現在該來討論如何將這些結合到元件；您有兩個選擇 Hook 形式的 API 和傳統裝飾器模式的 API （高階元件）

#### Hooks

目前主流的 React 應用程式會採用 Hook 取代高階元件。Hook 是 React 16.8 的功能之一，主要是支援函式元件可以使用 `state` 。它也非常適合管理具備狀態的元件和外部狀態進行互動（例如拖移）。

如果您還不熟悉 React Hooks 您可以參考官方文件。

React DnD 提供 Hook 連接您的元件並使用 `monitor` 提供狀態，渲染。



# 練習教學

在本篇教學我們將使用 React 和 React DnD 建置一個西洋棋遊戲。哈！開玩笑的，完整的西洋棋遊戲已經超出本教學太多。我們只會建置一個簡單的西洋棋盤搭配一個可以被拖移的騎士。

如果你妳已經熟悉 React 您可以直接跳到後面加入拖拉互動一段

我們將使用這個範例展示 `react-dnd` 資料驅動的核心哲學。您會學到如何建立拖移來源和拖入目標。

## 設定

本節教學使用函式元件和當前主流語法。由於這些語法需要為目標環境使用例如 Webpack，Rollup 或其他編譯工具才能使用，推薦直接使用 [create-react-app](https://github.com/facebook/create-react-app) 。

```sh
$ npx create-react-app demo
```



## 建置遊戲

### 設計元件

首先，我們從 React 元件開始，這個階段我們不加入拖拉的概念。對這個只有一個騎士的西洋棋遊戲，思考一下針對我們的需求元件大概有：

* `Kngith`：唯一的騎士棋子
* `Square`：棋盤上的格子
* `Board`：64 格的棋盤

接著思考屬性部分

* `Knight` 大概不需要任何屬性。它的確需要一個位置，但不需要在 `Knight` 保存位置的資料，我們可以讓它作為 `Square` 的子元素就好。
* 乍看之下 `Square`  好像需要位置資料，不過再想想，格子好像只需要知道渲染的顏色就好。我們先假設 `Square` 預設是白色，加入 `black` 布林屬性。當然 `Square` 可以加入子元素，就是擺在上面的棋子。
* `Board` 稍微麻煩一點。我們知道棋盤的格子是固定的，因此直接在 `Board` 寫死格子，而不需要動態傳入 `props.children` 。但接著棋格還需要包住 `Knight`，這表示 `Board` 需要知道 `Knight` 的位置。`Board` 需要的資料包含棋格顏色，全部棋子位置的資料，但這個範例來說只需要一個 `knightPosition` 騎士的位置。我們可以使用二維陣列 `[0, 0]` 表示 `A8` 。為什麼不是 `A1`？為了符合瀏覽器的座標系統。其他方式會大量增加複雜度。

![](http://i.imgur.com/OhWKkEm.png)



那目前的狀態要存在哪？直接放在 `Board` 元件中不太理想。元件盡可能減少自有的狀態會比較好，因為 `Board` 已經包含一些配置棋格的邏輯，如果把全部的邏輯管理放在這邊會變得很冗余。

這個階段我們還不需要處理這個問題，我們的元件只需要假設某個地方會有資料透過 `props` 傳入，我們只須確保渲染的結果是正確的即可。

### 建置元件

個人傾向由下而上建置元件，由於大部分面對的都不是全新的專案。如果先處理 `Board` 那至少就得等到 `Square` 做好才能看到結果。反過來說如果先建置 `Square`，我們可以立刻看到 `Square` 的結果不用管 `Board` 。我覺得可以立馬看到成果挺重要的。

但實際上，我們是從 `Knight` 開始。

```js
// src/Knight.js
import React from 'react';

const Knight = () => {
  return (
    <span>♘</span>
  );
};

export default Knight;
```

沒錯，我們使用 Unicode 的 ♘ 。好處是可以使用 `color` 屬性方便調整顏色，但這個範例我們只有一隻騎士所以不需要考慮不同陣營顏色的問題。

```js
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';

import './styles.css';
import './components/Knight';

ReactDOM.render(
  <Knight />,
  document.getElementById('root')
);
```

執行 `npm start` 可查看結果。開發元件的時候我都會確認結果，如果是在大型專案則會使用 [cosmos](https://github.com/skidding/cosmos) 。如此我們就不用等到一堆元件都寫完才來排除錯誤。

接著，我們來建置 `Square`

```js
// src/Square.js
import React from 'react';

const Square = ({ black }) => {
  const fill = black ? 'black' : 'white';
  return (
    <div
      style={{
        backgroundColor: fill,
      }}
    />
  )
};

export default Square;
```

然後在 `index.js` 改變進入點讓 `Square` 包住 `Kngith` 

```jsx
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';

import Knight from './Knight';
import Square from './Square';

ReactDOM.render(
  <Square black>
    <Knight />
  </Square>,
  document.getElementById('root')
);

```

什麼都沒出現！不好意思上面犯了一些錯誤：

* `Square` 沒有給任何寬高，整個縮起來。除此之外我們希望格子可以自動填滿容器因此設定 `100%`
* `Square` 忘記加 `children` 因此就算 `Knight` 傳入了也不會顯示。

就算我們修正了上面列的還是看不到 `Knight` 因為 `Square` 是黑色的，然後預設文字也是黑色的。因此我們要為 `Knight` 設定文字顏色的屬性。

```jsx
// src/Square.js
import React from 'react';

const Square = ({ black, children }) => {
  const fill = black ? 'black' : 'white';
  const stroke = black ? 'white' : 'black';
  const style = {
    backgroundColor: fill,
    color: stroke,
    width: '100%',
    height: '100%',
  };

  return (
    <div style={style}>
      {children}
    </div>
  )
};

export default Square;
```

![](http://i.imgur.com/jvgv6DV.png)

> 如果遇到高無法撐開，可以設定 `html, body` 的樣式，或給一個外層容器。官方範例的外層有包一個 500x500px 的容器。

終於我們可以來建立 `Board` 元件，我們先從超級簡化的版本開始：

```jsx
// src/Board.js
import React from 'react';
import Square from './Square';
import Knight from './Knight';

const Board = () => {
  return (
    <div>
      <Square black>
        <Knight />
      </Square>
    </div>
  );
};

export default Board;
```

這邊的目的只是讓 `Board` 可以先被渲染出來方便我們後續調整：

```jsx
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import Board from './Board';

ReactDOM.render(
  <Board knightPosition={[0, 0]} />,
  document.getElementById('root')
);
```

到此，我們依舊看到一樣的結果。是時候加入整個棋盤了。但我們該從何開始？我們該怎麼組織 `render` 的內容？使用 `for` 或 `map` 嗎？

實際上，我們不會立馬討論這個問題。我們已經知道如何輸出一個棋格不管有沒有包含棋子。我們也知道騎士的位置 `knightPosition` ，這代表我們可以著手 `renderSquare` 方法，現在還不用擔心整個棋盤的問題。

第一版的 `renderSquare` 大概如下：

```jsx
const renderSquare = (x, y, [knightX, knightY]) => {
  const black = (x + y) % 2 === 1;
  const isKnightHere = knightX === x && knightY === y;
  const piece = isKnightHere ? <Knight /> : null;
  
  return (
  	<Square black={black}>
      {piece}
    </Square>
  )
};
```

我們可以調整 `Board`

```js
import React from 'react';
import Square from './Square';
import Knight from './Knight';

const renderSquare = (x, y, [knightX, knightY]) => {
  const black = (x + y) % 2 === 1;
  const isKnightHere = knightX === x && knightY === y;
  const piece = isKnightHere ? <Knight /> : null;
  
  return (
    <Square black={black}>
      {piece}
    </Square>
  );
};

const Board = ({ knightPosition }) => {
  return (
    <div style={{ width: '100%', height: '100%' }}>
      {renderSquare(0, 0, knightPosition)}
      {renderSquare(1, 0, knightPosition)}
      {renderSquare(2, 0, knightPosition)}
    </div>
  );
};

export default Board;
```

棋格直接往下排是因為我們沒有設定佈局。這裡我們使用 Flexbox，我們在 `renderSquare` 加入 `div` 和樣式。一般來說元件封裝需要考慮彈性，而不是把佈局加入元件中。因此這裡我們補在 `renderSquare` 多加一個 `div`。

調整後的 `Board` 和 `index.js` 如下

```jsx
// src/Board.js
import React from 'react';
import Square from './Square';
import Knight from './Knight';

const renderSquare = (i, [knightX, knightY]) => {
  const x = i % 8;
  const y = Math.floor(i / 8);
  const isKnightHere = x === knightX && y === knightY;
  const black = (x + y) % 2 === 1;
  const piece = isKnightHere ? <Knight /> : null;

  return (
    <div key={i} style={{ width: '12.5%', height: '12.5%' }}>
      <Square black={black}>{piece}</Square>
    </div>
  );
};

const Board = ({ knightPosition }) => {
  const squares = Array.from({ length: 64 }).map((_, i) => renderSquare(i, knightPosition));
  
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        display: 'flex',
        flexWrap: 'wrap',
      }}
    >
      {squares}
    </div>
  );
};

export default Board;
```

```js
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import Board from './Board';

ReactDOM.render(
  <div style={{ width: 500, height: 500 }}>
    <Board knightPosition={[0, 0]} />
  </div>,
  document.getElementById('root')
);

```

現在您可以看到完整的棋盤了 LOL。現在我們可以透過調整 `knightPosition` 來改變騎士的位置。

### 加入狀態

我們希望 `Knight` 可以被拖移。為了實現這個目標 `knightPosition` 需要某種儲存形式保留狀態並且要能被修改。

因為管理狀態會需要你思考一些問題，我們不會在同時一併實作拖拉功能。因此我們從簡單的部分開始；透過點擊特定 `Square` 然後移動 `Knight` ，不過騎士還是要遵循西洋棋的規則，不可以亂走。實作這個邏輯已經能夠讓我們充分理解狀態管理，後續我們再將點擊的操作換成拖拉。

React 對於狀態管理並沒有限制，您可以使用 Flux，Redux，Rx 等並且[避免大量資料模型分散注意力](https://martinfowler.com/bliki/CQRS.html)

不過在這個範例為了單純期間我們不希望加入 Redux，但可以遵循一些簡單的設計模式，當然它無法像 Redux 一樣具備良好的可擴展性，不過在這個範例沒什麼問題。現在我們還沒決定狀態管理的 API，就先假設狀態都放在 `Game` 模組裡，隨後我們需要在裡面定義一些修改狀態資料的程式碼。

基於上面的假設，我們可以在 `index.js` 預寫一些還不存在，但我們希望的用法。我們可以使用這種方式來釐清我們所需的 API。

```jsx
// src/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import Board from './Board';
import { observe } from './Game';

const root = document.getElementById('root');

observe((knightPosition) => 
  ReactDOM.render(
    <div style={{ width: 500, height: 500 }}>
      <Board knightPosition={knightPosition} />
    </div>,
    root
  )
);

```

`observe` 函式極簡化了模擬訂閱狀態的設計模式。我們當然可以使用 `EventEmitter` 模擬訂閱機制，不過這裡只需要一個變更的事件不需要將事件變得複雜。

為了驗證這個訂閱 API 可行。我們來實作一隨機移動騎士位置

```js
// src/Game.js
export const observe = (cb) => {
  const getRandPosition = () => Math.floor(Math.random() * 8);
  setInterval(() => cb([getRandPosition(), getRandPosition()]), 500);
};
```

![](https://s3.amazonaws.com/f.cl.ly/items/1K0s0n0r0C0e2P2N2D1d/Screen%20Recording%202015-05-15%20at%2012.06%20pm.gif)



顯然這個東西沒什麼用。我們需要的是和使用者互動，我們需要一個從元件中修改狀態的方法。接下來，我們要實作一個簡單的 `moveKnight` 函式，讓它可以直接修改內部狀態。這裡得先聲明上面的作法在中等複雜的應用程式中效果就不好了，對於不同的操作應應該只更新對應的狀態，不過這個教學範例還行。

```js
// src/Game.js

let knightPosition = [0, 0];
let observer = null;

export const emitChange = () => {
  observer(knightPosition);
};

export const observe = (o) => {
  if (observer) {
    throw new Error('Multiple observers not implemented');
  }
  observer = o;
  emitChange();
};

export const moveKnight = (toX, toY) => {
  knightPosition = [toX, toY];
  emitChange();
};
```

現在，我們可以回到我們的元件。現階段的目標是當我們點擊某個 `Square`，`Knight` 要移動到那。其中一個方式是將事件放在 `Square` 身上然後它來呼叫 `moveKnight`。不過需要傳入 `Square` 的位置。這裡提供一個不錯的開發建議：

> 如果元件渲染時不需要某個資料，則它根本不需要。

`Square` 渲染的時候根本不需要知道位置的資料，因此最好避免將 `moveKinght` 交給 `Square`，我們將在 `Board` 中使用一個 `div` 綁定 `onClick` 來包住 `Square`：

```js
// src/Board.js
import React from 'react';
import Square from './Square';
import Knight from './Knight';
import { moveKnight } from './Game';

const handleSquareClick = (toX, toY) => {
  moveKnight(toX, toY);
};

const renderSquare = (i, [knightX, knightY]) => {
  const x = i % 8;
  const y = Math.floor(i / 8);
  const isKnightHere = x === knightX && y === knightY;
  const black = (x + y) % 2 === 1;
  const piece = isKnightHere ? <Knight /> : null;

  return (
    <div
      key={i}
      style={{ width: '12.5%', height: '12.5%' }}
      onClick={() => handleSquareClick(x, y)}
    >
      <Square black={black}>{piece}</Square>
    </div>
  );
};

const Board = ({ knightPosition }) => {
  const squares = Array.from({ length: 64 }).map((_, i) => renderSquare(i, knightPosition));
  
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        display: 'flex',
        flexWrap: 'wrap',
      }}
    >
      {squares}
    </div>
  );
};

export default Board;
```

當然我們也是可以在 `Square` 上面加入 `onClick`，但因為無論如何後面要加入拖拉功能的時候還是得把 `onClick` 事件移除，所以我們就不這麼做了。

現在我們只差把西洋棋騎士移動的規則加入。簡單說騎士只允許走 L 形；因此我們在 `Game` 補上 `canMoveKnight` 函式，並且調整初始化的位置到 `B1`

```js
// src/Game.js
export const canMoveKnight = (toX, toY) => {
  const [x, y] = knightPosition;
  const dx = toX - x;
  const dy = toY - y;

  return (
    (Math.abs(dx) === 2 && Math.abs(dy) === 1) ||
    (Math.abs(dx) === 1 && Math.abs(dy) === 2)
  );
};
```

然後在 `Board` 的 `handleSquareClick` 使用 `canMoveKnight`

```jsx
// src/Board.js
// ...
import { moveKnight, canMoveKnight } from './Game';

// ...
const handleSquareClick = (toX, toY) => {
  if (canMoveKnight(toX, toY)) {
    moveKnight(toX, toY);
  }
};
```

## 加入拖拉功能

這是本段教學最核心的一節。我們現在可以來見識一下 React DnD 如何讓拖拉互動的功能變得非常容易整合到既有的元件中。這個段落假設您已經閱讀完概覽一節，對於像是後端，狀態收集函式，類型，項目，拖移來源，拖入目標等詞有了基本的了解。

首先，我們需要安裝 React DnD 和後端的部分：

```sh
$ npm i react-dnd react-dnd-html5-backend
```

未來，您可以自行使用第三方的後端，但這超出了本教學的範圍。

### 設定拖拉的執行環境

第一件事情，我們需要為應用程式設定 `DndProvider` 。這個元件盡可能掛在整個結構的上層。然後設定後端

```jsx
import React from 'react';
import { DndProvider } from 'react-dnd';
import { HTML5Backend } from 'react-dnd-html5-backend';

// ...

const Board = ({ knightPosition }) => {
  // ...
  
  return (
    <DndProvider backend={HTML5Backend}>
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          flexWrap: 'wrap',
        }}
      >
        {squares}
      </div>
    </DndProvider>
  );
};

export default Board;
```

### 定義拖拉的項目類型

接著，我們要建立拖拉類型的常數。我們只有一個類型，`KNIGHT`，但我們還是建立一個 `Constants` 檔案來管理類型常數。

```js
// src/Constants.js
export const ItemTypes = {
  KNIGHT: 'knight',
};
```

### 為騎士加入拖移功能

`useDrag` hook 可以傳入函式參數然後回傳一個物件（暫存資料）。這個物件的 `item.type` 設定了我們定義的項目類型，接著可以定義狀態收集函式：

```js
const [{ isDragging }, drag] = useDrag(() => ({
  item: {
    type: ItemTypes.KNIGHT,
    collect: (monitor) => ({
      isDragging: monitor.isDragging(),
    }),
  },
}));
```

讓我們來解析 `useDrag` 的用法：

* `useDrag` 可以傳入指定的物件或工廠函式。`item.type` 是必須的，指定類型的項目才能被拖移。我們也可以加入其他的資訊例如完整西洋棋遊戲的話我們可以加入棋子的資料，是城堡或主教等，但因為這只是教學範例我們只需要 `type` 就夠了。
* `collect` 定義狀態收集函式；基本上這就是我們及那個拖拉的狀態轉換合進元件 `props` 的方法。
* 回傳的陣列資料包含
  * 陣列第一個資料是 `props` 物件，這裡包含我們從拖拉系統收集到的資料例如 `isDragging` 
  * 參考函式為第二個值，用來掛載到 DOM 元素以加入 React DnD 的功能。

現在，讓我們來看看如何在 `Knight`中使用：

```jsx
// src/Knight.js

import React from 'react';
import { useDrag } from 'react-dnd';
import { ItemTypes } from './Constants';

const Knight = () => {
  const [{ isDragging }, drag] = useDrag(() => ({
    item: {
      type: ItemTypes.KNIGHT,
    },
    collect: (monitor) => ({
      isDragging: monitor.isDragging(),
    }),
  }));

  return (
    <div
      ref={drag}
      style={{
        opacity: isDragging ? 0.5 : 1,
        fontSize: 25,
        fontWeight: 'bold',
        cursor: 'move',
      }}
    >
      ♘
    </div>
  );
};

export default Knight;
```

### 為棋格加入置放的功能

`Knight` 現在可以被拖移了，但不能放到其他格子上。因此這節我們要讓 `Square` 變成拖入目標。

此時，我們無法避免將位置傳入 `Square` ，畢竟如果 `Square` 不知道自己的位置，其實是要怎麼移動到上面。但還是覺得很奇怪因為 `Square`  在程式中並不會因為這些資料發生改變。當您面對這種問題的時候，建議參考如何拆分[容器與顯示元件](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)。

這裡我們要加入一個新元件叫 `BoardSquare`。它負責渲染我們已經有的 `Square` 但它負責處理位置和 `Board` 的 `renderSquare` 中局部的邏輯。

```jsx
// src/BoardSquare.js
import React from 'react';
import Square from './Square';

const BoardSquare = ({ x, y, children }) => {
  const black = (x + y) % 2 === 1;
  return (
    <Square black={black}>
      {children}
    </Square>
  );
};

export default BoardSquare;
```

然後調整 `Board`

```jsx
// src/Board.js
import React from 'react';
import { DndProvider } from 'react-dnd';
import { HTML5Backend } from 'react-dnd-html5-backend';
import Knight from './Knight';
import BoardSquare from './BoardSquare';
import { moveKnight, canMoveKnight } from './Game';

const handleSquareClick = (toX, toY) => {
  if (canMoveKnight(toX, toY)) {
    moveKnight(toX, toY);
  }
};

const renderPiece = (x, y, [knightX, knightY]) => {
  if (x === knightX && y === knightY) {
    return (<Knight />);
  }
};

const renderSquare = (i, knightPosition) => {
  const x = i % 8;
  const y = Math.floor(i / 8);

  return (
    <div
      key={i}
      style={{ width: '12.5%', height: '12.5%' }}
      onClick={() => handleSquareClick(x, y)}
    >
      <BoardSquare x={x} y={y}>
        {renderPiece(x, y, knightPosition)}
      </BoardSquare>
    </div>
  );
};

const Board = ({ knightPosition }) => {
  const squares = Array.from({ length: 64 }).map((_, i) => renderSquare(i, knightPosition));
  
  return (
    <DndProvider backend={HTML5Backend}>
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          flexWrap: 'wrap',
        }}
      >
        {squares}
      </div>
    </DndProvider>
  );
};

export default Board;
```

現在我們可以在 `BoardSquare` 加入 `useDrop` hook。我們的拖入目標只處理 `drop` 事件：

```js
const [, drop] = useDrop(() => ({
  accept: ItemTypes.KNIGHT,
  drop: () => moveKnight(x, y),
}), [x, y])
```

`drop` 方法可以取得 `BoardSquare` 範圍內 `props` 的資料，因此當騎士移入格子的時候就知道移動到哪了。

在實務上我們可能會從 `beginDrag` 方法使用 `monitor.getItem()` 來讀取拖移來源的資料，但這裡只有一個拖移的項目因此不需要。

在下面的狀態收集函式中，我們會從 `monitor` 取得目前游標在那個 `BoardSquare` 上面，我們就可以修改樣式。

```js
const [{ isOver }, drop] = useDrop(() => ({
  accept: ItemTypes.KNIGHT,
  drop: () => moveKnight(x, y),
  collect: (monitor) => ({
    isOver: monitor.isOver(),
  }),
}), [x, y])
```

現在的成果看起來還不錯！但還有一個功能還沒完成。就是我們希望 `BoardSquare` 可以顯示符合規則的移動位置，並且只有在符合規則的時候棋子才可以被移動。

因為 React DnD 的關係要實現這個功能非常容易我們只需要定義 `canDrop` 方法

```js
const [{ isOver, canDrop }, drop] = useDrop(() => ({
  accept: ItemTypes.KNIGHT,
  canDrop: () => canMoveKnight(x, y),
  drop: () => moveKnight(x, y),
  collect: (monitor) => ({
    isOver: monitor.isOver(),
    canDrop: monitor.canDrop(),
  }),
}), [x, y]);
```

同時我們也加入 `monitor.canDrop()` 到狀態收集函式。然後加入一個 `Overlay` 元件和調整 `BoardSquare` 

```jsx
// src/Overlay.js
import React from 'react';

const Overlay = ({ color }) => {
  return (
    <div
      style={{
        position: 'absolute',
        top: 0,
        left: 0,
        height: '100%',
        width: '100%',
        zIndex: 1,
        opacity: 0.5,
        backgroundColor: color,
      }}
    />
  );
};

export default Overlay;
```

```jsx
// src/BoardSquare.js

import React from 'react';
import { useDrop } from 'react-dnd';
import Overlay from './Overlay';
import Square from './Square';
import { canMoveKnight, moveKnight } from './Game';
import { ItemTypes } from './Constants';

const BoardSquare = ({ x, y, children }) => {
  const black = (x + y) % 2 === 1;

  const [{ isOver, canDrop }, drop] = useDrop(() => ({
    accept: ItemTypes.KNIGHT,
    canDrop: () => canMoveKnight(x, y),
    drop: () => moveKnight(x, y),
    collect: (monitor) => ({
      isOver: monitor.isOver(),
      canDrop: monitor.canDrop(),
    }),
  }), [x, y]);

  return (
    <div
      ref={drop}
      style={{
        position: 'relative',
        width: '100%',
        height: '100%',
      }}
    >
      <Square black={black}>
        {children}
      </Square>
      {isOver && !canDrop && <Overlay color="red" />}
      {!isOver && canDrop && <Overlay color="yellow" />}
      {isOver && canDrop && <Overlay color="green" />}
    </div>
  );
};

export default BoardSquare;
```



![](https://s3.amazonaws.com/f.cl.ly/items/0X3c342g0i3u100p1o18/Screen%20Recording%202015-05-15%20at%2002.05%20pm.gif)



### 加入拖移預覽

最後一件事就是我們希望加入拖移時預覽畫面。沒錯，瀏覽器會擷取 DOM 的畫面，但如果您想顯示自訂的圖片呢？

幸運的是 React DnD 可以很容易完成這個功能。我們只需要加入 `preview` 參考

```js
const [{ isDragging }, drag, preview] = useDrag(() => ({
  item: { type: ItemTypes.KNIGHT },
  collect: (monitor) => ({
    isDragging: monitor.isDragging(),
  }),
}));
```

`preview` 讓我們可以在 `render` 設定一個 `dragPreview` ，類似我們在拖移項目使用的一樣。`react-dnd` 支援 `DragPreviewImage` 元件我們可以在拖移的時候透過參考顯示一個預覽。

```js
return (
  <>
    <DragPreviewImage connect={preview} src="https://i.imgur.com/cPpanFj.png" />
    <div
      ref={drag}
      style={{
        opacity: isDragging ? 0.5 : 1,
        fontSize: 25,
        fontWeight: 'bold',
        cursor: 'move',
      }}
    >
      ♘
    </div>
  </>
);
```

### 總結

這章教學帶我們過了一次如何建立元件，決策狀態設計最後加入拖拉功能。目的是向您介紹 React DnD 符合 React 的理念，加上您應該在深入實作複雜互動之前，先考慮應用程序結構。
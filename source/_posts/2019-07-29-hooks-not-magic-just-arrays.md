---
title: React Hooks 不是黑魔法，只是陣列
date: 2019-07-29 17:25:12
categories: Program
tags:
  - react
  - javascript
---

本文旨在提供一些關於 Hooks API 一些乍看之下奇怪的規範以及理解這些規範的見解。

<!-- more -->

## 理解 Hook 的運作機制

我聽到有些人正在為了理解 Hooks 的黑魔法以及提案奮鬥，因此我試著解開語法提案是如何運作的。



### Hooks 規範

React 核心團隊在 Hooks 的提案文件上提出了兩個主要是使用規範開發者必須要遵守

* 不可以在迴圈，條件式或巢狀函式中調用 Hook
* 只能在 React Function 中調用

第二點是比較容易理解的。要掛載行為到一個函式型元件（Functional Component），您需要以某種形式將行為和元件關聯。



### Hooks 中狀態管理都跟陣列有關

為了得到清晰的觀念，讓我們看看簡單的 Hooks API 實作。

> 請注意這只是推測其中一種可能實作 API 的方式，主要是為了呈現思路。這並不是 API 內部運作完全等價的東西。



### 如何實作 useState() ?

讓我們解析範例來展示一個 State Hook 的實作可能的運作

首先，我們從元件開始：

```jsx
function RenderFunctionComponent() {
  const [firstName, setFirstName] = useState('Rudi');
  const [lastName, setLastName] = useState('Yardley');

  return (
  	<Button onClick={() => setFirstName('Fred')}>Fred</Button>
  );
}
```

Hooks API 背後的概念是您可以使用 setter，這個 setter 是 `useState` 執行之後回傳陣列的第二個值。 setter 可以控制 Hook 管理的 state 。



### React 是如何完成上述概念？

接著讓我們來說明 React 內部大概的運作機制。下面說明的概念也可以運用在特定執行環境下渲染元件。意思是儲存的資料是位於被渲染的元件外層。該 `state` 不會和其他元件共用，並且存在渲染該元件之後可以被存取的範圍。

### 1) 初始化

建立兩個空陣列 `setters` 和 `state`

設定一個指標為 0

![](https://miro.medium.com/max/640/1*LAZDuAEm7nbcx0vWVKJJ2w.png)

### 2) 第一次渲染

第一次執行元件函式。

每一個 `useState()`  第一次執行時，會 `push` 一個 setter function (和一個指標位置繫結) 到 `setters` 陣列然後 `push` 狀態到 `state` 陣列。

![](https://miro.medium.com/max/630/1*8TpWnrL-Jqh7PymLWKXbWg.png)

### 3) 隨後渲染

每次渲染之後， 指標就會重置，然後這些值就可以從陣列被讀取

![](https://miro.medium.com/max/627/1*qtwvPWj-K3PkLQ6SzE2u8w.png)

### 4) 事件處理

每一個 setter 都有一個參考指向該指標位置，所以觸發 setter 的時候就會變更陣列中 `state` 的值

![](https://miro.medium.com/max/630/1*3L8YJnn5eV5ev1FuN6rKSQ.png)



### 簡易的實作

下面是一個簡易的實作

```jsx
let state = [];
let setters = [];
let firstRun = true;
let cursor = 0;

function createSetter(cursor) {
  return function setterWithCursor(newVal) {
    state[cursor] = newVal;
  }
}

// 下面是 useState 的虛擬碼
export function useState(initVal) {
  if (firstRun) {
    state.push(initVal);
    setters.push(createSetter(cursor));
    firstRun = false;
  }

  const setter = setters[cursor];
  const value = state[cursor];
  cursor++;
  return [value, setter];
}

// 元件使用 Hook
function RenderFunctionComponent() {
  const [firstName, setFirstName] = useState('Rudi');
  const [lastName, setLastName] = useState('Yardley');

  return (
  	<div>
    	<Button onClick={() => setFirstName('Richard')}>Richard</Button>
      <Button onClick={() => setLastName('Fred')}>Fred</Button>
    </div>
  );
}

// 模擬 React 渲染
function MyComponent() {
  cursor = 0; // 重置 cursor
  return <RenderFunctionComponent />
}

console.log(state); // Pre render: []
MyComponent();
console.log(state); // First render: ['Rudi', 'Yardley']
MyComponent();
console.log(state); // Subsequent-render: ['Rudi', 'Yardley']

// click the 'Fred' button
console.log(state); // After-click: ['Fred', 'Yardley']
```



### 為什麼順序很重要

假設我們在渲染週期基於元件的狀態或一些外部條件改變了 Hooks 的順序會發生什麼事呢？

讓我們來試試那些 React 核心團隊建議我們**不該**做的事：

```jsx
let firstRender = true;

function RenderFunctionComponent() {
  let initName;

  if (firstRender) {
    [initName] = useState('Rudi');
    firstRender = false;
  }

  const [firstName, setFirstName] = useState(initName);
  const [lastName, setLastName] = useState("Yardley");

  return (
    <Button onClick={() => setFirstName("Fred")}>Fred</Button>
  );
}
```



### 第一次渲染

上面我們在條件式中使用了 `useState` 。接著我們來看看會造成什麼問題：

![](https://miro.medium.com/max/635/1*C4IA_Y7v6eoptZTBspRszQ.png)

### 第二次渲染

到這一步 `firstName` 和 `lastName` 的值都還是正確的，不過我們再觀察一下第二次渲染：



![](https://miro.medium.com/max/637/1*aK7jIm6oOeHJqgWnNXt8Ig.png)

> 備註：再多讀一次簡易實作，第一次渲染時會 push state 並回傳您給定的值。第二次開始就是直接取值。

所以現在 `firstName` 和 `lastName` 都是 `Rudi`。這是很明顯的錯誤，第一次和後續取值的東西是不一致的。因此 React 團隊規範了使用規則因為不遵循這個規則那麼資料將會不一致。

思考一下 Hooks 是操作一組陣列而您不應該破壞其規則。現在您應該很清楚為什麼不可以在條件式或迴圈調用 Hook 了，因為它是依賴 `cursor` 來取得陣列中的值，如果改變了順序那麼資料和 setter 就不對了。



## 結論

希望我有清楚的給您一個關於 Hook API 的說明。記得 Hook 真正的價值在於將相關邏輯的狀態組織在一次，並且您應該要小心順序。


* [React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)

---
title: 如何使用 React Hooks 搭配 Context API 取代 Redux 快速範例入門
date: 2019-08-11 21:24:47
categories: Program
tags:
  - react
  - javascript
---

本筆記為閱讀 [How to Replace Redux with React Hooks and the Context API](https://www.sitepoint.com/replace-redux-react-hooks-context-api/) 後自行實作調整簡化之範例。對於希望直接從範例學習的讀者可自行練習，本範例並非非常完整的教學，不過在能帶給您在使用 Hook 和 Context 上一些啟發：

<!--more-->

```bash
# 為簡化專案本範例使用 Parcel
$ npm i -g parcel-bundler
```

## 建立 Parcel 專案

```bash
$ mkdir demo-hooks-context
$ cd demo-hooks-context
$ npm init -y
$ touch index.html
$ touch index.js
```

####  *index.html*

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Demo</title>
</head>
<body>
  <div id="app"></div>
  <script src="./index.js"></script>
</body>
</html>
```

#### *index.js*
```js
console.log('Hello World');
```

```bash
$ parcel index.html
# Server running at http://localhost:1234
```

## 設定 React

```bash
$ npm i react react-dom
# 安裝 Babel
$ npm i babel-preset-env babel-preset-react --save-dev
$ touch .babelrc
```

#### *.babelrc*

```json
{
  "presets": ["env", "react"]
}
```

#### 變更 *index.js*

```js
import React from 'react';
import ReactDOM from 'react-dom';

const App = () => (
  <h1>Hello, React</h1>
);

const rootElement = document.getElementById('app');
ReactDOM.render(
  <App />,
  rootElement,
);
```

#### 加入 npm scripts

```json
{
  ...
  "scripts": {
    "start": "parcel index.html"
  },
  ...
}
```

## 第一個範例 - 計數器

```bash
# 安裝 constate
$ npm i constate
# 建立 context 目錄彙整 context 物件
# context 物件用於整合 useContext 和 Context 元件
$ mkdir context
$ touch context/CounterContext.js
```

#### *context/CounterContext.js*

```js
import { useState } from 'react';
import createUseContext from 'constate';

// 步驟 1 建立自訂 Hook，包含要使用的 state 和處理函式
const useCounter = () => {
  const [count, setCount] = useState(0);
  const increment = () => setCount(prevCount => prevCount + 1);
  const decrement = () => setCount(prevCount => prevCount - 1);
  return {
    count,
    increment,
    decrement,
  };
};

// 步驟 2 利用 constate 的函式協助我們建立 Context 物件
export const useCounterContext = createUseContext(useCounter);
```

#### 建立 Counter 和其子元件

這裡為了示範共享狀態，所以我們把一個單純可以在同元件的效果拆成多個：

```bash
$ mkdir views
$ touch views/Counter.js
$ mkdir components
$ touch components/CounterDisplay.js
$ touch components/CounterButtons.js
```

#### *views/Counter.js*

```jsx
import React from 'react';
import CounterDisplay from '../components/CounterDisplay';
import CounterButtons from '../components/CounterButtons';
import { useCounterContext } from '../context/CounterContext';

// 步驟 3 類似 Redux 的 connect ，將需要取得共享狀態的元件使用 Provider 包起來
export default function Counter() {
  return (
    <useCounterContext.Provider>
      <h3>Counter</h3>
      <CounterDisplay />
      <CounterButtons />
    </useCounterContext.Provider>
  );
}
```

#### *components/CounterDisplay.js*

```jsx
import React from 'react';
import { useCounterContext } from '../context/CounterContext';

export default function CounterDisplay() {
  // 步驟 4  使用 Context 物件存取共享的狀態
  const { count } = useCounterContext();
  return (
    <div>
      Counter: {count}
    </div>
  );
}
```

#### *components/CounterButtons.js*

```jsx
import React from 'react';
import { useCounterContext } from '../context/CounterContext';

export default function CounterButtons() {
  // 步驟 4  使用 Context 物件調用處理函式
  const { increment, decrement } = useCounterContext();
  return (
    <div>
      <button onClick={increment}>Add</button>
      <button onClick={decrement}>Minus</button>
    </div>
  );
}
```

#### 調整 *index.js* 使用 Counter 元件

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import Counter from './views/Counter';

const App = () => (
  <div>
    <Counter />
  </div>
);

const rootElement = document.getElementById('app');
ReactDOM.render(
  <App />,
  rootElement,
);
```

```bash
# 瀏覽結果
$ npm start
```



## 第二個範例 - 聯絡人清單功能

#### 安裝相依套件

```bash
$ npm i lodash
```



#### 建立相關檔案

利用 `useReducer` 處理複雜的資料結構。

> 後續可以斟酌再和 [**redux-starter-kit**](https://github.com/reduxjs/redux-starter-kit/blob/master/docs/api/createSlice.md) 的 `createSlice` 一起使用。

```bash
# Context 物件
$ touch context/ContactContext.js
# 主元件
$ touch views/Contacts.js
# 子元件 - 聯絡人列表表格
$ touch components/ContactTable.js
# 子元件 - 新增聯絡人表單
$ touch components/ContactForm.js
```

#### *context/ContactContext.js*

```js
import { useReducer } from 'react';
import _ from 'lodash';
import createUseContext from 'constate';

// 宣告初始化狀態
const initialState = {
  contacts: [
    {
      id: '001',
      name: 'Andy',
      email: 'andy@uxtesting.io'
    },
    {
      id: '002',
      name: 'Calvert',
      email: 'calvert@uxtesting.io'
    },
    {
      id: '003',
      name: 'Aaron',
      email: 'aaron@uxtesting.io'
    },
  ],
};

// 宣告 reducer
const reducer = (state, action) => {
  switch (action.type) {
    case 'ADD':
      return {
        contacts: [
          ...state.contacts,
          action.payload,
        ],
      };
    case 'DEL':
      return {
        contacts: state.contacts.filter(contact => contact.id !== action.payload),
      };
    default:
      throw new Error();
  }
};

// 自訂 Hook 包含 state, dispatch, 處理函式等
const useContacts = () => {
  const [state, dispatch] = useReducer(reducer, initialState);
  const {
    contacts,
  } = state;
  const addContact = (name, email) => {
    dispatch({
      type: 'ADD',
      payload: {
        id: _.uniqueId(10),
        name,
        email,
      },
    });
  }
  const delContact = id => {
    dispatch({
      type: 'DEL',
      payload: id,
    });
  }

  return {
    contacts,
    addContact,
    delContact,
  };
};

export const useContactsContext = createUseContext(useContacts);
```

#### *views/Contacts.js*

```jsx
import React from 'react';
import ContactForm from '../components/ContactForm';
import ContactTable from '../components/ContactTable';
import { useContactsContext } from '../context/ContactContext';

export default function Contacts() {
  return (
    <useContactsContext.Provider>
      <h1>Contacts</h1>
      <ContactForm />
      <ContactTable />
    </useContactsContext.Provider>
  );
}
```

#### *components/ContactTable.js*

```jsx
import React from 'react';
import { useContactsContext } from '../context/ContactContext';

export default function ContactTable() {
  const { contacts, delContact } = useContactsContext();
  return (
    <div>
      <table>
        <thead>
          <tr>
            <th>Id</th>
            <th>Name</th>
            <th>Email</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          {contacts.map(contact => (
            <tr key={contact.id}>
              <td>{contact.id}</td>
              <td>{contact.name}</td>
              <td>{contact.email}</td>
              <td>
                <button onClick={() => {
                  delContact(contact.id);
                }}>
                  X
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

    </div>
  );
}
```

#### *components/ContactForm.js*

```jsx
import React, { useState } from 'react';
import { useContactsContext } from '../context/ContactContext';

const initialFormState = {
  name: undefined,
  email: undefined,
};

export default function ContactForm() {
  const [inputs, setInputs] = useState(initialFormState);
  const { addContact } = useContactsContext();

  const onChange = (e) => {
    const {
      name,
      value,
    } = e.target;
    setInputs({
      ...inputs,
      [name]: value,
    });
  };

  const onSubmit = (e) => {
    e.preventDefault();
    const {
      name, email,
    } = inputs;
    addContact(name, email);
    // Reset
    setInputs(initialFormState);
  };

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input type="text" name="name" onChange={onChange} placeholder="Name" value={inputs.name || ''} />
        <input type="text" name="email" onChange={onChange} placeholder="Email" value={inputs.email || ''} />
        <input type="submit" value="Submit" />
      </form>
    </div>
  );
}
```

#### 調整 *index.js*

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
// import Counter from './views/Counter';
import Contacts from './views/Contacts';

const App = () => (
  <div>
    {/* <Counter /> */}
    <Contacts />
  </div>
);

const rootElement = document.getElementById('app');
ReactDOM.render(
  <App />,
  rootElement,
);
```

## 小結

希望這兩個小範例能夠讓您建立一些使用上的觀念。使用這種方式可以省去匯入一堆 Redux 相關的函式也可以省去 props 的傳遞。核心觀念就是利用 `constate` 的 `createUseContext` 建立一個包含 `Provider` 和 Hook 集合的 Context 物件。[constate](https://github.com/diegohaz/constate/blob/master/src/index.tsx) 本身原始碼也不多，如果您有興趣理解更細的話可以自行閱讀。

目前這樣的作法有個缺點就是不能使用 Redux DevTool。所以採用前請三思。



## 資源

* [Set Up A React Project With Parcel](https://scotch.io/tutorials/setting-up-a-react-project-with-parcel)
* [How to Replace Redux with React Hooks and the Context API](https://www.sitepoint.com/replace-redux-react-hooks-context-api/)

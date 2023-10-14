---
title: "認識 for await"
date: 2023-09-01 17:08:40
tags:
  - javascript
categories: Program
---

首先，參考以下的實務範例：

```js
async function main() {
  const stream = await opeanai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      { role: 'user', content: '測試回答' },
    ],
  });
  for await(const part of stream) {
    process.stdout.write(part.choices[0]?.delta?.content || '');
  }
}
```

這是一段 OpenAI Node SDK 的範例，使用 GPT-4 搭配串流的方式。

<!-- more -->



## 目標

理解 `for await` 這個神奇的語法。


## Generator

讓我們從 `Generator` 開始，為什麼要使用 `Generator`? 這是因為 `Generator` 有以下的特性：

- 逐一訪問元素。
- 優化效能，採用 Lazy Evaluation, 假設我們有一個超大的集合，陣列會在讀取時就將所有的值計算並放入記憶體，而 `Generator` 只有在呼叫 `next()` 時才會計算出下一個值。
- 統一不同資料結構（陣列、字串、集合（Set））遍歷的方式。

> `Generator Function` 的另一個使用情境就是可以暫停函式，再恢復執行。

簡單說，一個 `Generator Funciton` 會回傳 `Generator` 物件。

且這個物件必須符合**可迭代協議**和**迭代器協議**。

- 可迭代協議：指的是物件必須實作 `Symbol.iterator` 方法，然後就可以使用 `for ... of` 迴圈。陣列、字串、集合（Set）都是可迭代的，因為它們都實作了。
- 迭代器協議：則是物件必須要包含 `next()` 方法。

讓我們使用一個例子來看看 `Generator` 的使用方式：

```js
const go = function* () {
  yield '1';
  yield '2';
  yield '3';
}
for (const n of go()) {
  console.log(n);
}

// 逐步展示 Generator 運作
function* missions() {
  console.log('任務開始');
  const afterY1 = yield "A";
  console.log('接收', afterY1);
}

const g = missions();  // 第一次呼叫回傳一個 Generator 物件
const y1 = g.next();   // 執行到第一個 yield 函式暫停執行，回傳一個物件
console.log(y1.value); // value 為剛剛暫停 yield 右側的 expression 執行結果 A
console.log(y1.done);  // false;
const y2 = g.next('B');// 傳入值 B
console.log(y2.done);  // true;
g.next();              // 再呼叫會拋出例外。
```

## Async Generator + for await

了解了 `Generator`，您大概已經想到一個常見的使用情境 - 依序發送 API 請求。

您大概想要在 `yield` 右邊的 expression 呼叫 API 並取得資料，甚至使用 `await`。
`yield` 的右邊確實可以使用 `Promise` ，但問題是它不會等 `Promise` 執行完成。

因此我們需要 `Async Generator`。

```js
async function* missions() {
  yield await Promise.resolve('1'); // 這裡可以替換成您的 API request
  yield await Promise.resolve('2');
  yield await Promise.resolve('3');
}

const cb = async () => {
  // await 須在 async 裡面。
  for await (const v of missions()) {
    console.log(v);
  }
}
```

現在您已經了解 OpenAI SDK 的範例了。


## 補充：Mobx flow 不太一樣

如果您曾經使用過 Mobx 對於上面說的一般 `Generator` 不會等 `Promise` 完成有點疑問。
因為您確實看過如下的例子：

```js
import { observable, flow } form 'mobx';

class UserStore {
  @observable user = null;
  @observable isLoading = false;
  @observable error = null;
  
  fetchUser = flow(function* (id) {
    this.isLoading = true;
    this.error = null;
    
    try {
      const response = yield fetch('/your/api/${id}');
      if (resposne.ok) {
        this.user = yield response.json();
      } else {
        this.error = 'Failed';
      }
    } catch (error) {
      this.error = error;
    } finally {
      this.isLoading = false;
    }
  });
}
const userStore = new UserStore();
userStore.fetchUser(1);
```


原因是雖然 MobX 基於 Generator，但使用 `flow` 包住 `Generator` 時，其行為有所不同。當遇到 yield，flow 會暫停並等待 Promise 完成，再繼續執行。這讓非同步操作在 action 中不會違反 MobX 的反應性原則。
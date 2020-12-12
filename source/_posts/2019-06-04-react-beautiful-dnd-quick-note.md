---
title: React Beautiful Dnd 快速使用筆記
date: 2019-06-04 16:14:55
categories: Program
tags:
  - react
  - javascript
  - dnd
---


[react-beautiful-dnd](https://github.com/atlassian/react-beautiful-dnd) 有 3 個主要元件：

* DragDropContext - 建立一個可 DnD 的範圍。
  * onDragStart
  * onDragUpdate
  * onDragEnd
* Droppable - 建立可以被拖曳放入的區塊。
* Draggalbe -  可被拖拉元件

<!-- more -->

0. 安裝

```bash
$ npm i react-beautiful-dnd
```

1. 於希望 DnD 的位置使用 DragDropContext 包起來，且 `onDragEnd` 必須設定：

```jsx
import { DragDropContext } from 'react-beautiful-dnd';

<DragDropContext
  onDragEnd={() => {}}
>
  {/* Your target */}
</DragDropContext>
```

2. 使用 Droppable 配置可被拖入的區塊
   * Droppable 必須設定 `droppableId`
   * Droppable 使用 *render props pattern* 意味著內部須使用一個 function。這個 function 使用 `provided` 將所需的參數代置 DOM
   * `provided` 是一個物件
     * `provided.droppableProps`，該參數須套用至欲被拖放的 DOM 上
     * `ref` 參數須設定
     * `provided.placeholder`

```jsx
impoort { Droppable } from 'react-beautiful-dnd';

<Droppable droppableId="id">
  {
    provided => (
    	<List
        ref={provided.innerRef}
        {...provided.droppableProps}
      >
      	{provided.placeholder}
      </List>
    )
  }
</Droppable>
```

3. 將可被拖移的元件使用 `Draggable` 包起來：
   * `draggableId` 和 `index` 必須。
   * 與 `Droppable` 一樣，children 為 *render props pattern*
   * 內部項目 `ref` 須設定，`provided.draggableProps` 和 `provided.dragHandleProps` 須套用至元件

```jsx
<Draggable
  draggableId={t.id}
  index={i}
 >
    {p => (
        <Item
        ref={p.innerRef}
        {...p.draggableProps}
        {...p.dragHandleProps}
        key={t.id}
      >
        {t.text}
        </Item>
    )}
  </Draggable>
```

4. `onDragEnd` 變更排序

```jsx
// onDragEnd(result) result 資訊

const result = {
  draggableId: 1,
  type: 'TYPE',
  reson: 'DROP',
  source: {
    droppableId: 1,
    index: 0,
  },
  destination: {
    droppableId: 1,
    index: 1,
  }
}
```

Reorder:

```JS
onDragEnd={result => {
  const { source, destination, draggableId } = result;
  if (!destination) {
    return;
  }

  let arr = Array.from(this.state.todos);
  const [remove] = arr.splice(source.index, 1);
  arr.splice(destination.index, 0, remove);
  this.setState({
    todos: arr,
  });
}}
```

<iframe src="https://codesandbox.io/embed/y31owyopwv" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

在拖拉的時候，Droppable， Draggable 元件除了 `provided` 之外還提供了 `snapshot`

下面是 `snapshot` 提供的資訊：

```js
const draggableSnapshot = {
  isDragging: true,
  draggingOver: 'droppable-id'
}

const droppableSnapshot = {
  isDraggingOver: true,
  draggingOverWith: 'draggable-id'
}
```



回到 DragDropContext 除了 `onDragEnd` 之外還有 `onDragStart` 和 `onDragUpdate` 兩個 *events*。
這兩個事件可以協助我們加入一些樣式：

```js
// onDragStart
const start = {
  draggableId: 'draggalbe-id',
  type: 'TYPE',
  source: {
    droppableId: 'droppable-id',
    index: 0,
  }
}

// onDragUpdate
const update = {
  ...start,
  destination: {
    droppableId: 'droppable-id',
    index: 1,
  }
}
```



* drag handle  是 Draggable 的一部分用來處理可拖拉的部分（可拖拉整個元件或局部）。
* 要停止拖拉可以設定 `isDragDisabled`
* Droppable 可以限制可被拖入的元件：
  * `type` 同樣 type 的 Droppable 可以互相拖移
  * `isDropDisabled` 控制可否被拖入

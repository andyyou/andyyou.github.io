## 範例教學 - Todo List
讓我們透過一個經典的範例 TodoMVC 應用程式，展示關於 Flux 架構的實際 coding 是怎麼回事。關於全部的範例都在 [flux-todomvc](https://github.com/facebook/flux/tree/master/examples/flux-todomvc/)
檔案庫 `example` 目錄中，不過讓我們一步一步的介紹整個開發流程。

首先，我們需要一個樣板和準備模組化的執行環境。基於 CommonJS 的 Node 模組系統非常適合，您可以透過我們預先建好的[react-boilerplate]快速的取得這個環境。
如過您已經安裝了 npm ，只需要下載這個專案，接著在該目錄中執行 `npm install` -> `npm build` 最後執行 `npm start` 使用 `Browserify`。

關於 TodoMCV 的範例已經完整的建置好了，不過如果您想從 `react-boilerplate` 開始請檢查比對您的 package.json與 TodoMVC 的 package.json 是否有差異，以確保關於目錄結構與套件相依性的設定一致。

## 原始碼的架構
我們的確可以直接拿 `index.js` 來當作程式的進入點，不過在這個範例我們將會把程式碼放置到 `js` 的目錄中，接著透過 `Browserify` 來做這件事，現在讓我們開啟終端機來看看這個目錄它應該如下:

```
myapp
| + ...
  + js
  | + app.js
    + bundle.js // 無論對程式碼做了什麼修改 Browserify 都會為我們產生更新這隻檔案
  + index.html
  + ...
```
> 關於這邊我們將不回深入探討 Browserify 不清楚的可以暫時先理解為它把 CommonJS require() 這種模組化用法的機制搬到了前端。

接著我們來深入探討關於 `js` 目錄以及組織應用程式主要的目錄架構:

```
myapp
  |
  + ...
  + js
    |
    + actions
    + components // all React components, both views and controller-views
    + constants
    + dispatcher
    + stores
    + app.js
    + bundle.js
  + index.html
  + ...
```
現在我們已經準備好先來建立 `dispatcher`。有不切實際的 Dispatcher 類別範例將使用 Javascript promises polyfilled。(即我們透過額外的處理讓我們的程式支援 ES6 的寫法)


```js
var Promise = require('es6-promise').Promise;
var merge = require('react/lib/merge');

var _callbacks = [];
var _promises = [];

var Dispatcher = function () {};

Dispatcher.prototype = merge(Dispatcher.prototype, {
  register: function (callback) {
    _callbacks.push(callback);
    return _callbacks.length - 1;
  },

  dispatch: function (payload) {
    var resolves = [];
    var rejects = [];
    _promises = _callbacks.map(function (_, i) {
      return Promise(function (resolve, reject) {
        resolves[i] = resolve;
        rejects[i] = reject;
      });
    });
    _callbacks.forEach(function (callback, i) {
      Promise.resolve(callback(payload)).then(function () {
        resolves[i](payload);
      }, function () {
        rejects[i](new Error('Dispatcher callback unsuccessful'));
      });
    });
    _promises = [];
  }
});

module.exports = Dispatcher;
```

基本的 Dispatcher 由 register() 和 dispatch() 兩個 Methods 所組成。我們會在 `stores` 裏面使用 register() 來註冊關於儲存方面的回呼函式(callback)。
然後我們在 `actions` 裏面使用 dispatch() 去觸發 callback 的執行。

現在針對這個應用程式建立了一個 dispatcher 分派調度器: AppDispatcher。

```js
var Dispatcher = require('./Dispatcher');

var merge = require('react/lib/merge');

var AppDispatcher = merge(Dispatcher.prototype, {
  handleViewAction: function (action) {
    this.dispatch({
      source: 'VIEW_ACTION',
      action: action
    });
  }
});

module.exports = AppDispatcher;
```

針對我們的需求，完成了上面的實作，它提供了一個輔助方法讓我們在 View 的事件處理的 actions 中使用。稍後我們可能會擴展它提供獨立的輔助函式以更新伺服器端的資料，但現在這就是我們要的全部了。

## 建立 Stores
我們可以使用 Node 的 EventEmitter 來學習 store。我們需要 EventEmitter 來廣播 'change' 事件到我們的 controller-views。所以讓我們來看看它到底長成什麼樣子。
為了簡單易懂這邊省略了一些程式碼，不過您可以在 Github 上找到完整的版本 [TodoStore.js](https://github.com/facebook/flux/blob/master/examples/flux-todomvc/js/stores/TodoStore.js)

```js
var AppDispatcher = require('../dispatcher/AppDispatcher');
var EventEmitter = require('events').EventEmitter;
var TodoConstants = require('../constants/TodoConstants');
var merge = require('react/lib/merge');

var CHANGE_EVENT = 'change';

var _todos = {}

function create(text) {
  var id = Date.now();
  _todos[id] = {
    id: id,
    complete: false,
    text: text
  };
}

function destory(id) {
  delete _todos[id];
}

var TodoStore = merge(EventEmitter.prototype, {
  getAll: function () {
    return _todos;
  },

  emitChange: function () {
    this.emit(CHANGE_EVENT);
  },

  addChangeListener: function (callback) {
    this.on(CHANGE_EVENT, callback);
  },

  removeChangeListener: function (callback) {
    this.removeListener(CHANGE_EVENT, callback);
  },

  dispatcherIndex: AppDispatcher.register(function (payload) {
    var action = payload.action;
    var text;

    switch(action.actionType) {
      case TodoConstants.TODO_CREATE:
        text = action.text.trim();
        if (text !== '') {
          create(text);
          TodoStore.emitChange();
        }
        break;
      case TodoConstants.TODO_DESTORY:
        destory(action.id);
        TodoStore.emitChange();
        break;
    }
    return true;
  })

});

module.exports = TodoStore;
```
這邊有一些很重要的事情需要注意，一開始我們維護了一份 private 的資料結構稱為 `_todos`。這個物件包含了所有獨立的待辦事項。因為這個變數存在類別之外，但是在模組的閉包中。
所以它仍然是 private，它不能被直接存取或修改。這可以幫助我們保持不同的資料流輸入/輸出界面，這樣就能做到不使用 action 來更新資料。
另外一個重要的部分是關於 store callback 和 dispatcher 的註冊，我們透過 payload 處理 callback 到 dispatcher 和保存 index 到這個 store

## 監聽 Controller-View 的變化
我們需要一個 React 元件接近結構頂層來監聽 store 的變化，在一個大型應用程式中我們需要監聽更多的元件，或者是頁面其中一部份。在 Facebook Ads 建立工具，我們有很多類似 Controller 的 View
每一個各自統支特定部分的 UI，在 Lookback Video Editor 我們只有兩個，一個是動畫預覽一個是選擇圖片
在這個 TodoMVC 範例，再一次這略有刪減，不過完整版的程式碼您還是可以在 Github 找到
```js
/**
 * @jsx React.DOM
 */
var Footer = require('./Footer.react');
var Header = require('./Header.react');
var MainSection = require('./MainSection.react');
var React = require('react');
var TodoStore = require('../stores/TodoStore');

function getTodoState() {
  return {
    allTodos: TodoStore.getAll()
  };
}

var TodoApp = React.createClass({
  getInitialState: function () {
    return getTodoSate();
  },

  componentDidMount: function () {
    TodoStore.addChangeListener(this._onChnage);
  },

  componentWillUnmount: function () {
    TodoStore.removeChangeListener(this._onChange);
  },

  render: function () {
    return (
      <div>
        <Header />
        <MainSection
          allTodos={this.state.allTodos}
          areAllComplete={this.state.areAllComplete}
          />
        <Footer allTodos={this.state.allTodos}/>
      </div>

    )
  },

  _onChange: function () {
    this.setState({getTodoState()})
  }
});

module.exports = TodoApp;
```
我們已經熟悉如何使用 React 的生命週期這部分。在這邊我們設定了初始化的狀態，在 componentDidMount() 註冊了事件監聽，然後當 componentWillUnmount 時清除。
我們輸出了當作容器的 div 以及從 TodoStore 取得的狀態資料。
Header 元件包含一個純文字 input ，這部分不需要知道關於 store 的狀態，MainSection 和 Footer 需要這些資料所以我們也把它加入到底下。

## 更多 Views
從上來看，這個程式的 React 階層結構長得像這樣:

```js
<TodoApp>
  <Header />
  <TodoTextInput />
  <MainSection>
    <ul>
      <TodoItem />
    </ul>
  </MainSection>
</TodoApp>
```

如過一個 TodoItem 處於編輯模式，那他也需要輸出一個 TodoTextInput 子元件。讓我們來看看這些元件該如何呈現資料例如收到得參數 `props` 以及它們如何透過 actions 跟 dispatcher 溝通。
MainSection 需要迭代從 TodoApp 拿到的 to-do 代辦項目集合來建立 TodoItems 列表。在這個元件的 render() 方法中，我們可以這麼做如下:

```js
var allTodos = this.props.allTodos;

for (var key in allTodos) {
  todos.push(<TodoItem key={key} todo={allTodos} />);

}

return (
  <section id="main"> <ul id="todo-list">{todos}</ul>
)
```

接著每一個 TodoItem 可以顯示它們各自的項目文字與透過唯一的 ID 來執行刪除之類的動作。解釋 TodoItem 可以調用的所有不同動作(actions) 超出了這篇文章的範圍，不過我們還是看看關於刪除動作，下面有一個省略的版本

```js
/**
 * @jsx React.DOM
 */

var React = require('react');
var TodoTextInput = require('../actions/TodoActions');
var TodoTextInput = require('./TodoTextInput.react');

var TodoItem = React.createClass({
  propTypes: {
    todo: React.PropTypes.object.isRequired
  },

  render: function {
    var todo = this.props.todo;

    return (
      <li
        key={todo.id}>
        <label>
          {todo.text}
        </label>
        <button className='destory' onClick={this._onDestroyClick} />
      </li>
    )
  },

  _onDestroyClick: function () {
    TodoActions.destroy(this.props.todo.id);
  }
});

module.exports = TodoItem;
```

TodoActions 的函式庫提供了一個 destroy 的動作，且 store 準備處理它，連接程式的使用者介面的狀態。我們只是將這些處理的動作包在 onClick 處理事件中，並提供他一個 ID。
現在使用者可以點擊刪除的按鈕接著觸發 Flux 的流程去更新應用程式。

文字輸入 input，從另一個角度來看，有一些些比較複雜，因為我們需要和元件內部的 state 打交道，同樣的觀察一下 TodoTextInput 如何運作。
當你看到下面，任何在 input 的改變，React 期待我們去更新元件的 state ，所以當我們準備儲存時，我們必須將值在動作的 payload 中的元件的 state 之中，並且
持續在心裡記住他們的區別有助您配置 state 到正確的地方。所有程式的 state 應該放在 store, 少數情況下元件需要自己儲存狀態，理想來說 React 元件應該盡可能不要保留狀態。
因為 TodoTextInput 被用程式的很多地方，且具備著不同的行為，我們將需要從父元件透過 prop 傳遞 onSave 方法，這讓 onSave 可以根據不同的情況調用不同的 action

```js
/**
 * @jsx React.DOM
 */

var React = require('react');
var ReactPropTypes = React.PropTypes;

var ENTER_KEY_CODE = 13;

var TodoTextInput = React.createClass({
  propTypes: {
    className: ReactPropTypes.string,
    id: ReactPropTypes.string,
    placeholder: ReactPropTypes.string,
    onSave: ReactPropTypes.func.isRequired,
    value: ReactPropTypes.string
  },

  getInitialState: function () {
    return {
      value: this.props.value || ''
    };
  },

  render: function () {
    return (
      <input
        className={this.props.className}
        id={this.props.id}
        placeholder={this.props.placeholder}
        onBlur={this._save}
        onChange={this._onChange}
        onKeyDown={this._onKeyDown}
        value={this.state.value}
        autoFocus={true}
      />
    );
  },

  _save: function () {
    this.props.onSave(this.state.value);
    this.setState({
      value: ''
    });
  },

  _onChange: function (e) {
    this.setState({
      value: e.target.value
    })
  },

  _onKeyDown: function (e) {
    if (e.keyCode === ENTER_KEY_CODE) {
      this._save();
    }
  }
})

module.exports = TodoTextInput;

```

Header 標題列則需要把 onSave 方法作為 prop 傳給 TodoTextInput 來建立一個新的項目:

```js
/**
 * @jsx React.DOM
 */
var React = require('react');
var TodoActions = require('../actions/TodoActions');
var TodoTextInput = require('./TodoTextInput.react');

var Header = React.createClass({
  render: function () {
    <header id='header'>
      <h1>todos</h1>
      <TodoTextInput
        id='new-todo'
        placeholder='what needs to be done?'
        onSave={this._onSave}
        />
    </header>
  },

  _onSave: function (text) {
    TodoAction.create(text)
  }
});

module.exports = Header;

```
在不同的執行環境下，例如編輯一個已存在的項目，我們可能換成在 onSave 函式中換成調用 TodoActions.update(text)

## 建立語意動作
下面是關於我們上面所提到會需要的兩個動作的基本程式碼:

```
```

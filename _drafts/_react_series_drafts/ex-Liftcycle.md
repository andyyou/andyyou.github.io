# 元件的規範與生命週期

## 元件的規範
當我們透過 `React.createClass` 建立一個元件類別時，您應該提供一個遵循規範的物件 - 即至少包含一個 `render` 方法和其他生命週期中可選的方法。

## render
```
ReactComponent render()
```
render 是必須存在的方法，當其被呼叫後它會去檢查 `this.props` 和 `this.state` 接著只回傳一個元件。這個元件可以是 DOM 元件(例如 <div/> 或者 React.DOM.div)或者是其他您自己定義的元件。
當然您也可以回傳 null 或者 false，如此一來便不會渲染輸出任何東西，不過在內部 React 還是渲染了一個 `<noscript>` 用來在虛擬 DOM 機制中做比較的。當您回傳 null 或者 false 之後 `this.getDOMNode()` 將會回傳 null。
關於 render() 方法應該盡可能保持單純，指的是說在這個方法中您不應該去修改 state ，否則可能會造成一直回傳相同的值，因為你一修改 state，state 就會發動 render。再者是您也不應該在這個地方讀取或修改 DOM。
如果您需要和 DOM 互動，則應該在 `componentDidMount()` 做事，或者其他適合的生命週期方法中。

## getInitialState
```
object getInitialState()
```
在元件被掛載之前會被執行，且只有一次。這個方法是用來初始化 this.state 的。

## getDefaultProps
```
object getDefaultProps()

```
一旦類別被建立這個方法會被執行一次，且保存在暫存區。如果 props 沒有指定任何值，它的值會被映射到 this.props。
這個方法會在任何實例物件建立之前被執行。此外要注意的是如果您透過 getDefaultProps 傳回一個複雜的物件，那麼這個物件將被所有實例物件共享，簡單說就是指向參考而非複製。

## propTypes
```
object propTypes
```
propTypes 是用來驗證 props 的，如果您需要知道更多關於限制型別的資訊請[查閱](http://facebook.github.io/react/docs/reusable-components.html)

## mixins
```
array mixins
```
mixins 陣列可以讓我們混入一些方法，讓不同的元件可以共用一些行為。

## static
```
object static
```
static 物件讓我們可以定義一些靜態的方法，然後可以在元件類別中使用，舉例來說:
```js
var MyComponent = React.createClass({
  statics: {
    customMethod: function(foo) {
      return foo === 'bar';
    }
  },
  render: function() {
  }
});

MyComponent.customMethod('bar');  // true
```
在 static 區塊中定義方法就會是靜態函式，意味著您可以在建立任何實例物件之前使用它們，而且這些方法不需要存取元件 props, state ，如果您想要在靜態方法中檢查 props ，您可以把 props 傳進這個靜態方法中。

## displayName
```
string displayName
```
displayName 字串通常被用在除錯時顯示一些資訊。JSX 會自動幫您設定這個值。

# 生命週期函式

## Mounting: componentWillMount
```
componentWillMount()
```
在輸出前觸發，只執行一次如果您在這個方法中呼叫 setState() ，會發現雖然 render() 再次被觸發了但它還是只執行一次。

## Mounting: componentDidMount
```
componentDidMount()
```
只在客戶端執行一次，當渲染完成後立即執行。當生命週期執行到這一步，元件已經俱有 DOM 所以我們可以透過 this.getDOMNode() 來取得 DOM 。
如果您想整和其他 Javascript framework ，使用 setTimeout, setInterval, 或者是發動 AJAX 請在這個方法中執行這些動作。

## Updating: componentWillReceiveProps
```
componentWillReceiveProps(object nextProps)
```
當元件收到新的 props 時被執行，這個方法在初始化時並不會被執行。使用的時機是在我們使用 setState() 並且呼叫 render() 之前您可以比對 props，舊的值在 this.props，而新值就從 nextProps 來。

```
componentWillReceiveProps: function(nextProps) {
  this.setState({
    likesIncreasing: nextProps.likeCount > this.props.likeCount
  });
}
```

> 注意並沒有 componentWillReceiveState 這種方法。通常傳入的 props 可能導致狀態改變，不過反過來操作是不正確的，如果您需要在 state 改變時做一些操作那麼請使用 componentWillUpdate。

## Updating: shouldComponentUpdate
```
boolean shouldComponentUpdate(object nextProps, object nextState)
```
如同其命名，是用來判斷元件是否該更新，當 props 或者 state 變更時會再重新 render 之前被執行。這個方法在初始化時不會被執行，或者當您使用了 forceUpdate 也不會被執行。
當你確定改變的 props 或 state 並不需要觸發元件更新時，在這個方法中適當的回傳 false 可以提升一些效能。

```
shouldComponentUpdate: function(nextProps, nextState) {
  return nextProps.id !== this.props.id;
}
```
如果 shouldComponentUpdate 回傳 false 則 render() 就會完全被跳過直到下一次 state 改變，此外 componentWillUpdate 和 componentDidUpdate 將不會被觸發。
當 state 產生異動，為了防止一些奇妙的 bug 產生，預設 shouldComponentUpdate 永遠回傳 true ，不過如果您總是使用不可變性(immutable)的方式來使用 state，並且只在 render 讀取它們那麼你可以複寫 shouldComponentUpdate
或者是當效能遇到瓶頸，特別是需要處理大量元件時，使用 shouldComponentUpdate 通常能有效地提升速度。

## Updating: componentWillUpdate
```
componentWillUpdate(object nextProps, object nextState)
```
當收到 props 或者 state 立即執行，這個方法在初始化時不會被執行，使用時機通常是在準備更新之前。
> 注意您不能在這個方法中使用 this.setState()。如果您需要在修改 props 之後更新 state 請使用 componentWillReceiveProps 取代

## Updating: componentDidUpdate
```
componentDidUpdate(object prevProps, object prevState)
```
在元件更新之後執行。這個方法同樣不在初始化時執行，使用時機為當元件被更新之後需要執行一些操作。

## Unmounting: componentWillUnmount
```
componentWillUnmount()
```
元件被從 DOM 卸載之前執行，通常我們在這個方法清除一些不再需要地物件或 timer。

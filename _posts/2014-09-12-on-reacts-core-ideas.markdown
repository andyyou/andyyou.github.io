---
layout: post
title: '關於 Reactjs 的核心觀念'
date: 2014-09-12 16:14:00
categories: Reactjs
---
這篇文章為官方部落格的文章隻翻譯。[原文](http://facebook.github.io/react/docs/thinking-in-react.html)。

以觀念來說`React` 是一種使用 Javascript 來快速建立大型 Web 的方式，它非常容易擴展，且官方已將其使用在 Facebook 與 Instagram 上。

其中最好的部分就是 React 讓您在建立程式時重新思考關於應用程式。在這篇文章，將會引導您使用 React 完成一個可以搜尋過濾產品資料的範例。

## 從模擬架構開始
想像我們已經有了一個 JSON 的 API 以及一個設計師模擬的草圖。我們的設計師顯然不是很優，因為他的模擬像這樣：

![](http://facebook.github.io/react/img/blog/thinking-in-react-mock.png)

而我們的 JSON API 傳回來的資料長得像這樣：

{% highlight json %}
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
{% endhighlight %}

## 第一步：拆解 UI 為元件階層結構

您即將要做的第一個步驟是根據模擬的 UI 畫出每一個元件(包含子元件)的階層方塊並且給予名稱，如果你正在與設計師一起工作那可能你已經完成這個任務。
看看他們的 Photoshop 中圖層的名稱大略就是你 React 元件最後的名稱。
不過我們怎麼知道哪個部分應該是元件？當您建立一個新的函式或物件，您可以根據單一職責原則，指的是每一個元件理想的情況下
應該只做一件事。如果該元件的功能不斷增加那就應該再把它拆解，並建立更小的子元件。

常見的需求是 - 顯示 JSON 的資料給使用者。根據過去的經驗，您會發現如果您的資料模型(Model)建立的正確，您的 UI (同時表示您的元件結構)就可以輕鬆的將資料呈現給使用者。
其原因是使用者界面和資料模型往往遵循一樣的資料結構，意味著其實將 UI 獨立為元件並非很困難。
就只是根據每一個小區塊需要呈現的資料模型去分解成個別的元件。

![](http://facebook.github.io/react/img/blog/thinking-in-react-components.png)

看到上圖，這個應用程式將會有 5 個元件，下面的斜體字表示每一個元件對應的模型

1. `FilterableProductTable` (橘色) 用來組織包含其他子元件，即這個元件的最上層的容器。
2. `SearchBar` (藍色) 取得 _使用者輸入的搜尋條件。_
3. `ProductTable` (綠色) 根據 _使用者輸入的搜尋條件_ 顯示過濾後的資料列表。
4. `ProductCategoryRow` (青色) 顯示 _分類_ 標題。
5. `ProductRow` (紅色) 顯示每一個 _產品_。

如果您認真觀察 `ProductTable` 你會看到表格還有標題列(即 `Name` 和 `Price` 欄位名稱那邊)並沒有被規劃為獨立元件。這只是偏好問題。
根據這個範例，我們規劃這個區塊為 `ProductTable` 的一部份，這是因為輸出產品列表資料是 `ProductTable` 的責任，當然包含欄位名稱，且目前看來它的工作很單純並不需要再拆出一個元件。
然而如果這個標題列變得越來越複雜(舉例來說：如果我們需要增加排序功能)，如此一來增加一個 `ProductTableHeader` 元件會是比較好的做法。

現在我們已經定義好關於這個模擬的元件架構，讓我們重新組織成一個階層圖，這樣我們就能清楚看出元件的主從關係。

* FilterableProductTable
  - SearchBar
  - ProductTable
    	- ProductCategoryRow
    	- ProductRow

## 第二步：建立一個靜態版本的 React 元件

{% highlight js %}
/** @jsx React.DOM */

var ProductCategoryRow = React.createClass({
    render: function() {
        return (<tr><th colSpan="2">{this.props.category}</th></tr>);
    }
});

var ProductRow = React.createClass({
    render: function() {
        var name = this.props.product.stocked ?
            this.props.product.name :
            <span style={{color: 'red'}}>
                {this.props.product.name}
            </span>;
        return (
            <tr>
                <td>{name}</td>
                <td>{this.props.product.price}</td>
            </tr>
        );
    }
});

var ProductTable = React.createClass({
    render: function() {
        var rows = [];
        var lastCategory = null;
        this.props.products.forEach(function(product) {
            if (product.category !== lastCategory) {
                rows.push(<ProductCategoryRow category={product.category} key={product.category} />);
            }
            rows.push(<ProductRow product={product} key={product.name} />);
            lastCategory = product.category;
        });
        return (
            <table>
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Price</th>
                    </tr>
                </thead>
                <tbody>{rows}</tbody>
            </table>
        );
    }
});

var SearchBar = React.createClass({
    render: function() {
        return (
            <form>
                <input type="text" placeholder="Search..." />
                <p>
                    <input type="checkbox" />
                    Only show products in stock
                </p>
            </form>
        );
    }
});

var FilterableProductTable = React.createClass({
    render: function() {
        return (
            <div>
                <SearchBar />
                <ProductTable products={this.props.products} />
            </div>
        );
    }
});


var PRODUCTS = [
  {category: 'Sporting Goods', price: '$49.99', stocked: true, name: 'Football'},
  {category: 'Sporting Goods', price: '$9.99', stocked: true, name: 'Baseball'},
  {category: 'Sporting Goods', price: '$29.99', stocked: false, name: 'Basketball'},
  {category: 'Electronics', price: '$99.99', stocked: true, name: 'iPod Touch'},
  {category: 'Electronics', price: '$399.99', stocked: false, name: 'iPhone 5'},
  {category: 'Electronics', price: '$199.99', stocked: true, name: 'Nexus 7'}
];

React.renderComponent(<FilterableProductTable products={PRODUCTS} />, document.body);
{% endhighlight %}

現在您已經有了元件的階層結構。該是時候實作程式的功能了。
在學習 React 的過程中，我們推薦最簡單的方式是一開始只要建立一個能取得資料模型和渲染出 UI 畫面但是不能互動的版本。
拆成這些步驟是因為建立靜態版本通常是需要打很多字且不太需要思考，而建立互動機制需要您仔細的構思，如此一來在建立元件時比較不會出錯。

在建立靜態版本的時候通常你會思考關於元件 _重複使用_ 的機會以及該怎麼透過 `props` 從父元素傳入資料。如果您已經熟悉關於 `state` 的觀念，就會知道在建立靜態版本時根本不應該使用 `state`。
`state` 是互動時才會需要用到的功能，所謂的互動指的是當資料變動，而 UI 也需要對應更新。由於這只是靜態版本所以根本不需要用。

您可以由底層往上或者由上而下撰寫您的元件，意思是說你可以選擇從結構中最外層的元件開始建起(即從 FilterableProductTable)開始，或者從最內部的子元件開始(ProductRow)。
在單純的範例中，通常從上至下相對快速，而如果專案較大，通常從下而上會比較推薦，因為也同時方便您撰寫測試，可以逐步測試元件是否正常。

在這一步的最後，您會得到一個可重複使用元件的函式庫，你可以用它來輸出呈現你的資料模型，以確認 UI 的呈現是否有誤。
不過這個元件只有 `render()` 方法，因為截至目前為止它還只是靜態版本。

元件的最上層(FilterableProductTable)將會取得資料模型，透過`屬性`傳入資料。如果你修改了 Model 的資料且再次執行 `renderComponent()` 你應該會看到資料更新了。
這讓你可以清楚地觀察這個元件是怎麼更新資料，這就是 React 透過 one-way data flow (或稱 one-way binding) 單向數據流的方式去保持所有資料一致，同時也方便模組化。

### 簡易補充: props vs state
在 React 裏有兩種類型的 `Model` 就是你放資料的地方：`props` 與 `state`。理解他們的區別非常重要，如果您還不懂他們之間的差別請閱讀[官方](http://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html)或者[這篇文章](http://andyyou.logdown.com/posts/178450-react)

## 第三步：定義最少但完整的 UI 狀態
為了讓您的 UI 俱有互動性，你可能會需要讓資料模型做些修改，接著 UI 根據 one-way binding 更新資料。
React 透過使用 `state` 讓這一切變得很簡單。您可以把 `state` 想是讓您存放動態資料的地方，而當資料有所變動，React 會自動呼叫 `render()` 執行 UI 的更新。

而要讓建立的程式能夠正確執行，首先需要思考關於這個程式最少需要哪些可變動的狀態，只有變動的資料才需要放到 `state`。
關鍵的原則是 DRY (Don't Repeat Yourself) ，不重複原則。找出程式在特定需求內必須要的最少狀態。例如：如果你要建立一個 TODO List，其實你就只要一個陣列包含待辦清單的項目。
當你需要計算項目總數時，不需要在 `state` 中的儲存另一個變數，而是單純使用陣列取得數量即可。

思考我們這個範例中各種取得的資料

* 所有產品的列表
* Search input 的搜尋條件
* checkbox 是否有被選取的值
* 過濾後的清單

讓我們一個一個討論看看誰是屬於 `state` 。簡單的思考關於這三個問題

1. 資料是透過 props 從父元素傳進來的嗎？如果是，這可能不屬於 state 。
2. 這資料會隨著時間推移而改變嗎？ 如果不是，那它應該不屬於 state 。
3. 你能從現有任何 state 或者 props 計算出這個資料嗎？如過是！那這肯定不屬於 staet。

產品列表是透過 props 傳遞進來的，所以這不應該存在 `state`，當然有經驗的開發者會說這通常從資料庫來，但這個範例不是。
搜尋條件和 checkbox 似乎是狀態，因為他們會改變。而且是不能透過計算得到的。
最後過濾後的清單也不該儲存在 `state` ，因為他是可以被計算出來的，根據我們拿到的過濾條件去運算。
所以最後我們歸納出應該被放在 `state` 的有:

* 搜尋條件
* checkbox 的值


## 第四步：應該在何處使用 `state`

{% highlight js %}
/** @jsx React.DOM */

var ProductCategoryRow = React.createClass({
    render: function() {
        return (<tr><th colSpan="2">{this.props.category}</th></tr>);
    }
});

var ProductRow = React.createClass({
    render: function() {
        var name = this.props.product.stocked ?
            this.props.product.name :
            <span style={{color: 'red'}}>
                {this.props.product.name}
            </span>;
        return (
            <tr>
                <td>{name}</td>
                <td>{this.props.product.price}</td>
            </tr>
        );
    }
});

var ProductTable = React.createClass({
    render: function() {
        var rows = [];
        var lastCategory = null;
        this.props.products.forEach(function(product) {
            if (product.name.indexOf(this.props.filterText) === -1 || (!product.stocked && this.props.inStockOnly)) {
                return;
            }
            if (product.category !== lastCategory) {
                rows.push(<ProductCategoryRow category={product.category} key={product.category} />);
            }
            rows.push(<ProductRow product={product} key={product.name} />);
            lastCategory = product.category;
        }.bind(this));
        return (
            <table>
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Price</th>
                    </tr>
                </thead>
                <tbody>{rows}</tbody>
            </table>
        );
    }
});

var SearchBar = React.createClass({
    render: function() {
        return (
            <form>
                <input type="text" placeholder="Search..." value={this.props.filterText} />
                <p>
                    <input type="checkbox" value={this.props.inStockOnly} />
                    Only show products in stock
                </p>
            </form>
        );
    }
});

var FilterableProductTable = React.createClass({
    getInitialState: function() {
        return {
            filterText: '',
            inStockOnly: false
        };
    },

    render: function() {
        return (
            <div>
                <SearchBar
                    filterText={this.state.filterText}
                    inStockOnly={this.state.inStockOnly}
                />
                <ProductTable
                    products={this.props.products}
                    filterText={this.state.filterText}
                    inStockOnly={this.state.inStockOnly}
                />
            </div>
        );
    }
});


var PRODUCTS = [
  {category: 'Sporting Goods', price: '$49.99', stocked: true, name: 'Football'},
  {category: 'Sporting Goods', price: '$9.99', stocked: true, name: 'Baseball'},
  {category: 'Sporting Goods', price: '$29.99', stocked: false, name: 'Basketball'},
  {category: 'Electronics', price: '$99.99', stocked: true, name: 'iPod Touch'},
  {category: 'Electronics', price: '$399.99', stocked: false, name: 'iPhone 5'},
  {category: 'Electronics', price: '$199.99', stocked: true, name: 'Nexus 7'}
];

React.renderComponent(<FilterableProductTable products={PRODUCTS} />, document.body);
{% endhighlight %}

OK 我們已經決定了這個元件程式最少的 `state`，下一步我們需要定義哪些元件是要變動的，以及是哪些要使用 `state` 。
記住！React 提供的是一種單向資料流的結構，通常是由上而下。剎那間可能不太輕易判斷哪個元件該管理或使用 `state`，這通常也是初學者最難理解的部分。
請跟著下面這些規則去推敲:

思考程式中需要使用到 `state` 的部分

* 找出哪些元件需要根據 `state` 輸出不同的結果
* 找出共同的擁有者元件(最上層的元件通常需要管理 `state`)
* 如果你不能找出某一個元件該擁有狀態的理由，那就建立一個新的元件用來管理狀態，並且加在共同擁有者元件之上。

讓我們應用這些規則在這個範例上

* `ProductTable` 需要根據 `state` 過濾產品列表以及 `SearchBar` 需要顯示搜尋條件的值和 checkbox 的狀態。
* 共同擁有者元件是 `FilterableProductTable`。
* 把過濾條件和 checkbox 值都放在 `FilterableProductTable` 在概念上也是合理的。

所以我們決定 `state` 應該放在 `FilterableProductTable` ，首先加上 `getInitialState()` 方法，讓它回傳一個物件 `{filterText: '', inStockOnly: false}`
這是用來初始化 `state` 的，接著傳入 `filterText` 和 `inStockOnly` 給 `SearchBar` 當作屬性，最後在 `ProductTable` 中使用這些屬性值去過濾，並且設定 form 的值。

現在你可以看到您的應用程式俱有這些行為：在 `state` 中把 `filterText` 的值設成 `ball` 然後資料就會更新。
> 註：先別急著操作網頁上的 form。

## 第五步：加入反向數據流

{% highlight js %}
/** @jsx React.DOM */

var ProductCategoryRow = React.createClass({
    render: function() {
        return (<tr><th colSpan="2">{this.props.category}</th></tr>);
    }
});

var ProductRow = React.createClass({
    render: function() {
        var name = this.props.product.stocked ?
            this.props.product.name :
            <span style={{color: 'red'}}>
                {this.props.product.name}
            </span>;
        return (
            <tr>
                <td>{name}</td>
                <td>{this.props.product.price}</td>
            </tr>
        );
    }
});

var ProductTable = React.createClass({
    render: function() {
        console.log(this.props);
        var rows = [];
        var lastCategory = null;
        this.props.products.forEach(function(product) {
            if (product.name.indexOf(this.props.filterText) === -1 || (!product.stocked && this.props.inStockOnly)) {
                return;
            }
            if (product.category !== lastCategory) {
                rows.push(<ProductCategoryRow category={product.category} key={product.category} />);
            }
            rows.push(<ProductRow product={product} key={product.name} />);
            lastCategory = product.category;
        }.bind(this));
        return (
            <table>
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Price</th>
                    </tr>
                </thead>
                <tbody>{rows}</tbody>
            </table>
        );
    }
});

var SearchBar = React.createClass({
    handleChange: function() {
        this.props.onUserInput(
            this.refs.filterTextInput.getDOMNode().value,
            this.refs.inStockOnlyInput.getDOMNode().checked
        );
    },
    render: function() {
        return (
            <form>
                <input
                    type="text"
                    placeholder="Search..."
                    value={this.props.filterText}
                    ref="filterTextInput"
                    onChange={this.handleChange}
                />
                <p>
                    <input
                        type="checkbox"
                        value={this.props.inStockOnly}
                        ref="inStockOnlyInput"
                        onChange={this.handleChange}
                    />
                    Only show products in stock
                </p>
            </form>
        );
    }
});

var FilterableProductTable = React.createClass({
    getInitialState: function() {
        return {
            filterText: '',
            inStockOnly: false
        };
    },

    handleUserInput: function(filterText, inStockOnly) {
        this.setState({
            filterText: filterText,
            inStockOnly: inStockOnly
        });
    },

    render: function() {
        return (
            <div>
                <SearchBar
                    filterText={this.state.filterText}
                    inStockOnly={this.state.inStockOnly}
                    onUserInput={this.handleUserInput}
                />
                <ProductTable
                    products={this.props.products}
                    filterText={this.state.filterText}
                    inStockOnly={this.state.inStockOnly}
                />
            </div>
        );
    }
});


var PRODUCTS = [
  {category: 'Sporting Goods', price: '$49.99', stocked: true, name: 'Football'},
  {category: 'Sporting Goods', price: '$9.99', stocked: true, name: 'Baseball'},
  {category: 'Sporting Goods', price: '$29.99', stocked: false, name: 'Basketball'},
  {category: 'Electronics', price: '$99.99', stocked: true, name: 'iPod Touch'},
  {category: 'Electronics', price: '$399.99', stocked: false, name: 'iPhone 5'},
  {category: 'Electronics', price: '$199.99', stocked: true, name: 'Nexus 7'}
];

React.renderComponent(<FilterableProductTable products={PRODUCTS} />, document.body);
{% endhighlight %}

到上面為止你會發現除非你手動更改 state 的設定，如果你在網頁的表單上輸入任何東西，input 完全沒反應。這是因為 React 是單向數據流的模式。
現在讓我們補上其他方向來的數據，由於表單元件在這個結構的內部，而我們只能用 `FilterableProductTable` 去更新 `state` 。
React 使得數據流非常明確，清楚易懂，歸納的結論就是更新 state 和資料操作請在 owner 擁有者元件裡作，而當子元件的觸發的行為需要更新數據時還是拿父元件的方法。
不過這個方式的缺點就是你需要多打一些字，相較于 two-way binding。雖然 React 也提供一個擴充套件叫做 `ReactLink` 它可以協助您快速做到 two-way binding，不過這篇文章是用來說明整個
React 基礎的觀念，所以我們不打算在這篇提太多額外的東西以免造成混淆。

如果您是著輸入一些條件或者勾起 checkbox 在這上一版的程式碼，您會看到 React 忽略您的輸入。這是故意的，因為我們已經設定 input 的 value 是 this.state.filterText ，他就要確保永遠等於這個參考
讓我們來想想我們希望怎樣，我們希望確保使用者輸入的任何改變都是去更新 `state` ，而 input 則一樣從 state 取得資料。

`FilterableProductTable` 就要把修改 `state` 的函式傳給 `SearchBar`，如此一來當 input 觸發 onChange 時才能變更 state。
雖然這樣聽起來好像會多了不少程式碼，但這能確保資料流向是非常清楚的。

## 最後，就這樣而已
希望這篇文章能夠使您理解關於 React 如何建立元件和應用程式的觀念。
雖然它比起你現在的框架或程式碼的確讓你多打了一些字，不過記住讀程式碼遠遠比撰寫還要困難，而這麼做會讓你的程式碼模組化且非常容易閱讀。

---
layout: post
title: '輕鬆學 Flux'
date: 2014-11-15 20:58:00
categories: Program
tags: [reactjs]
---

# 前言
小弟身為一個資質駑鈍的人，這正是我在學習 Flux 初期最希望有人可以幫我總結的事。服用本篇前須對 React 有基本的認識。
因為底子不好在參透官方範例時一直東奔西跑的查資料一下這個 merge 是什麼意思，一下又怎麼這邊一個 Dispatcher, AppDispatcher 然後又 ActionCreator
總之是你搞得我好亂啊。不過因為最近 React 的盛行讓我得以閱讀許多大大的分享因而有這一篇
<!--more-->


# 我應該使用 Flux 嗎？
如果您的應用程式需要處理很多動態的資料那麼答案是 YES! 您可能應該使用 Flux
但如果您的應用程式只是靜態頁面，且不需要去共用一些應用程式的`狀態`，也從來不需要更新資料那麼這個答案就是 NO
Flux 不能帶給你任何好處

# 為什麼要用 Flux?
Flux 是一個相當複雜的概念，為什麼要增加程式的複雜度呢？哈！開玩笑的！！
百分之九十的 iOS 應用程式透過 table view 來呈現資料。iOS toolkit 擁有非常好的架構來處理關於資料模型的問題，這使得開發起來非常容易。

不過在前端的世界(HTML, Javascript, CSS) 我們沒有那些東西也沒人強迫我們一定得用這些，取而代之的是我們有一個大問題。沒人知道該怎麼完美的處理前端架構這個問題。
處理前端的工作已經好一陣子了，所謂的最佳實踐從來沒有完美的解決所有問題，現實反而是針對個別小問題的函式庫解決了他們
jQuery? Backbone? Handlebars? 其實我們也都知道真正的問題是關於資料，一但它邏輯和 UX 變得越來越複雜就很少人可以精準的控制他。

# 什麼是 Flux?
Flux 是一個 Facebook 創造的術語: 用來描述單一方向的資料流搭配特定的事件和註冊監聽的設計模型。並沒有一定是指 Flux 的函式庫，不過您的確需要 Flux Dispatcher 以及事件函式庫。
官方文件是用一種概念的方式在介紹因此對於像我這種資質比較差的人的確不是個很好的起點。沒幫我分解片段片段程式碼就吸收得很慢。
不過一旦你了解了關於 Flux 的想法您應該就能夠讀懂那些東西。

先不要試圖去比較 Flux 和 MVC 結構，把它們兩者搞在一起的話只會得到混亂。

OK! 談論夠多了，讓我們慢慢的探討，這篇文章將會慢慢解釋所有概念佐以程式碼。

# 1. 你的 Views (React Component) 分派了動作
一個 dispatcher 本質上是一套事件機制。它負責廣播事件和註冊回呼函式(callback)。而且全部就只有一個，一個全域的 dispatcher 物件。
為了讓事情單純你應該就直接使用 Facebook 的 Dispatcher Library 官方在解釋 Dispatcher 那段一開始的確讓我慌了。

~~~js
var Dispatcher = require('flux').Dispatcher;
var AppDispatcher = new Dispatcher();  
~~~

接著我們假設您的應用程式中有一個新增的按鈕功能是加入一個項目到清單中:

~~~js
<button onClick={ this.createNewItem }>New Item</button>  
~~~

當按鈕點擊的時候會發生什麼事情呢？您的 View 派送了一個非常特別的事件，這個事件包含著兩件事`事件名稱`和`該項目的資料`

~~~js
createNewItem: function( evt ) {

    AppDispatcher.dispatch({
        eventName: 'new-item',
        newItem: { name: 'Andy' } // example data
    });

}
~~~

# 2. 您的 Store 需要回應被派送來的事件
就像 Flux 一樣 "store" 也是 Facebook 創造的術語。對於我們的應用程式來說，我們需要一個邏輯集合(就是處理邏輯的物件)與資料來處理這份清單。
這指的就是 Store ，它不只要管理資料模型也需要回應上面提到的`特殊的事件`。在這邊我們就稱它為 ListStore
一個 store 本身是單獨的物件，應該就只有一個，意思是你不應該 new 出另外一個物件。換言之我們的 ListStore 是獨一無二全域的物件。

~~~js
// 一個全域物件用來處理清單資料和邏輯
var ListStore = {

    // 實際資料模型的集合
    items: [],

    // 存取方法，後續我們將使用它來取得資料
    getAll: function() {
        return this.items;
    }

};
~~~

這個 store 接著要回應 dispatcher 發過來的`特殊事件`。

~~~js
var ListStore = …

AppDispatcher.register( function( payload ) {

    switch( payload.eventName ) {

        case 'new-item':

            ListStore.items.push( payload.newItem );
            break;

    }

    return true;

});
~~~

這是典型的 Flux 處理回應 dispatcher 派送過來的 action 的機制。每一個 payload 包含著事件名稱和資料，透過 dispatcher 分派過來。好了我們同時解釋了在官網那張圖上的 action 與 payload 。

> dispatcher.dipatch({}) 發動這個 method => 派送一個 action
> {} 裡面的物件我們稱為 payload

接著用一個 switch 程式片段用來決定該執行什麼動作。

* 核心概念 1: 一個 store 不只是一個資料模型，但其包含著資料模型。
* 核心概念 2: store 在程式中是唯一知道該如何更新資料的角色。這也是整個 Flux 最重要的部分。dispatcher 觸發的事件並不知道如何處理資料。對應回官方的說明這叫做一個 action。
* 對應的行為是寫在 store 然後透過 AppDispatcher.register 註冊。

舉例來說如果有其他部分需要追蹤關於圖片的資料您就應該再開一個 store 並叫做 `ImageStore`。一個 store 只負責處理單一需求。
當您的程式變大的時候可以很輕易地根據需求找到對應的部分。如果程式不複雜可能您只需要一個 store。

記住！只有 store 是被允許註冊 dispatcher 的 callback。View 永遠不會呼叫 `AppDispatcher.register` 。而 dispatcher 也只能夠從 View 送訊息到 store。

# 3. Store 觸發了一個 "Change" 事件
我們幾乎快學完了！現在您的資料確實已經改變了，但是我們需要通知程式中其他角色。
store 接著會觸發一個事件，但不是靠 dispatcher 。這邊通常是初學者容易搞混的地方，但這就是 Flux 採用的方式。這邊我們讓 store 具備有觸發事件的能力
方法有很東種例如使用 [MicroEvent](http://notes.jetienne.com/2011/03/22/microeventjs.html) 或者採用 EventEmitter。這邊為了讓你釐清觀念也不要加入太多東西
所以我們先用 MicroEvent 其觀念就是讓 Store 具備廣播事件的能力，你可能就會問廣播什麼事件？大略你可以先理解成這是一個發佈/訂閱的事件機制。
當 store 告訴全世界: 嘿！我飯煮好了！該吃飯的人就自己自動過來吃 XD。

> 註: 官方採用的方式可能一直在調整，從 merge 到 object-assign 其實觀念都是一樣的就是讓 store 具備廣播事件的能力且官方使用 EventEmitter。

這邊我們就透過 MicroEvent 讓 store 可以通知全世界:

~~~js
MicroEvent.mixin( ListStore );  
~~~

好了！store 已經具備該能力了那就直接在下面呼叫

~~~js
AppDispatcher.register( function( payload ) {

    switch( payload.eventName ) {

        case 'new-item':

            ListStore.items.push( payload.newItem );

            // 告訴其他人我已經改變好了
            ListStore.trigger( 'change' );
            break;

    }

    return true;

});
~~~

核心觀念: 當我們觸發事件時我們不需要再把資料帶出去。view 只需要知道資料已經有更新了。讓我們繼續看下去來理解原理

# 4. View 回應 Change 事件
現在我們需要顯示清單。當清單發生改變，我們的 view 將會全部重新渲染輸出，沒有錯是全部！
為了讓 view 知道何時該更新，從 view 被掛載後它就必須監聽從 store 發出的 `change` 事件。

~~~js
componentDidMount: function() {  
    ListStore.bind( 'change', this.listChanged );
},
~~~

為了簡單起見，我們會使用 `forceUpdate` 強制重新渲染。另一個方法是將整個清單存到 `state`。
在 React 元件內的方法就會如下:

~~~js
listChanged: function() {  
    // Since the list changed, trigger a new render.
    this.forceUpdate();
},
~~~


別忘記當卸載時把監聽清除

~~~js
componentWillUnmount: function() {  
    ListStore.unbind( 'change', this.listChanged );
},
~~~

然後呢？讓我們來看看 `render` 函式，我們特意保留到最後再看

~~~js
render: function() {

    // 記住, ListStore 是全域物件!
    // 透過它取得資料
    var items = ListStore.getAll();


    var itemHtml = items.map( function( item ) {

        return <li key={ listItem.id }>
            { listItem.name }
          </li>;

    });

    return <div>
        <ul>
            { itemHtml }
        </ul>

        <button onClick={ this.createNewItem }>New Item</button>

    </div>;
}
~~~

好了！我們已經完成整個循環。當你加入新的項目 -> view 透過 dispatcher 派送一個 action -> store 回應這個 action 處理資料(處理的 callback 已經被註冊到 dispatcher) -> store 處理完畢觸發 change 事件
-> view 因為有監聽這個事件所以做出對應的處理更新。

不過這邊還有一個問題，每一次我們都重新渲染了整個 view ，這難道不會造成什麼效能異常糟糕嗎？

不會！

沒錯我們的確是讓 render 方法重新在渲染一次，所有在 render 內部的程式碼會重跑一次，不過 React 只會在當資料有所改變的時候才會更新實際的 DOM，關於 render 事實上他只是產生一個虛擬的 DOM。
然後 React 會自動去和上一次的比較，如果兩個虛擬的 DOM 不同的話 React 才會更新實際的 DOM 而且是只有實際 DOM 不同的地方而已。

核心觀念: 當 store 的資料改變 view 不需要知道資料到底是增加還是減少或者修改，view 只要負責重新輸出整個元件，接著 React 的虛擬 DOM 機制會幫你處理如何有效率的更新 DOM。
是不是整個變得很單純。

# 還有一個東西: Action Creator 是什麼鬼？
記得，當我們點擊我們的按鈕時我們派送了一個特殊的事件:

~~~js
AppDispatcher.dispatch({  
    eventName: 'new-item',
    newItem: { name: 'Andy' }
});
~~~

如果很多 view 需要這一個事件，那麼很快這一小段程式碼將到處重複，很快當你需要修改的時候又會搞不清楚。Flux 建議我們將這些派送的事件抽象化，叫做 action creator。
就只是把這些 `AppDispatcher.dispatch` 根據其功能分門別類，這樣其他 view 要用就只要引用就好

~~~js
ListActions = {

    add: function( item ) {
        AppDispatcher.dispatch({
            eventName: 'new-item',
            newItem: item
        });
    }

};
~~~

現在您的 view 就可以單純呼叫 `ListActions.add` 。

希望到這邊為止可以建立起 Flux 的概念，剩下的就在看看官方的範例應該就比較看得懂了。

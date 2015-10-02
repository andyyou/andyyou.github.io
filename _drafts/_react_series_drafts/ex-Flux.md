# Flux
## 概覽
Flux 是一種應用程式的架構，Facebook 使用其建置客戶端的網頁應用程式。
與 React 元件使用的單向數據流相輔相成，它並不是一個普通的框架而是一些設計模式，且您可以直接使用 Flux 不需要任何新的程式碼。
Flux 應用程式有三個主要的部分: Dispatcher, Sotres, Views(即 React 元件)。這些東西不該和 MVC 混淆在一起。首先是 Controller 已經存在 Flux 應用程式中
但是他們是 Controller-View - 即在元件階層結構最頂端，那個用來檢索 Store 資料和往下傳遞給子元素的部分。
另外 action - 是負責調派的輔助方法，先記住他就是協助您呼叫 Dispatcher 的方法，通常被用來加入語意的調派 API，通常被拿來當作 Flux 更新循環的第四部分。
Flux 在支援單向數據流這方面採用了跟 MVC 不同的方式。當使用者跟 React 元件互動時， View 會透過一個位於中心的派送器 dispatcher 來分派 action。
action 會被分派到各個 Store，Store 的工作則是負責處理應用程式中的資料和商業邏輯，Store 同時也要負責去更新相關的 View。
這非常適合在 React 的宣告式編程風格下運作，讓 Store 去觸發更新事件且不需要特別設定關於 View 和 State 之間該如何變換。

例如: 我們想要在訊息列表中顯示一個未讀取訊息的數量，且針對未讀取的訊息標題使用高亮的樣式。這在過去 MVC 架構下其實處理有些繁瑣 - 當一個訊息被讀取馬上更新訊息的資料模型，同時也需要更新未讀取數量的資料
這些相依性和一連串的更新操作在 MVC 架構下是很常遇到的，其後果導致資料流糾結複雜，且很難預測其結果。

反過來讓 Store 來控制: Store 接收更新然後在適當的時機使其他相關聯的事物一致，而不是依賴額外的行為來更新資料。除了 Store ，沒有其他外部的東西知道如何管理資料，這讓我們能做到關注點分離。
這也使得 Store 比起 Model 更俱有可測試性，因為 Store 不直接使用 setter 例如 setAsRead()，取而代之的是它只提供一個輸入點將實際資料(payload)傳入。
換個角度說 Store 不直接處理資料而是把資料透過 dispatcher 來傳遞，然後搭配 action 一起被執行。

## 架構與資料流
資料在一個 Flux 應用程式中遵循單一方向, 這個流程如下:

Views ---> (actions) ----> Dispatcher ---> (registered callback) ---> Stores -------+
Ʌ                                                                                   |
|                                                                                   V
+-- (Controller-Views "change" event handlers) ---- (Stores emit "change" events) --+

一個單向數據流是 Flux 設計模式的中心，而且事實上 Flux 是名字取自拉丁文的『流』。上面的圖表中，dispatcher, store, 以及 view 的輸出與輸入都是獨立的。
action 是用來傳遞資料給 dispatcher 的輔助函式，透過它我們可以根據語義去分類，不要讓所有的事情混在一起。
所有資料都會經過 dispatcher 它就像是一個中央的 hub，或者比喻為電話總機。 action 通常來自于使用者操作了 view，而且僅僅是調用 dispatcher。
接著 dispatcher 會執行 store 註冊的 callback，同時把實際資料包在一個 action 中，實際的一個 payload 如下:

```
{
  source: "SERVER_ACTION",
  action: {
    type: "RECEIVE_RAW_NODES",
    addition: "some data",
    rawNodes: rawNodes
  }
}
```
在已註冊的 callback 中，store 會根據 action type，執行回應的動作。store 接著會觸發 change 事件來通知 controller-view，controller-view 相關的處理事件會被加入監聽並且從 store 取得資料。
最後 controller-view 會透過 setState() 執行 render() 重新渲染輸出。這種結構使得我們可以很容易推論我們的程式，因為我們的資料流只有單一方向，並沒有真實的 Two-way binding。
應用程式的狀態只存在 store ，這讓應用程式不同的部分可以達到低藕合度。凡是跟 store 相依的地方都需要透過 dispatcher 來管理同步更新。
我們發現 two-way binding 會導致聯級更新，意思是改變了一個物件導致另外一個物件改變。當應用程式逐漸增加，這個連動的關係將會變得非常複雜且難以預測。
當更新只能在在單一回合更改資料，那整個系統會變得容易預測。
讓我們更深入得來看看 Flux 各個部分，從 dispatcher 開始會是一個不錯的選擇

## Dispatcher
dispatcher 就像是一個中央的集線器，管理著所有的資料流。本質上它就是 store callback 的註冊表。每一個 store 各自註冊自己的 callback 以提供對應的處理動作
當 dispatcher 送出一個 action 所有 store 都會透過 callback 收到 action 的實際資料。

## Stores
store 包含著應用程式的 state 和邏輯。它們的角色有點類似 MVC 中的 Model，不過他們還需要管理物件的狀態 - 它們並不是物件的實例也不像 Backbone 的集合。

例如 Facebook 的 Lookback Video Editor 使用了一個 TimeStore 來存放時間點和播放的狀態，另一方面 ImageStore 負責保存一系列的圖片。在 TodoMVC 範例中的 TodoStore 也是類似的概念在管理待辦事項的集合
一個 store 同時俱有資料模型集合和特定需求邏輯的資料模型的特色，就如同之前提到的，一個 store 會把自己註冊到 dispatcher 並同時提供一個 callback。
這個 callback 會有一個 payload 的參數，payload 包含著 type 的屬性用來辨別 action 的類型。然後在 store 註冊的 callback 內部會用一個 swtich 去判斷該執行哪些動作。
這使得 action 就可以透過 dispatcher 觸發在 store 中對應的更新 ，在 store 更新之後，他們會廣播一個事件告知狀態已經變更了，接著 view 就可以查詢新的狀態。

## Views 和 Controller-Views
React 提供了一種可組合式的 View 讓我們可以組織我們需要的顯示層。在接近結構頂層的地方有些 View 需要監聽 store 廣播的事件。人們稱之為 controller-view 。
這裡提供了一些中介程式碼讓我們能夠從 store 取得資料接著層層傳遞下去。我們會利用其中一個 controller-view 來幫我們處理頁面的某個部分。
當它收到從 store 來的事件他就會去請求新的資料，接著他就會去使用 setState() 或者 forceUpdate() 來促使 render() 重新輸出。

在單一物件中，我們時常會把整個 store 的狀態傳進 View 裏面，讓內部不同的子元件根據他們的需求取得資料。此外在結構的頂部維持類似 controller 的行為，也因此維持子元件像一個 function
傳入整個狀態也可以有效的減少 props 。

偶而我們可能需要在子元件或內部中加入額外的 controller-view 來讓元件邏輯保持單純一點。這可能有助於封裝階層中一部份特定邏輯。然而要注意在階層內部增加 controller-view
可能會因為增加了新的數據流而違反單一數據的原則，可能會產生潛在的衝突。所以在作出是否增加額外的 controller-view 時，必須要在元件單純原則與複數資料流的複雜性之間取得平衡。
多個資料來源可能會導致 React 因為更新而重複觸發 render() 造成奇怪的影響，反而增加 debug 的困難。

## Action
dispatcher 提供了一個方法讓 view 可以觸發分派任務包含 payload 的資料或 action 到 store。action 的結構被包進一個俱有語意的輔助函式，它是用來把 payload 送到 dispatcher。
舉例來說在 Todo 應用程式中我們想要改變待辦事項的文字，我們可以在 TodoActions 物件中建立一個名為 updateText(todoId, newText) 的方法，這就是一個俱有語意的 action。
這個方法可能會在我們 view(元件) 中的事件處理函式被呼叫，所以我們可以稱它為"回應一個使用者的操作(action)"。這個 action 方法也在 payload 中加入 action type 所以當 payload 在 store 中被解析時可以根據 action type 來選擇適當的程式片段處理 payload 的資料
在 Todo 範例裏可能我們會為 action type 取名為 TODO_UPDATE_TEXT。
action 也可以來自其他地方，例如 Server。例如在初始化資料時。也可能應用在當 Server 返回一個錯誤碼或 Server 需要更新時。

## 關於 Dispatcher
如同之前提到的，dispatcher 也可能用來管理 store 之間相依的關係。要實作這個功能需要透過在 dispatcher 中使用 `waitFor()` 方法，在簡單的 TodoMVC 範例中我們不需要用到它，不過當程式邏輯很複雜的時候可能會用到。
在 TodoStore 已經註冊的 callback 我們可以設定讓 callback 等他其他相依的 callback 先更新完在執行:

```
case 'TODO_CREATE': Dispatcher.waitFor([ PrependedTextStore.dispatcherIndex, YetAnotherStore.dispatcherIndex ], function() { TodoStore.create(PrependedTextStore.getText() + ' ' + action.text); TodoStore.emit('change'); }); break;
```
waitFor() 的參數是一個 dispatcher 註冊索引所組成的陣列，最終 callback 會等待您放入索引的 callback 都完成了才輪到自己。因此 store 透過 waitFor 可以相依于其他 Store。
如果我們建立了一個循環依賴就會產生問題，什麼意思？如果 store A 要等 store B 然後 B 又要等 A 結果是我們製造了一個很糟糕的情況，於是 dispatcher 就會在 console 產生一個錯誤。
不幸的是這點已經超出這份文件的範圍。

如果上面這一小段您無法理解我們提供更多的說明如下:

  Dispatcher 是用來廣播 payload 到已註冊的回呼函式。這跟一般的發佈/訂閱系統有兩個地方不一樣
    1) 回呼函式並不有綁定任何特定事件，每一個 payload 是透過分派到每一個註冊的回呼函式，而 payload 大概長成如下範例:
```
     {
       source: "SERVER_ACTION",
       action: {
         type: "RECEIVE_RAW_NODES",
         addition: "some data",
         rawNodes: rawNodes
       }
     }
```
     2) 回呼函式可以被延遲，直到其他部分或者其他回呼函式被執行完成

  舉例來說，一個假想的航班描述表單，當使用者選定一個國家時它可以自帶預設的城市
```
    var flightDispatcher = new Dispatcher();

    // 記錄哪個國家被選到了
    var CountryStore = {country: null};

    // 記錄哪個城市被選到了
    var CityStore = {city: null};

    // 記錄所選城市的基本票價
    var FlightPriceStore = {price: null}
```
  當一個使用者改變了所選的城市，我們就會 dispatch 派送一個 payload (實際數據包含調用類型)
```
    flightDispatcher.dispatch({
      actionType: 'city-update',
      selectedCity: 'paris'
    });
```
  該實際數據由`CityStore` 處理，所以我們會在 Store 中註冊如何處理的回呼函式:
```
    flightDispatcher.register(function(payload)) {
      if (payload.actionType === 'city-update') {
        CityStore.city = payload.selectedCity;
      }
    });
```
  當使用者選擇了一個國家，我們派送實際數據:
```
    flightDispatcher.dispatch({
      actionType: 'country-update',
      selectedCountry: 'australia'
    });
```
  這個實際數據則是兩者處理:
```
     CountryStore.dispatchToken = flightDispatcher.register(function(payload) {
      if (payload.actionType === 'country-update') {
        CountryStore.country = payload.selectedCountry;
      }
    });
```
  當回呼函式去更新被註冊的 `CountryStore` 時我們儲存一個金鑰的參考，在 waitFor() 使用這個金鑰我們可以保證
   `CountryStore` 在回呼函式更新 `CityStore` 或其他相依這個資料的部分之前先被更新完成
```
    CityStore.dispatchToken = flightDispatcher.register(function(payload) {
      if (payload.actionType === 'country-update') {
        // `CountryStore.country` 此時可能還沒被更新完成
        flightDispatcher.waitFor([CountryStore.dispatchToken]);
        // `CountryStore.country` 此時保證被更新完成

        // 當國家變更後需要設定新的城市當作預設城市
        CityStore.city = getDefaultCityForCountry(CountryStore.country);
      }
    });
```
  `waitFor()` 的使用方式可以被鏈接，舉例來說:
```
    FlightPriceStore.dispatchToken =
      flightDispatcher.register(function(payload)) {
        switch (payload.actionType) {
          case 'country-update':
            flightDispatcher.waitFor([CityStore.dispatchToken]);
            FlightPriceStore.price =
              getFlightPriceStore(CountryStore.country, CityStore.city);
            break;

          case 'city-update':
            FlightPriceStore.price =
              FlightPriceStore(CountryStore.country, CityStore.city);
            break;
      }
    });
```
  `country-update` 將會被保證照下列順序執行 `CountryStore`, `CityStore`, 然後 `FlightPriceStore`.

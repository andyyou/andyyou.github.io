---
layout: post
title: 'React Virtual DOM vs Incremental DOM vs Ember’s Glimmer'
date: 2015-11-30 05:30:00
categories: Javascript
---

這篇文章翻譯自 [React Virtual DOM vs Incremental DOM vs Ember’s Glimmer: Fight](https://auth0.com/blog/2015/11/20/face-off-virtual-dom-vs-incremental-dom-vs-glimmer/)

在這篇文章我們將要探討三個用來動態建立操作 DOM 的技術。當然我們也會測試紀錄其效能並且找出誰比較快。
最後我們當然會分享為什麼我們選擇了其中之一用在我們的專案上。

# 介紹

在網路上有許多關於操作 Document Object Model(DOM) 的框架和函式庫。在他們之中有三個值得我們注意，因為他們的核心專注在提升效能上。
`React.js`, `Ember.js`, 和最近開始崛起的 `Incremental DOM`。 而 React.js 和 Ember.js 提供更多功能不僅僅是建立和更新 DOM
Incremental DOM 只專注在建立 DOM 樹狀結構以及讓他們可以動態更新。下面我們將探索這些框架或函式庫誰比較快一點。

![](https://cdn.auth0.com/blog/dombench/domtree.png)

在我們深入介紹分析之前如果你不是網頁設計師你也許會問什麼是 DOM 操作？
通常網站是透過我們定義不同的元素 element 來構成一個樹狀結構，透過這個樹狀結構的物件來呈現網站，而這些我們稱為元素的東西就是 HTML 規範所定義的那些標籤

透過組織這些元素，開發者可以建立他們想要的網站。而 DOM 就是將一個網站的外觀透過元素來表示，例如我們可以透過 `<input>` 轉換成使用者看到的輸入框
這些標籤元素是 W3C 定義的然後瀏覽器的開發商實作，最後當我們在 HTML 檔案中使用時即會呈現該元素。
也因此我們可以理解隨著我們對於 UIUX 的需求，我們很常會需要動態去更新這些元素，就是當使用者與之互動時產生變化。

除了幫助我們把資料跟視覺呈現的 VIEW 繫結在一起外，上面提到的函式庫通常也會協助我們有效率的去更新 DOM。
一般來說一系列的操作會對應到呼叫一系列的 DOM API 而這些呼叫會自動打包成一個呼叫或者是自動精簡調用的次數。
舉例來說假設來說當我們更新了資料之後我們可能需要

1. 移除該元素
2. 加入新元素
3. 修改新加入元素的屬性

所以我們會直接透過呼叫 DOM API 來執行這些變更。然後網站就會立即的呈現出我們的異動。但是這樣一步一步操作是很浪費效能的。
如果換成透過虛擬模型的機制這些步驟可以被精簡到剩下一步

### 樣板 Templates

一種非常好用且流行的一種方式就是透過`樣板`來建立 DOM 結構。開發者可以使用特定語法來告訴編譯器如何轉換成 DOM 結構即 HTML 文件。
樣板大多看起來就像是增強版的 HTML 就是 HTML 多了一些額外的語法(.erb, .ejs)，也可能完全是另外一種(.jade, .slim)
雖然樣板用起來非常直覺但並不是所有函式庫都偏好這種方式，舉例來說 React 就偏好使用 JSX，算是擴展 JS 的一種語法而不是 HTML，他允許我們在 JS 中插入一些類似 HTML 的語法
另一方面 Ember 則偏好使用 Handlebars 樣板引擎

Incremental DOM 並不偏好特定樣版引擎，雖然說是不偏好特定但是 [closure-templates
](https://github.com/google/closure-templates/) 正默默地在開發中。Incremental DOM 也可以搭配 superview.js, starplate 甚至是 JSX

### React.js 的 Virtual DOM

Virtual DOM 是 React 開發者給這套 DOM 操作引擎取的名字。Virtual DOM 提供一系列方法來讓函式庫知道該如何建立一個在記憶體中的 DOM 樹狀結構
同時也讓其知道該如何更新那些綁定繫結的資料。關於 Virtual DOM 最重要的部分就是其辨識差異的邏輯。一旦模型被改變就會對應到記憶體中的 DOM 副本，接著
這套演算法就會找出最少次數的操作來完成所有的更新 DOM 的動作。

![](https://cdn.auth0.com/blog/dombench/reactdom.png)

##### 優點
* 快速的辨識差異演算法
* 輕量化，在行動裝置上也可以使用
* 非常多媒體，個人 Blog 強力推薦
* 即使沒有 React 也可以使用
* 多個前端能夠使用

##### 缺點
* 所有 DOM 副本都在記憶體中(高記憶體使用)
* 靜態和動態元素沒有分別

另外 React 已經著手在開發偵測那些靜態元素以減少檢查是否需要更新的元素數量


### Ember 的 Glimmer

Glimmer 是 Ember.js 最新渲染引擎的名字。 Glimmer 是 Ember 開發者試圖要在 Ember 中加入 React Virtual DOM 優點但既有的 API 仍然相容的成果
Glimmer 幾乎是全部重寫 Ember 的渲染引擎並且沒有使用任何 Virtual DOM 的程式碼

在 Glimmer 中分成靜態和動態的元件，因此降低了需要確認是否更新的元素，這樣的區隔可以被實現多虧了 Handlebar 的樣板

另外一個 Glimmer 和其他解決方案的關鍵差異在於節點儲存和比對的方式。Glimmer 透過類似串流方式的物件而不是類似於 DOM 節點的物件。
為了找出哪個節點需要被更新，Glimmer 的節點資料會一直跟最新得到的節點資料做比對，如果資料沒有異動就完全不需要執行任何動作

##### 優點

* 快速的差異比對演算法
* 靜態與動態元素區隔
* 完全相容於 Ember 的 API
* 使用記憶體的部分較少

##### 缺點

* 只能夠搭配 Ember 使用
* 只有一個前端能使用

### Incremental DOM

Incremental DOM 試圖要帶來更簡單的方式來處理這個問題，不在記憶體中保存整個 DOM 副本也不產生輕量化的物件型態來儲存樹狀結構的資料。
Incremental DOM 使用的就是原本的 DOM 來找出資料異動的元素。你可能會問為什麼？如果這件事這麼簡單，就不需要有其他的解決方案啦
它其實就是在速度和記憶體中取得一個平衡，Incremental DOM 透過移除多餘的 DOM 副本其結果就可以減少一些記憶體的使用。
在實務上當然在檢查差異的時候也會降低一些速度，降低記憶體的使用主要是為了行動裝置和一些記憶體有限的裝置。

![](https://cdn.auth0.com/blog/dombench/idom.png)

##### 優點

* 降低記憶體的使用
* 簡單的 API
* 容易和許多前端框架整合

##### 缺點

* 並不像其他兩者這麼快，不過這點還有點爭議下面我們會看實際的測試資料
* 較少社群使用

# 效能測試

我們將挑選 dbmonster test app 來做一系列的測試，`dbmonster` 是一個簡單的應用程式它模擬應用程式更新大量資料時的狀況
這個程式是原本是 Ember 團隊用來測試效能的，我們將使用 React, Ember 1.x 和 2.x (都具備 Glimmer) 還有 Incremental DOM
所有的測試都在 Linux (Core i5-5200U CPU) 的 Chromium 46 版本執行。每個測試執行 5 次然後平均

![](https://cdn.auth0.com/blog/dombench/MajorGC.png)

![](https://cdn.auth0.com/blog/dombench/MinorGC.png)

在這兩張圖中顯示記憶體回收的時間，如同預測 Incremental DOM 在這個部分非常有效率
在 MajorGC 的部分非常接近 Incremental DOM 但到了 MinorGC 就突然被拉開了。不過在上兩張圖比較有意思的是關於
Ember 在 1 和 2 之間的改進的程度

![](https://cdn.auth0.com/blog/dombench/LayoutAndPaint.png)

在渲染輸出與繪圖方面 Ember 則表現得比其他人還要突出，Incremental DOM 則因為降低了記憶體影響了速度，在這方面 React 仍然保持第二名且非常接近 Ember

![](https://cdn.auth0.com/blog/dombench/droppedFrameCount.png)

這張圖則顯示出因為長時間停止導致 Chrome 決定要移除的 frames 數量 這個數據越大將會導致有感的閃一下畫面。
這一次 Incremental DOM 再次勝出，因為其花比較少的時間在 GC 意味著更多的時間可以去畫 frames
其他三者差異不大

一個圖表上無法顯示出來比較重要的點是當使用瀏覽器時 Incremental DOM 感覺會比較快因為幾乎是及時回饋，不會閃一下。
觀察其他收集的資料顯示 Incremental DOM 調用 Javascript 呼叫的次數相對少。
不過我們還是得說 Incremental DOM 只是動態更新 DOM 的一套函式庫，而 React 和 Ember 可以處理更多事情，像是事件啊，資料的傳入等等

可以參考一下這個測試的[彙整表](https://github.com/auth0/blog-dombench/blob/master/article_results/results.csv)

# 總結

Virtual DOM, Glimmer, 和 Incremental DOM 在處理動態 DOM 操作與更新這方面都是不錯的選擇。不過因為 React 這陣子較多人在參與以及容易整合
雖然使用較多的記憶體不過這個問題影響卻越來越小因為行動裝置搭載了越來越大的記憶體。Incremental DOM 的確是令人驚艷。
我們期待看到 Incremental DOM 整合到其他函式庫。

React 和 Ember 事實上都取得在實務上的平衡並且二者採用了不同的方式，不過當要挑選一個函式庫時我們傾向優先關注在是否有大量使用者和是否容易整合。
不過凡事還是要謹慎求證，你應該質疑這篇文章所說的並且實際跑一次自己的效能測試。

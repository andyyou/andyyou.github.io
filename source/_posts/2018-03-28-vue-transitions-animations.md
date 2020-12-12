---
title: 'CSS Transitions, Animations 與 Vue'
tags:
  - javascript
  - vue
  - css
categories: Program
date: 2018-03-28 10:21:36
---


# 關於 transitions

要理解 Vue 轉場與動畫的運作，首先我們需要具備 CSS transitions 和 animations 的基本知識。
這邊我們不會完整交代所有的教學，而是重點式的節錄。

讓我們先從 transitions 開始，transitions 中文我們可以稱為轉場，其主要的意思定義 CSS 樣式產生變化時，兩者之間的轉換效果，所謂的轉換效果指的是變化的長度時間，延遲時間，時間控制函式（timing-function）。
舉例來說：

<!--more-->

```scss
// color 在 hover 時轉換，其中 transition 用來設定效果
a {
  color: black;
  transition: all 1s ease;

  &:hover {
    color: cyan;
  }
}
```

transition 相關的設定有 4 + 1 個

* transition-property
* transition-duration
* transition-timing-function
* transition-delay
* transiton (混合寫法)

讓我們先從 `transition` 的格式介紹起，其他 4 個屬性只是分開設定，其效果一樣。

1. transition 一次可以針對 0 個 `none`，1 個 `color` ([參考可設定屬性](http://oli.jp/2010/css-animatable-properties/)) 或 全部 `all` 樣式屬性設定效果。
2. 要設定 2 組以上時使用 `,` 分隔。`transition: color 2s, background-color 5s;`
3. 不指定預設為 `all`
4. timing-function 預設為 `ease`
5. 時間值例如 `1s` 出現的順序很重要，第一個時 `transition-duration`，第二個時 `transition-delay`。
6. `transition-duration` 和 `transition-delay` 預設為 `0s`。

```
transition: [property] [duration];
transition: [property] [duration] [delay];
transition: [property] [duration] [timing-function];
transition: [property] [duration] [timing-function] [delay];
transition: [property_1] [duration_1], [property_2] [duration_2];
transition: all [duration] [timing-function];
```

明白了 transition 的核心用法之後，我們來看看 animation。

# 關於 animation

CSS animation 主要由兩個部分組成：

* 動畫的 CSS 樣式
* `keyframes` 設定起始到結束之間動畫的樣式

類似 `transition`，`animation` 有 8 + 1 個相關屬性。

* animation-name 設定 `@keyframes` 名稱
* animation-duration 動畫一次循環的時間長度
* animation-timing-function 時間控制函式
* animation-delay 延遲時間
* animation-iteration-count 重播次數，也可使用 `infinite` 無限循環
* animation-direction 執行方向，從 0% 播放到 100% ，反過來播放，或一正一反
  + normal 正常
  + reverse 反向
  + alternate 一正、一反...
  + alternate-reverse 一反、一正...
* animation-fill-mode 動畫執行前後套用的樣式設定
  + `none` 不套用
  + `forwards` 使用最後一個 `keyframe` 的樣式，這個屬性受到 `animation-direction` 和 `animation-iteration-count` 影響所以所謂的最後一個 `keyframe` 會有不同結果
  + `backwards` 使用第一個 `keyframe`
  + `both` 同時受 forwards 和 backwards 樣式影響
* animation-play-state 暫停/恢復
* animtion (混合寫法)

> 如果要測試瀏覽器是否支援 animation 請[參考](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Detecting_CSS_animation_support)

上面這些屬性主要是用來調整 `動畫的時間軸` ，而動畫效果（樣式）則使用 `@keyframes` 來定義。

```scss
.slidein {
  animation: 3s slidein;
}

@keyframes slidein {
  0% {
    transform: scaleX(0);
  }
  100% {
    transform: scaleX(1);
  }
}
```

有了基本的知識之後我們可以開始學習 Vue transition 的部分。

# Vue Transition

Vue 在 `插入`，`更新` ，`刪除` 元素的時候，提供了多種套用 transition 效果的方式。這些方式包含：

* 自動加入 class
* 整合第三方的 CSS 動畫函式庫
* 使用 JavaScript 在 transition hook 直接操作 DOM 
* 整合第三方的 JavaScript 動畫函式庫 例如：Velocity.js

## `<transition>` 元件

Vue 提供一個 `transition` 元件，讓我們可以輕鬆的在元素進入（entering）或離開（leaving）的流程中加入轉場效果。

所謂的 entering 和 leaving 具體指的是

* 使用 `v-if` 渲染的過程，出現（entering）消失（leaving）
* 使用 `v-show` 渲染的過程
* 動態元件
* 元件的根節點

下面是一個具體的例子

```html
<div id="demo">
  <button v-on:click="show = !show">
    Toggle
  </button>
  <transition name="fade">
    <p v-if="show">
      Hello, Transition
    </p>
  </transition>
</div>
```

```js
new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
```

```scss
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}
```


當元素被包在 `<transition>` 元件裡面的時候，如果元素被新增，移除，Vue 具體會執行下列行為：

* Vue 會自動偵測目標元素是否套用了 css ，然後在適當的時間點自動加入或移除
* 如果 transition 有設定 `JavaScript hook` 則 hook 事件會在對應的時間點觸發
* 如何沒有 css 和 JavaScript hook 那麼新增或移除的動作會在下一偵（brower animation frame）立刻執行，這個概念和 Vue 的 `nextTick` 是不一樣的


## 自動加入樣式與慣例

自動套用的 class 樣式有一套慣例，Vue 共提供 6 種 class 對應過程中不同的階段

* `v-enter` 當元素進入 DOM 時（顯示/新增元素時，entering），代表 entering transition 開始狀態。class 會在元素加入之前先套用，然後在元素插入完畢後的下一偵移除
* `v-enter-active` 當元素進入 DOM 時（顯示/新增元素時，entering），代表在整個 entering transition 正在運作的狀態。class 會在整個 entering transition 中被套用。class 會在元素被插入之前套用，在整個 transition 或 animation 完成後被移除。這個 class 通常用來定義 duration，delay，timing-function 
* `v-enter-to`  適用於 2.1.8 以上版本，表整個 entering transition 結束的狀態，在元素被加入後，下一偵加入 class，在同一時間點 `v-enter` 會被移除，在整個動畫結束時移除。
* `v-leave` 當元素離開 DOM 時（隱藏/移除元素時，leaving），代表 leaving transition 的起始狀態，在一個leaving transition 被觸發時加入 class，然後下一偵 移除 class
* `v-leave-active` 類似 `v-enter-active` 當元素離開 DOM（隱藏/移除元素時，leaving），代表整個 leaving transition 正在運作的狀態。當 leaving transition 被觸發的同時加入 class，在整個 transition 或 animation 完成後移除 class。同樣的的通常會在這個 class 設定 duration，delay，timing-function。
* `v-leave-to` 適用於 2.1.8 以上版本，表示整個 leaving transition 結束的狀態，當 leaving transition 被觸發後的下一偵加入 class，同個時間移除 `v-leave`，然後當整個 transition 或 animation 完成後移除 class。

這些 class 通常會使用 transition 的 name 來當作前綴，當你使用 `<transition>` 沒有設定 name 的時候，`v-` 則是預設值。假如我們使用 `<transition name="my-transition"></transition>` 則 `v-enter` 的 class 會是 `my-transition-enter`。


## vue + css animations

css animation 套用的方式和 transition 一樣，唯一的差別是 `v-enter` 不會在元素插入之後的下一偵移除，會在 `animationend` 事件移除

* *重點：`v-enter-active` 和 `v-leave-active` 是用來設定 duration, timing-function 的地方，如果是動畫則在這個地方設定 animation-name 等屬性*

* 除了通過 name 搭配慣例來使用 6 種 class 之外也可以之間透過屬性設定自訂 class
  * enter-class
  * enter-active-class
  * enter-to-class
  * leave-class
  * leave-active-class
  * leave-to-class
* 使用自訂 class 的話會覆寫原本的慣例

## transitions + animations

為了知道 transition 何時結束，Vue 需要綁定事件。可以是 `transitionend`或 `animationend` 怎麼使用取決於套用什麼樣式。如果我們只使用其中一種，那麼 Vue 會自己偵測。

不過在某些情況我們可能在一個元素使用 2 種（transition, animation）舉例來說：我們使用 Vue 來觸發一個 CSS animation 然後這個元素還有 hover 時的 transition 效果，在這種情況我們就必須要定義我們想要 Vue 處理哪種類型

在大部分的情況下，Vue 會自動偵測 transition 是否完成，預設 Vue 會等根元素第一個 `transitionend` 或 `animationend` 觸發（同時存在的話會使用 時間比較長的），然而這並不總是我們要的。例如：我們想要編排內部巢狀元素們的 transition 的順序，或讓它們稍微延遲，甚至內部元素 transition 的時間比 root 元素的還長的設定

在這種情況下我們可以使用 `<transition :duration="1000">` 或者 `<transition :duration="{enter: 500, leave: 1000}">`

## JavaScript hooks

transition 和 animation 的核心概念就是在對應的時間點新增，移除 class，enter-active 和 leave-active 用來設定動畫或 transition 的 duration, timing-function 等，enter 和 leave 是開始效果，enter-to 和 leave-to 是結束的效果，事務上 enter 和 leave-to 通常效果一樣，enter-to 和 leave效果一樣。

 除了這種在被動的時間點補上 class 的方式外，Vue 也提供對應時間的 JavaScript hook 讓我們在該時間點使用 JavaScript

```html
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"
  
  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
            
            
  ></transition>
```

* 如果只有用 JavaScript hook 
* enter 和 leave 函式如果沒有 `(el, done)` 的話會自動 call done 結束，如果有參數的話就會卡住等呼叫 `done()`
* 關鍵在有沒有帶參數 `done` 如果想要搭配 css 同步自動 call done 的話記得不要帶 `done` 參數，一旦參數有 `done` 就需要手動調用，css 的 transition 是無法自動同步調用的

## 初始化執行 transition

如果您也想要在第一次渲染元素的時候加入 transition 效果，可以在 `<transition>` 加入 `appear` 屬性

```html
<transition appeear>

</transition>
```

預設會使用已經設定的 entering 和 leaving transition 效果。如果想要初始化時的效果不一樣可以使用如下的參數

```html
 <transition appear appear-class="" appear-to-class="" appear-active-class=""></transition>
```

對應的 JavaScript hook

```html
<transition
	v-on:before-appear=""
  v-on:appear=""
	v-on:after-appear=""
  v-on:appear-cancelled=""
	>
</transition>
```

## 元素之間的 transition

後續我們會討論`元件`之間切換的效果，不過除了元件我們也可以對元素之間套用切換的效果例如使用 `v-if/ v-else` 其中最常見的情況就是一個列表和沒有資料時的切換

```html
<transition>
  <table v-if="items.length > 0">
    
  </table>
  <p v-else>Sorry, no items found.</p>
</transition>
```

使用 `v-if / v-else` 時，如果是上面這種不同 tag 的狀況時是可以正常運作的（兩者會同時執行效果可能不是您想要的效果，下面會有 transition-mode 處理這個問題）

當切換的兩個元素時一樣的時候我們必須讓 Vue 知道如何區分兩者，所以需要給 `key` 。否則 Vue 因為效能的因素只會把元素中的內容換掉。雖然技術上不是必要的但考量到比較好的作法我們在使用 `<transition>` 的時候最好為項目設定 key

```html
<transition>
  <button v-if="isEditing" key="save">
    Save
  </button>
  <button v-else key="edit">
    Edit
  </button>
</transition>
```

在上面這種情況下我們也可以動態 binding 不同的狀態值到 `key` 到同一個元素上，例如

```html
<transition>
	<button v-bind:key="isEditing">
    {{ isEditing ? 'Save' : 'Edit' }}
  </button>
</transition>
```

實務上我們是有機會在任意數量的元素之間切換，或者使用多個 `v-if` ，一個元素 binding 一個動態屬性

```html
<transition>
  <button v-if="docState === 'saved'" key="saved">
    Edit
  </button>
  <button v-if="docState === 'edited" key="edited">
    Save
  </button>
  <button v-if="docState === 'editing" key="editing">
    Cancel
  </button>
</transition>
```

這種時候可以重構成下面這樣

```html
<transition>
  <button v-bind:key="docState">
    {{ buttonMessage }}
  </button>
</transition>
```

```js
//..
computed: {
  buttonMessage: function () {
    switch (this.docState) {
      case 'saved': return 'Edit'
      case 'edited': return 'Save'
      case 'editing': return 'Cancel'
    }
  }
}
```

## transition mode

到這邊我們的切換效果仍然有些問題。舉例上面開關的效果，在從 `on` 按鈕切換到 `off` 按鈕之間，兩個按鈕都會被渲染出來，然後各自執行自己的效果，這是 `<transition>` 預設的行為，entering 和 leaving 會同時執行

有時這樣的行為會運作正常，例如兩個交換的元素使用 `position: absolute;` 。又或者它們使用 `translate`位移的效果，這時同時執行效果就不會很奇怪。

但同時執行有時候不是我們要的效果，因此 Vue 提供了 transition mode

* `in-out` 新元素插入的效果先執行完畢，再處理要移除的元素效果
* `out-in` 移除效果先，新增效果後

## 元件之間的切換

元件之間切換又更簡單了我們甚至不需要 key 屬性，直接使用 `<component>` 來動態切換

```html
<transition>
	<component v-bind:is="view"></component>
</transition>
```

```js
new Vue({
  el: '#root',
  data: {
    view: 'v-a'
  },
  components: {
    'v-a': {
      template: '<div>Component A</div>'
    },
    'v-b': {
      template: '<div>Component B</div>'
    }
  }
})
```

## 列表的 transition 效果

到目前為止我們已經能夠處理的 transition 包含

* 個別的元素
* 多個元素一次只顯示一個（`v-if/v-else`轉場過程的處理）

那麼如何同時處理一個列表（同時出現多個元素），例如使用 `v-for`。這種情況下我們要使用 `<transition-group>`

在看範例之前我們還有些關於 `<transition-group>` 的重點需要先知道

* 在 Vue 的元件裡面需要有一個根節點，在 `<transition>` 時沒有問題，但 `<transition-group>` 不像 `<transition>`，預設外面會包一個 `span` 作為 wrapper，如果需要修改則使用 `tag` 屬性
* 在 `<transition-group>` 裡面的元素一定要有 key

`<transition-group>` 元件還有一個特殊的功能，它不只能夠在 entering 和 leaving 時期加入轉場效果，在其他元素位置改變的時候也可以加入效果。要使用這個新功能只需要認識新的 `v-mode` class，它會在元素產生位置變化替元素加入 class。同樣的跟之前的慣例一樣使用 name 當作前綴還有 `move-class` 的設定屬性。

這個 class 大部分用來設定 timing-function 

> 我們不需要做其他設定 v-move 會自動套用，唯一要注意的是`只有位置移動的元素`才會加上 v-move
>
> 然後 FLIP 技術無法套用在 `display: inline` 的元素上，所以要用 `display: inline-block;` 另外，在移除元素的時候，如果沒有設定 `absoulte` 那麼因為位置沒有改變所以不會有效果。
>
> 參考https://codepen.io/andyyou/pen/geWdYJ 理解 transofrom vs absolute

FLIP 的技術並沒有限制在單一軸的方向使用

## 交錯切換

我們可以透過 binding `data-` 屬性，在 JavaScript transitions 例如：`enter`時間中錯開執行的時間（delay）

## 重複使用 transition

要重複使用動畫效果，我們可以將 `<transition>` 和 `<transition-group>` 封裝進一個元件中在搭配  `<slot>` 即可

```vue
Vue.component('my-sp-transition', {
	template: `
		<transition
                name="very-special-transition"
                mode="out-in"
                @before-enter="beforeEnter"
                @after-enter="afterEnter"
    	<slot></slot>
		</transition>
	`,
  methods: {
		...
  }
})
```

function 化元件尤其適合這種情況。







---
title: "[譯] 理解 React Hooks"
date: 2019-07-29 17:25:12
categories: Program
tags:
  - react
  - javascript
---

## 為什麼要使用 Hooks

我們都知道元件是由上而下傳遞資料，藉此協助我們分解一個複雜的大型介面，利用較小單位的介面功能（元件）組成。這些元件各自是獨立的，可重複使用的。雖然理想上是如此，可是實務上我們卻常常無法拆解複雜的元件，使其變成更小的單元，那是因為其相關的邏輯和狀態無法被提取到其他 function 或元件中。

<!-- more -->

這也是有些開發者會說 React 無法達到關注點分離的原因。

這種情況非常常見，包含動畫的部分，表單處理，連結外部的資料等等。試圖解決這些問題常常會遇到下面幾種結果：

* 大型的元件，難以重構或測試
* 重複的邏輯散落在不同的元件與生命週期中
* 被迫採用複雜的設計模式例如 render props 和 high-order 元件

我們認為 Hooks 是這些問題最好的解決方案。Hooks 讓我們可以組織邏輯的部分，使其成為可以獨立並重複使用的單元。

Hooks 在元件內套用了 React 的哲學（明確定義資料流和可組合的特性），而且不僅僅是組件之間。這也是為什麼 Hooks 可以很自然套用在 React 元件上。

不像其他設計模式例如 render props 或 high-order 元件，Hooks 不會在整個元件樹結構中加入多餘的元件結構，也沒有 mixins 的缺點。

即使在第一次看到這東西的時候內心排斥，但我希望您可以嘗試看看。您應該會喜歡的。



## Hooks 使 React 檔案容量增加？

在我們深入探討 Hooks 之前，您可能擔心在 React 加入 Hooks 概念的部分。這很正常，但我認為短期的學習成本是值得的。

如果 React 社區傾向使用 Hooks，那麼它將會減少一些原本在開發 React 需要克服的問題和學習的概念。Hooks 讓我們只要使用 function 而不需要頻繁的在 function ，class，high-order 元件，render props 之間纏鬥。

就最後實作的容量大小，Hooks 只增加大約 1.5kB。並不是很大。使用 Hooks 通常還可以減少您最後編譯出來的 Bundle 大小，因為 Hooks 通常在優化， minify 的時候會比同樣功能用 class 寫的程式產出更小的 Bundle。

另外，Hooks 沒有包含任何重大變動。就算在您既有的程式碼加入使用依舊可以正常運作。事實上這也是我們推薦的作法，不要大量的改寫已經存在的程式。同樣的我們會很感謝您可以參與 16.7 alpha 提供更多的反饋。



## Hooks 究竟是什麼？

要理解 Hooks 我們需要退一步思考關於程式碼重複的部分。

在 React 應用程式中有很多方式可以重複使用邏輯的部分。我們可以單純使用 funaion，反覆調用。我們也可以使用元件的方式。雖然元件比較多功能但是它們必須要輸出渲染介面（即 `render` 是必須的）。當我們提取元件用來處理一些不需要畫面的功能時會變得有麻煩。這也是為什麼會有這麼多複雜的設計模式出現，例如 render props，high-order 元件。難道 React 不能提供一種通用簡單的方式取代這些東西嗎？

對於重複使用片段程式碼，function 似乎是一個完美的機制。在 funcion 之間移動調整邏輯不需要耗費太多的時間。但是 function 不能使用 React 自身的狀態。您沒辦法在不重構整個 Class 元件或不使用其他設計模式像是 Observables 的情況之下，將一些像是監聽 window size 然後更新 `state` 或者隨著時間變化調整 `state` 的行為提取出來。

這些東西都讓 React 的簡易性大受打擊。

Hooks 確實解決這些問題。Hooks 讓我們可以在 function 中使用 React 的功能（例如 `state`）- 只要調用一個簡單的 function 。React 提供一些內建的 Hooks 讓我們可以使用 React 內部的功能。

由於 Hooks 就是一般的 JavaScript function，您可以組合這些內建的 Hooks 來自訂並擴展 Hooks。這讓您可以將一些複雜的問題用一行程式碼就解決，並且在各個元件中都可以共用又或者分享到社群。

注意，自訂 Hooks 技術上並不是 React 的功能。自訂 Hooks 的功能來自於 Hooks 的設計方式

## 實作範例

例如我們想要讓元件觀察當前瀏覽器視窗 `window` 的寬（舉例來說針對不同的寬顯示不同的內容）。

我們確實可以透過一些方式完成這個功能。不過通常需要使用 class 搭配生命週期方法。如果您想重複使用，甚至需要動用到 render props 或 high-order 元件。

有了 Hook，我相信沒什麼方式比下面範例這樣更好：

```jsx
function MyResponsiveComponent() {
  const width = useWindowWidth(); // Our custom Hook
  return (
    <p>Window width is {width}</p>
  );
}
```

這段程式碼完全符合其陳述。我們在元件中使用了 `window` 的 `width`，接著如果值發生變動 React 會重新渲染元件。這就是 Hook 的目標 - 即使包含 state 或 side effect 都可以讓元件完整陳述性（declarative）。

> Declarative programming - a style of building the structure and elements of computer programs—that expresses the logic of a [computation](https://en.wikipedia.org/wiki/Computation) without describing its [control flow](https://en.wikipedia.org/wiki/Control_flow)

讓我們來看看如何自訂 Hook。我們將使用 React 內部的 `state` 來保存 `window` 的 `width`。然後使用 side effect 的方式來設定 `state`

```jsx
import { useState, useEffect } from 'react';

function useWindowWidth() {
  const [width, useWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    }
  });

  return width;
}
```

如您所見，內建的 React Hook 像是 `useState`，`useEffect` 提供了一些基本的功能。我們可以在元件中直接使用這些功能或者組合它們變成像是 `useWindowWidth`。我們就可以像使用內建的 Hook 一樣使用自訂 Hook。

您可以從[概覽](https://reactjs.org/docs/hooks-overview.html)中學習更多的 Hooks。

Hook 是完全封裝的 - 每次您調用 Hook，會取得一個當前元件內部獨立的 `state` 。它們不是共用相同的狀態，但共用相同的邏輯。

每個 Hook 可能包含一些 `state` 或 side effect 您可以在多個 Hook 中傳入不同的資料，就像您平常使用 function 一樣，可以傳入參數並回傳值。因為它們就是 JavaScript function。

下面是一個動畫的範例：

<iframe src="https://codesandbox.io/embed/sleepy-dijkstra-47dtc?fontsize=14" title="sleepy-dijkstra-47dtc" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
注意到範例中的動畫是靠將不同的參數傳入自訂 Hook 完成的。

```jsx
const [{ pos1 }, set] = useSpring({ pos1: [0, 0], config: fast });
const [{ pos2 }] = useSpring({ pos2: pos1 }, config: slow);
const [{ pos3 }] = useSpring({ pos3: pos2 }, config: slow);
```

在不同 Hooks 中傳入參數使其可以適用於動畫，訂閱，表單管理，和其他狀態。不像 render props 或 high-order 元件，Hooks 不會在您的元件樹結構中建立一個 `return false` 的結構。它們更像是一個扁平化的記憶體儲存單元直接掛載到元件上，也沒有多餘的階層。



## 那麼 Class 呢？

我們很喜歡自訂 Hook 的部分而且為了讓 Hook 運作，React 提供了狀態和 side effect 的處理機制，這也是為什麼要內建 `useState` 和 `useEffect`。

事實證明這些內建的 Hook 並不只是適用自訂 Hook，它們也適用於一般的情景。因此未來我們傾向讓 Hook 變成主流的方式。

但我們並沒有任何計畫要棄用 Class。在 Facebook 我們有成千上萬的元件是用 Class 元件。我們並不想重寫它們。不過如果社群擁抱 Hooks，那麼有兩種方式定義元件的確有些不合理。

Hook 可以涵蓋所有 Class 能處理的情景並且在提取，測試，重用的面向提供更大的彈性。這也是為什麼 Hook 代表了我們對 React 未來的願景。



## Hook 是某種黑魔法?

您可能會對於 [Hook 的規範](https://reactjs.org/docs/hooks-rules.html) 有些驚訝。

雖然 Hooks 必須在元件的上層調用乍看之下很奇怪。即便可以，但您可能也不希望在條件式裡面定義狀態。

如果第一時間覺得很奇怪可以思考一下 - 在過去 4 年裡還沒有 React 開發者抱怨不能在 Class 依據條件定義狀態。

這個設計對於自訂 Hook 不造成額外語法上的混亂和問題起了至關重要的影響。我們認為這個取捨是值得的。

我們已經開始將 Hook 用在產品上幾個月了，也觀察開發者是否會對這些規範感到困惑。我們發現大概幾個小時，人們就會習慣。我承認在一開始我也覺得這些規範感覺像是錯了，不過很快的我就意識到它為什麼存在。

Hooks 並沒有任何黑魔法，如 Jamie 指出的點，它看起來非常類似

```jsx
let hooks = null;

export function useHook() {
  hooks.push(hookData);
}

function reactsInsternalRenderAComponentMethod(component) {
  hooks = [];
  component();
  let hooksForThisComponent = hooks;
  hooks = null;
}
```

每一個元件保存了一個 Hook 列表，並且下次使用 Hook 的時候這個列表會移至下一個項目。

多虧了 Hook 的規範，每次渲染時 Hook 的順序會保持一致，也因此每次調用的時候狀態都會正確。 React 不需要額外的資訊知道哪個元件正在渲染，React 就只是調用該元件。

或許您想知道 React 在哪裡保存了 Hook 的狀態。答案是跟 Class 一樣。React 有一個內部更新佇列，那裡就是唯一的狀態資料來源，不管您是使用哪種方式定義元件。

Hook 不依靠像是 Proxies 或 getters ，因此 Hook 照理來說沒什麼黑魔法，頂多就是 `array.push` 和 `array.pop`。

這個設計讓 Hook 不會只限於 React 才能使用。事實上在提案發表後的幾天，已經有人實驗性為 Vue, Web Component 甚至是單純的 JavaScript 實作 Hook API。

最後，如果您是個純粹函數式編程主義者對於 React 依賴可變動的狀態和實作細節感到不滿意，您也可以利用 *Algebraic effect* （如果 JavaScript 支援）實作一套純函數式的方式。

無論您是從實務派還是理論派的角度來評論分析，我希望至少理由是合理的。如果您還是好奇其他問題，Hooks 提案的作者 Sebastian 也會在 [RFC](https://github.com/reactjs/rfcs/pull/68#issuecomment-439314884) 提供回應。最重要的是我認為 Hook 讓我們可以更輕易的建置元件。



## 散播愛，不炒作

如果上面仍然沒有說服您使用  Hook ，我想我大概也能夠明白。但我依然希望您可以嘗試一次，無論您是否有遇過類似 Hook 試圖想解決的問題，或者您有不同的解決方案歡迎到 RFC 留言。

如果您對於我所說的或者對於 Hook 感到有興趣，那麼這裡需要請您幫個忙。現在有很多人正在學習 React 的路上，如果官方很快的提供教學文件並宣稱這是最佳實踐，那麼他們勢必會感到困惑。並且關於 Hook 還有些東西即便是 React team 也還不是非常清楚。

如果您在撰寫任何關於 Hook 的內容，在還沒穩定的之前麻煩備註這是一個實驗性的功能並且提供官方文件的連結。我們會持續更新。我們也會致力於讓它更全面。

當您在跟其他人談論 Hook 時請保持您的風度。如果您看到任何人誤解，您可以分享您所理解的和相關資源。不過改變是令人感到害怕的，作為一個社群我們應該協助他人而不是選邊站批評。



## 資源


* [Making Sense of React Hooks](https://dev.to/dan_abramov/making-sense-of-react-hooks-2eib)
* [Hooks in react-spring, a tutorial](https://medium.com/@drcmda/hooks-in-react-spring-a-tutorial-c6c436ad7ee4)
* [React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)

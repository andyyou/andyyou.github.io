[原文](http://facebook.github.io/react/docs/why-react.html)

# 為什麼使用 React ？
React 是 Facebook 和 Instagram 用來建置使用者介面的函式庫。近來有許多人考慮使用 React 來處理 MVC 中的 V 的部分。
Facebook 創造了 React 是為了解決構建一個大型且資料不斷變動的應用程式時遇到的問題。
為了達到這個需求，React 採用了兩個主要的核心概念。

## 單純性(Simple)
任何一個時間點您的應用程式都應該傳達同樣的資訊，且當在背後的資料改變的時候 React 會自動管理關於界面 UI 上的更新。

## 定義的方式(Declarative)
這個翻譯有點不是很精準，大略是說您不應該從外部去控制元件該如何更新資料，而是在元件內部定義資料是哪來的怎麼更新。
當資料發生變更的時候，概念上就是 React 點擊了 `refresh` 按鈕，接著元件會自己知道只變更有更新的部分。

# 構建可組合的元件
什麼是元件呢？在 `React` 中所有的東西都是元件。事實上，使用 `React` 就是在建立這些可以重複使用的元件。因此你的目標就是封裝，組件化，重複使用，關注點分離。

> 小弟認為最好實務上的舉例就是類似 WinForm 或 WPF 的控制項(dll)。用起來就像 XAML ，透過屬性傳遞參數，至於程式行為已經都封裝在類別裡面了。

# 給個 5 分鐘看看
React 挑戰了很多傳統的做法，而且第一次大略看到這東西也許會覺得它瘋了嗎。給個 5 分鐘閱讀[官方文件](http://facebook.github.io/react/docs/getting-started.html)：這些概念已經建立了上千個元件且我們應用在 Facebook 和 Instagram。

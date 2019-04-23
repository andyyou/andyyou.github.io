---
title: "Vuepacks 開發筆記"
categories: Program
tags: [rails, javascript, ruby, gem]
---

# RubyGem 檔案名稱命名慣例

在開發 gem 的時候請保持 `lib` 和 `bin` 目錄底下的`主要進入點程式`和`執行檔`檔名要和 gem 名稱一致。這麼作的好處是其他開發者就可以直接 `require 'hola'` 使用該 gem。

下面為一個標準的結構範例：

```bash
├── Rakefile
├── bin
│   └── hola
├── hola.gemspec
├── lib
│   ├── hola
│   │   └── translator.rb
│   └── hola.rb
└── test
    └── test_hola.rb
```

## RubyGem 命名

為您開發的 gem 取個名字是非常重要的，不過在開始之前請先到[RubyGems.org](http://rubygems.org/)和[Github](https://github.com/search)確認名稱是否已經被別人使用了。由於上傳到 RubyGems.org 公開的 gem 名字必須要唯一。這裡提供一些官方的建議：

|**Gem name**|**Require statement**|**Main class or module**|
----
|ruby_parser|require 'ruby_parser'|RubyParser|
|rdoc-data|require 'rdoc/data'|RDoc::Data|
|net-http-persistent|require 'net/http/persistent'|Net::HTTP::Persistent|
|net-http-digest_auth|require 'net/http/digest_auth'|Net::HTTP::DigestAuth|

這些建議的主要目的是讓使用者可以快速取得關於如何使用、匯入這些 gem 檔案的資訊。遵循這些慣例同時也讓 Bundler 在處理匯入您的 gem 時不需要額外的設定。

### 多個單字組成使用 `_` 底線

假如一個 class 或 module 的名稱需要多個單字組成，慣例使用底線將字和字之間分開。

### 擴展功能使用 `-` 中線、連字符號

如果您是要替其他 gem 增加功能，則使用 dash 來命名。通常 dash 在匯入（require）時會對應成 `/`，當然目錄結構也會多一層。
在 class 或 module 方面則會對應成 `::`

### 混合 `_` 底線和 `-` 中線

當您的 gem 名字是由多個英文單字組成時，且這個 gem 也是要幫其他 gem 增加功能。舉例來說：`net-http-digest_auth` 就是要幫 `net/http` 函式庫
增加 HTTP digest authentication (HTTP 摘要驗證) 的功能，而它的名稱為 digest_auth 使用 `_` 區隔兩個單字。於是使用者就要 `require 'net/http/digest_auth'` 這樣匯入使用，而類別 class 則是 `Net::HTTP::DigestAuth`。

### 避免使用大寫英文

OSX 和 Windows 預設檔案系統（Journaled 日誌格式）是不區分大小寫的。

> touch a 和 touch A 只會有一個檔案。

這容易導致使用者誤以為使用大寫也沒問題，一旦在其他區分大小寫的系統格式下就會出錯，為了避免疑義所以統一使用小寫。

# 語意化版號

關於版號策略部分其實就是一套簡單的規則，規定版號該如何分配指定。我們可以單純只用一個數字例如從 1 開始，每次更新都加 1。
也可以非常奇怪例如 Knuth 的 TeX 專案版本：3, 3.1, 3.14, 3.141, 3.1415 每次更新就增加一碼圓周率 PI 的小數數字。

而 RubyGem 鼓勵開發者使用一套語意化版號（Semantic Versioning）的規則來制定版號，雖然這並不強制使用，但使用奇怪的規則可能會讓社區的其他協同開發者困惑。
假設我們有一個 `stack` gem ，裡面有 `Stack` class 這個類別。它具備 `push` 和 `pop` 的功能，如果我們使用語意化版本，那麼 `CHANGELOG` （開發更新日誌）看起來就會像下面一個虛擬的範例這樣：

* 0.0.1 初始化第一版 Stack class 發佈
* 0.0.2 使用連結串列（Linked list）資料結構實作，因為感覺比較厲害
* 0.1.0 新增 `depth` 方法
* 1.0.0 新增 `top`，使 `pop` 回傳 `nil`（舊版 pop 會回傳上一個 top 資料）
* 1.1.0 `push` 現在回傳傳人的值，上一版回傳 nil
* 1.1.1 修正連結串列實作的 bug
* 1.1.2 修正上一版產生的 bug

總結來說：

* 0.0.`x`  `Patch` 修訂號為最細粒度的實作修改，例如小 bug 修正，調整實作
* 0.`x`.0  `Minor` 次版號，向下相容的功能調整，API 參數不變，邏輯行為調整，增加新功能
* `x`.0.0  `Major` 主版號，重大修改，不向下相容的功能

# 相依性宣告

一般來說 gem 會和其他 gems

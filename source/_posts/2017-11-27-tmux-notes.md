---
title: tmux 快速入門筆記
tags:
  - tmux
  - ssh
categories: Tools
date: 2017-11-27 22:26:27
---



# 什麼是 tmux？

![](http://www.hamvocke.com/assets/img/uploads/tmux.png)

<!--more-->

上面您看到的就是一個 `tmux` 的畫面。

`tmux` 的作者解釋：`tmux` 是一個 Terminal Multiplexer。這個看起來很艱澀的術語背後其實概念很簡單; 就是在一個終端機（一個 Console）
下開啟多個視窗，或者分割視窗區塊（在 `tmux` 中我們會有 視窗 `window` 和視窗區塊 `pane` ）。每一個 `pane` 各自有各自獨立執行一個 `Terminal Instance` （各自輸入輸出的介面實例）讓我們可以同時執行多個指令，而不需要開啟多個 Terminal 視窗。如果您是 `iTerm` 的使用者可以把它們概略對應成 `Tab` 和 `Split`。

`tmux` 會在同一個 `session` （連線狀態下的執行環境）保存這些視窗和視窗區塊。我們可以在任何時間離開這個連線。這被稱為 `detaching`。
然後 `tmux` 會繼續維持這個 `session` 直到您把 `tmux server` 砍掉或者重開機的時候。然後重點是我們隨時可以在重新回到（`attaching`）上次離開 `session` 的狀態。

> 原本我們打開一個 terminal 會和機器建立一個 session 當我們關掉視窗時 session 就會關閉，我們剛下的指令就會被中止。使用 tmux 意味著我們是通過 tmux server 來和機器建立 session，我們的操作視窗或視窗區塊則是跟 tmux server 溝通。

> session 概略的說，指的是終端機和主機間建立的一個連線，在這個連線下的執行環境。後續文章將使用 session 。

如果您曾經遠端連線到伺服器工作或使用 ssh 來跟 Raspberry Pi 連線操作，您應該已經猜到這會有什麼幫助。例如：當我們 ssh 斷線的時候，其實對於 tmux 來說只是和 tmux server 中斷，tmux server 會在伺服器端背景繼續執行並保存剛剛的 `session`。要繼續回到剛剛的 session 只要在 ssh 連線回去並在和 session 連線（attach）即可。

到這邊我們大致已經了解了 `tmux` 最基本的兩大功能：視窗管理和 `session` 管理。如果您對於 GUN Screen 已經很熟悉的話這邊對您而言沒有什麼新的東西。
`tmux` 的核心概念就像是一個更輕易使用且更強大的`螢幕`替代工具。

我們已經討論夠多概念的東西了，接著讓我們動手操作。

# 入門

我們將手把手教您該如何安裝與操作 `tmux`。不過這邊我們只會介紹最基本的功能。

## 安裝

在大部分作業系統，安裝 `tmux` 都相當簡單，例如：在 Ubuntu 只要一道指令 `sudo apt-get install tmux`
在 OSX 只要 `brew install tmux` 就可以了。

## 建立新的 session

當我們要建立一個 `session` 時只要輸入 `tmux` 即可

```bash
$ tmux
```

這個指令會建立一個新的 `tmux session`，你會看到下面多了一條`綠色`狀態列。狀態列是 `tmux` 很重要的一個部分。
除了顯示當前的視窗（在左邊）它同時也會顯示一些關於系統的資訊像是時間（在右邊）。狀態列還可以依照需求客製，例如：顯示行事曆，電量等等。

## 分割視窗區塊（Panes）

現在我們已經建立了第一個 `session`，當我們建立 `session` 的時候 `tmux` 預設會開啟一個視窗包含一個單一的視窗區塊。
當我們要執行 `tmux` 的指令時，我們需要先輸入一個前置鍵。預設 tmux 使用 `C-b` 作為前置鍵。`C-b` 指的是 `Ctrl` + `b` 同時按。
接著，要分割區塊使用的指令是 `C-b %` 這個指令會幫我們把當前的畫面切割成垂直左右的兩個視窗區塊。

> 備註： `C-` 指的是 Ctrl + 某個鍵一起按 `M-` 則是 `Alt` 或 `Option` 和某個鍵一起按。

![](http://www.hamvocke.com/assets/img/uploads/tmux_split.png)

如果要水平切割則是 `C-b "`。

## 切換操作的視窗區塊

現在我們的操作被侷限在剛剛新建立的視窗區塊中。不過我們想回到左邊的那個。這個時候我們只要使用 `C-b` 搭配方向鍵即可。
現在您可以親自操作看看。

## 關閉視窗區塊

關閉視窗區塊只需要使用 `C-d`

## 建立視窗

`tmux` 的視窗類似於 Linux 中建立一個虛擬桌面例如：KDE，Gnome 等。

下面我們直接將常用的操作指令列出：

* `C-b c` 建立一個新視窗
* `C-b p` 上一個視窗
* `C-b n` 下一個視窗
* `C-b <number>` 依照編號直接切換（編號顯示於狀態列）

## session 管理

如果您已經完成您的工作，要 detach 只需要 `C-b d`，又或者 `C-b D` 可以選擇要從那個 `session` 離開。要注意的是這些 `session` 依然在背景執行。

要返回剛剛的 `session`，第一步我們需要知道要重新連線那個 `session`。透過使用下面指令可以列出在背景執行的 `session`

```bash
$ tmux ls

0: 1 windows (created Mon Nov 27 17:18:34 2017) [80x25]
1: 1 windows (created Mon Nov 27 17:18:59 2017) [176x23]
2: 1 windows (created Mon Nov 27 17:20:10 2017) [80x23]
```

要連回 session 我們需要指定參數，例如我們要連回第一個 session 則

```bash
$ tmux attach -t 0
```

`-t 0` 這個參數我們透過 `tmux ls` 來取得。

如果您偏好賦予每個 session 一個有意義的名稱，那麼我們可以使用下面這個指令

```bash
$ tmux new -s database
```

此時這個新建的 session 就會被命名為 `database`。除此之外我們還可以修改名稱

```bash
$ tmux rename-session -t 0 database
```

之後當我們要在重新連回該 session 時，只要使用該名稱即可

```bash
$ tmux attach -t database
```

就是這樣，恭喜您已經學完了 `tmux` 的基本功能與操作，當然 `tmux` 還有更多功能，但我們剛剛學習的這些足夠應付大部分的應用。

# 為什麼使用 tmux?

當我完成上面的介紹之後，一個我常見到的反應是：恩，的確看起來很不錯，但為啥我應該用 `tmux` 和這些奇怪的組合鍵呢？為什麼不能就用 `iTerm2` 就好？

恩，沒錯！但我們只需要要單純的視窗管理功能的時候，OSX 的使用者的確只要透過 `iTerm` 的分頁（Tab）功能就好。針對 Linux 使用者則是選擇使用 [Terminator](http://gnometerminator.blogspot.com/p/introduction.html)。

```txt
# 補充 iTerm2 對應效果

Command + d                 垂直分割 Pane
Command + Shift + d         水平分割 Pane
Command + Option + <方向鍵>  切換 Pane

* [iTerm2 教學](https://www.iterm2.com/documentation-one-page.html)
```

那為什麼還要去學這個老舊的東西呢？這邊我將點出幾個 `tmux` 的優點：

* session 的處理：attach 和 detach 協助我們在不同情境和遠端連線的情況下切換，保留 session。
* 跨平台：我們可以在 Mac 底下使用，也可以在 Linux 環境下使用，甚至遠端伺服器或 Raspberry Pi，BeagleBones 等等都可以使用一樣的東西。
* 客製化：我們可以去自訂 tmux 的環境設定並且可以和不同平台環境同步

# 進階

如果您有興趣知道關於 tmux 提供的其他功能，很簡單，輸入 `C-b ?` 您會看到所有有支援的指令，C-c 離開 Help。

到這邊我認為上面的這些資源足夠幫助您自己進一步去探索 tmux。網路上很多像我一樣分享關於他們使用 `tmux` 經驗的文章，您可以找到很多人在 Gihub 分享他們的設定。甚至 [Brian Hogan 還專門為 tmux 寫了一本書](https://pragprog.com/book/bhtmux/tmux)。

現在您自己去探索，嘗試看看。如果您對於自訂修改一些設定有興趣的話可以參考我的另一篇文章[自訂 tmux.conf](http://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)

# 整理

常用的指令紀錄備註

```bash
# 新增
$ tmux
# OR
$ tmux new -s <your_session_name>

# session 列表
$ tmux ls

# 重新連線 session
$ tmux a -t 0
# OR
$ tmux a -t <session_name>

# 刪除 session
$ tmux kill-session -t 0
# OR
$ tmux kill-session -t <session_name>
# OR
$ tmux kill-session -a # 全部

# 刪除 tmux server
$ tmux kill-server

# 重新命名 session
$ tmux rename-session -t 0 <new_session_name>

# 快捷鍵/視窗管理
# C-b ? Help
# C-b c 新增視窗
# C-b， 視窗命名
# C-b w 視窗列表
# C-b f 尋找視窗
# C-b & 刪除視窗
# C-b % 垂直分割區塊
# C-b “ 水平分割區塊
# C-b <方向鍵>
# C-b p 上一個視窗
# C-b n 下一個視窗
# C-b <number> 依照編號直接切換（編號顯示於狀態列）
# C-b d 離開 session
# C-b x 關閉 Pane
# C-d   關閉 Pane
# C-b z 讓一個 Pane 變成全螢幕，在輸入一次則回到剛剛的尺寸
```

# 參考資源

* [A Quick and Easy Guide to tmux](http://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)

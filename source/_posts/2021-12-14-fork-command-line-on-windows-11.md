---
title: "Windows 11(WSL2) 支援 Fork 應用程式 CLI"
date: 2021-12-14 14:37:33
tags:
  - windows
  - wsl
  - ubuntu
categories: System
post_asset_folder: true
---

[Fork](https://git-fork.com/home) 是一套 git 圖形介面工具。如果您曾經在 macOS 環境下開發，那麼您應該不陌生可以使用指令介面就像 `code` 那樣。

<!-- more -->
{% asset_img 1.gif %}


安裝完 Fork 或您希望使用指令方式啟動的應用程式後取得 `exe` 路徑

```
C:\Users\<USERNAME>\AppData\Local\Fork\
```

加入環境變數


{% asset_img 2.png %}

{% asset_img 3.png %}

{% asset_img 4.png %}


開啟 Windows Terminal 執行 `fork.exe` 

確認可以執行之後，可以在您的 `bashrc` 或 `zshrc` 補上 alias

```
alias fork="fork.exe"
```

## 其他

您也可以將一些常用的指令加入

```
alias open="explorer.exe"
```


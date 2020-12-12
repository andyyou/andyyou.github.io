---
title: "Github 協同工作開發流程"
tags:
---

# Github 協同工作開發流程 - 專案團隊方式

## 準備

Github 專案管理者須先將參與開發者之 Github 帳號加入`專案`協同開發成員。

## 開發 與 Pull Request 流程



協同開發成員使用各自的 Github 帳號登入 Github。至目標[開發專案 GitHub 頁面](https://github.com/ab-inbev/APAC_Yedian_Wechat)點擊 Fork。此功能為複製一份隸屬於自己帳戶底下的完整檔案庫(Repository)

![](http://imgur.com/3hG8oA6.png)
![](http://imgur.com/RToDUyC.png)

注意：須有權限的帳號以及確認自己的檔案庫和`原始專案`的差異

![](http://imgur.com/q7zSOax.png)

下載自己的遠端檔案庫之前，例如：之後要以 `development-sogo` 為統一 sogo 小隊的開發分支，先將 fork 檔案庫的 default 設為 `development-sogo` 後續分支切換比較方便。

![](http://imgur.com/nSCqBjW.png)

下載自己的遠端檔案庫

![](http://imgur.com/TCD8pTj.png)

# 操作

```bash
$ git clone git@github.com:<your_account>/<project_name>.git
$ cd <project_name>
# 預設分支是 `development-sogo`，小隊成員的最終推送分支是 ab-inbev:development-sogo

# 開發流程
# 0. 每次送 PR 之前確認檔案更新 ab-inbev 檔案庫的 ab-inbev:development-sogo 分支
#    ab-inbev:development-sogo 分支永遠是小隊的最新程式碼
#    隊長負責在固定時間跟 ab-inbev:development 分支 Merge 或 更新程式碼回來。

# 為了方便更新 加入 ab-inbev 遠端檔案庫
$ git remote add ab-inbev git@github.com:ab-inbev/APAC_Yedian_Wechat.git
# 更新程式碼
$ git pull ab-inbev development-sogo

# 1-1. 不想開分支 -> 直接在 development-sogo 開發
# 1-2. 送 PR 之前更新 ab-inbev:development-sogo
# 1-3. 直接 push 到自己的 <your_account>:development-sogo 
$ git push origin development-sogo
# 1-4. 接著發 PR 給 ab-inbev/development-sogo

# 2-1. 開新分支 
$ git co -b sogo-dev/new-feature
# 2-2. 修改
# 2-3.1. 合併回自己的 <your_account>:development-sogo (2-3.1, 2-3.2 流程可以討論)
# 2-3.2. 直接發到 ab-inbev:development-sogo 
# 2-4. 送 PR 之前更新 ab-inbev:development-sogo
# 2-5. 解衝突
# 2-6. 發 PR 給 ab-inbev:development-sogo
```


# 總結方法

* 加入 Team 直接 clone 專案，開 branch 直接推回主專案，不 Fork。
* 加入 Team Fork 專案，規定統一 branch 

# 開發紀錄

* 2017-07-27 15:00-18:00 
  + 閱讀 Night+ 架構記錄開發流程文件
  + #559 切版 x1



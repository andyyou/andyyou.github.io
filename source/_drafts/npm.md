---
title: npm 筆記
categories: Program
tags: [npm, javascript]
---


```bash
$ npm set init.author.name 'andyyou'
$ npm set init.author.email 'andyyu0920@gmail.com'

$ npm config list

# 設定檔 ~/.npmrc

# Login
$ npm login
# Show current logged user
$ npm whoami 
```

# local 開發時

```bash
# 在 Module/Library 匯出本地模組
$ npm link 
# 移除
$ npm unlink

# 在專案安裝使用
$ npm link <module_name>
```
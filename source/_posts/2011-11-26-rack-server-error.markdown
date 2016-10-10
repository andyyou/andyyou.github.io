---
layout: post
title: '處理 Rack Server Error'
date: 2011-11-26 04:05:00
categories: Program
tags: ruby, RoR
---

## 錯誤訊息：

~~~~~
initialize: Address already in use
~~~~~

<!--more-->

## 解決方式：

查詢程序ID

~~~bash
$ lsof -iTCP:3000
$ lsof |grep 3000
~~~

刪除卡住程序

~~~bash
$ kill -9 00000  # 0000 = ID
~~~

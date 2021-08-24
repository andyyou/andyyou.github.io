---
layout: post
title: '關閉OSX通知時 ICON 跳跳跳的動畫'
date: 2014-08-08 09:08:00
categories: System
tags: [osx]
---
當你在 Mac 上開啓 Line 的時候，由其實開發時常常因為 Line 收到通知導致那個在 Dock 上的 Icon 一直在那邊彈跳(Dock Bouncing) 想把它關掉必須靠指令

<!--more-->

# 關閉

~~~bash
$ defaults write com.apple.dock no-bouncing -bool TRUE
$ killall Dock
~~~

# 開啓

~~~bash
$ defaults write com.apple.dock no-bouncing -bool FALSE
$ killall Dock
~~~

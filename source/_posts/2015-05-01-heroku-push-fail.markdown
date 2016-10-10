---
layout: post
title: 'Heroku 無法 fetch gem'
date: 2015-05-01 05:30:00
categories: Cloud
tags: [ruby, RoR, heroku]
---

當我們在本機設定 gem 的時候有時候會採用直接從 github 下載的方式

~~~ruby
gem 'datetimepicker-rails', github: 'zpaulovics/datetimepicker-rails', branch: 'master', submodules: true
~~~

不過當我們要把程式碼部署到雲上的主機時，有些時候會碰上該機器無法去 fetch repo 的狀況

這個時候請[參考](http://bundler.io/git.html)這邊改變設定即可

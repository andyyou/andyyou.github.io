---
title: 更新 heroku-18
date: 2019-05-02 10:00:17
tags:
  - heroku
  - ruby
  - rails
categories: Cloud
---

因舊有 `Heroku Cedar-14 stack` 將進入 EOL。

<!--more-->

官方提供了[完整的教學](https://devcenter.heroku.com/articles/upgrading-to-the-latest-stack)。
這裡只是簡易的筆記。

```bash
# 查詢
$ heroku plugins:install apps-table
# 個人帳戶
$ heroku apps:table --filter="STACK=cedar-14"
# 團隊
$ heroku apps:table --team=<team name> --filter="STACK=cedar-14"

# Clone & Upgrade
$ heroku stack:set heroku-18 -a <app_name>
$ heroku git:clone -a <app_name>
$ cd <app_name>
# https://devcenter.heroku.com/articles/ruby-support#supported-runtimes
$ git commit --allow-empty -m "Upgrading to heroku-18"
$ git push heroku master

# 如果您的舊站 Rails 跟我一樣是 5.0 ，且您不想動大刀，那麼最快的方式為
# ruby 升至 2.4.6
# rails 5.0.7
# 搭配 http://railsdiff.org/5.0.0/5.0.7.2
```

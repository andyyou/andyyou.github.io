---
layout: post
title: 'Openshift 筆記'
date: 2016-04-16 12:00:00
categories: docker, system
---

{% highlight zsh %}
# 安裝 rhc
$ gem install rhc
$ rhc setup


# 建立 app
$ rhc app create [app name] ruby-2.0 postgresql-9.2

# 設定好 database.yml

# 顯示 app 相關資訊
$ rhc show app [app name]

# DB migration
$ rhc ssh [app name] # SSH 至 app 環境
$ cd app-root/repo
$ bundle exec rake db:create RAIS_ENV=production
$ bundle exec rake db:migrate RAILS_ENV=production

# 修改 Server
$ rhc env set OPENSHIFT_RUBY_SERVER=puma -a [app name]

# 重啟
$ rhc app restart [app name]

# 查看錯誤 Logs
$ rhc tail [app name]

# 查詢 PostgreSQL
$ rhc port-forward -a [app name]
# 接著再用介面上的帳密登入


{% endhighlight %}

{% highlight yml %}
production:
  adapter: postgresql
  encoding: unicode
  pool: 5
  database: < %=ENV['OPENSHIFT_APPNAME']%>
  host: < %=ENV['$OPENSHIFT_POSTGRESQL_DB_HOST']%>
  port: < %=ENV['$OPENSHIFT_POSTGRESQL_DB_PORT']%>
  username: < %=ENV['OPENSHIFT_POSTGRESQL_DB_USERNAME']%>
  password: < %=ENV['OPENSHIFT_POSTGRESQL_DB_PASSWORD']%>
{% endhighlight %}

# 資源

* [完整教學](http://www.sitepoint.com/deploy-your-rails-to-openshift/)
* [解決 rails 4.2 無法啟動 bug](http://www.gerardcondon.com/blog/2015/06/15/upgrading-to-rails-4-dot-2-on-openshift/)
* [database.yml 範例](https://github.com/openshift/rails-example/blob/master/config/database.yml)

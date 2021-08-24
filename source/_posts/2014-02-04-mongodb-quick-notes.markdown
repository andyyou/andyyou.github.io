---
layout: post
title: 'MongoDB Quick Notes'
date: 2014-02-04 21:40:00
categories: Database
tags: [mongodb]
---

## Mac

~~~bash
$ brew update # 更新 brew
$ brew install mongodb # 安裝 MongoDB
$ brew upgrade mongodb # 更新 MongoDB
$ mongod # 啟動 MongoDB Server
$ mongo  # 連線到 localhost:27017/test ，不需帳密。
* 27017 預設 port
* 28017 Web 界面 MongoDB Server 資訊類似 PHP Admin 。

<!--more-->

$ mkdir -p /data/db # 建立資料庫儲存的目錄。
$ chown `id -u` /data/db # 授權當前使用者目錄權限。
$ id -u # 列出當前使用者 uid
$ sudo plutil -p /var/db/dslocal/nodes/Default/users/[YourAccount].plist # OSX 底下的 /etc/shadow 。
$ mongod --dbpath /data/db # 設定資料庫資料目錄。
~~~

基本對照


<table>
	<tr>
  	<th>SQL</th>
    <th>MongoDB</th>
  </tr>
  <tr>
  <td>database</td>
  <td>database</td>
  </tr>
  <tr>
  <td>table</td>
  <td>collection</td>
  </tr>
  <tr>
  <td>row(record)</td>
  <td>documents</td>
  </tr>
  <tr>
  <td>column</td>
  <td> field(key)</td>
  </tr>
  <tr>
  <td>value</td>
  <td>value</td>
  </tr>
</table>


## Mongo shell

* `db` 當前使用的資料庫。
* `show dbs` 資料庫列表。
* `show collections` 資料表清單。
* `use [dbname]` 切換使用目標資料庫。
* `db.getSlibingDB('dbname')` 取得資料庫參考物件。
* `db.dropDatabase()` # 移除資料庫。
* `db.addUser( { user: "<user>", pwd: "<password>", roles: [<roles>] } )` # 加入使用者
* `db.auth('id', 'pwd')` # 登入認證身份。
* `printjson(obj)` 將物件以 JSON 格式輸出。
* `db.createCollection(name, {capped: <Boolean>, autoIndexId: <Boolean>, size: <number>, max <number>} )` # 建立集合，主要的功能是用來開固定式集合，固定式集合不能刪除 Row，但可以移除整個資料表，一旦 `capped` 設為 `true` 就要指定 `size`，但如果要限制則必須在指定 `max` 。一旦超出 `max` 舊的資料就會被覆寫。
* `db.collection.drop()` # 移除資料表。
* `db.collection.ensureIndex({name: [1|-1]})` # 建立索引
* `db.collection.getIndexes()` # 取得索引
* `db.collection.dropIndex({name: 1})` # 移除索引



其他資料
[snapshot](http://nosqldb.org/topic/5181c4b9735345ad0a039dff)
[Performance](http://blog.xuite.net/flyingidea/blog/67641474-%5Bmongodb%5D%E5%A2%9E%E5%8A%A0mongoDB%E6%95%88%E8%83%BD%E7%9A%84%E6%8A%80%E5%B7%A7)
[Munin](http://munin.readthedocs.org/en/latest/installation/index.html)
[Indexes](http://blog.nosqlfan.com/html/3656.html)
[B+Tree](http://www.yhddba.com/%E5%8E%9F%E5%88%9B%E7%90%86%E8%A7%A3b%E6%A0%91%E7%AE%97%E6%B3%95%E5%92%8Cinnodb%E7%B4%A2%E5%BC%95/)
[Distributed System](http://wiki.mbalib.com/zh-tw/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F)
[Aggregate](http://calvert.logdown.com/posts/159915-sql-to-aggregation-mapping-chart)
[SQL 到 MongoDB 對應表 ](http://calvert.logdown.com/posts/159792-sql-to-mongodb-mapping-chart)
[Basic toturial](http://www.w3cschool.cc/mongodb/mongodb-tutorial.html)

---
layout: post
title: 'MongoDB 快速入門筆記'
date: 2014-01-29 13:13:00
categories: Database
---

## 安裝

{% highlight bash %}
$ brew update
$ brew install mongodb
{% endhighlight %}

## 更新

{% highlight bash %}
$ brew update
$ brew ugrade mongodb
{% endhighlight %}

## 啟動 mongodb

{% highlight bash %}
$ mkdir -p /data/db  /* 建立預設 `dbpath` */
$ chown `id -u` /data/db  /* 修改目錄權限 */
$ mongod  /* 啟動 */
$ mongod --dbpath <some alternate directory> /* 使用替代路徑啟動 */
{% endhighlight %}

## 停止
`Control+C`

## 連線資料庫

{% highlight bash %}
$ mongo  # 預設連限制 localhost:27017
$ mongo --hostname 0.0.0.0 --port 27017 # 遠端連線
$ mongo [hostname]:[port]/[dbname] # 遠端連線
{% endhighlight %}

注意：剛建立好時，沒有帳密限制。
如果你只使用 `mongo` 連線至資料庫時，預設會使用 `test` 這個資料庫。在 `mongo shell` 環境下使用 `db` 指令可以尋當前的資料庫。

## 常用指令
* `db` 列出當前的資料庫名稱。
* `show dbs` 列出資料庫清單。
* `use mydb` 切換資料庫。
* `db.auth('account', 'password')` 登入身份。
* `help` 顯示協助訊息。
* `show collections` 列出資料表。
* `mongo <dbname> --eval "db.dropDatabase()"` 移除 database。
或者下面這種方式：

{% highlight bash %}
$ use mydb; 
$ db.dropDatabase();
{% endhighlight %}

* `DBQuery.shellBatchSize = x` 讀取資料時迭代一次取回幾筆資料。
* `printjson()` 將 document 以 JSON 格式輸出。

提醒一個 MongoDB 的機制，包含預設連到的 `test` 或建立的新資料庫，只要裡面沒有資料，MongoDB 並不會永久地保存該資料庫。

## 建立一個集合(collection)並插入一筆資料(document)
在第一次使用 MongoDB 的時候預設會在背後自動建立一個`集合`(collection)或稱為`隱含方式建立`，`collection`對應傳統關聯式資料庫的說法這就是一個 `table`。
參考這篇文章也許可以幫助你快速從關聯式資料庫轉換到 MongoDB [SQL 到 MongoDB 對應表](http://calvert.logdown.com/posts/159792-sql-to-mongodb-mapping-chart)。
在你插入任何一筆資料之前你是不需要自己建立集合。又因為 MongoDB 是動態式架構(schema)。所以在建立一筆資料之前你也不需要事先定義 schema。這樣說可能有 SQL 經驗的人會覺得很抽象詭異。讓我們執行下面的實作例子：

1. 在 mongo shell 環境下，執行 `db`，這時他應該要傳回 `mydb` 。
2. 如果你之前並沒有照著上面文章執行那請使用 `use mydb` 來切換資料庫。
3. 建立兩筆資料(document) `j`，`k`

{% highlight js %}
j = { name : "mongo" }
k = { x : 3 }
{% endhighlight %}

4. 將 j, k 兩筆資料插入 `testData` 集合(資料表)

{% highlight js %}
db.testData.insert(j);
db.testData.insert(k);
{% endhighlight %}

當你建立資料的此刻， mongo 會自動幫你建立集合(testData)並且把資料寫入，此外你也可以透過 `db.createCollection("testData")` 明確的自己建立集合，或者稱之為資料表你會比較習慣。
5. 驗證 `testData` 集合是否存在，列出資料表。

{% highlight bash %}
$ show collections
{% endhighlight %}

此時你應該會看到兩個集合 `system.indexes `，`testData`。 `system.inidexs` 是每個資料庫都會有的集合。
6. 確認 testData 中是否有資料。

{% highlight js %}
db.testData.find();
{% endhighlight %}

這個操作之後你應該會看到

{% highlight js %}
{ "_id" : ObjectId("4c2209f9f3924d31102bd84a"), "name" : "mongo" }
{ "_id" : ObjectId("4c2209fef3924d31102bd84b"), "x" : 3 }
{% endhighlight %}

上面這個操作就等於 SQL 中的 `select`。
`mongo document`或者說每一筆資料都會有 `_id` 他會有一個唯一的 hash 值。上面這些操作並沒有明確指定 _id 的值，所以 mongo 預設會替一個物件產生 id 。
為了讓我們能夠實際的執行測試某些 mongo 指令，在官方文件提供了[產生測試資料的教學](http://docs.mongodb.org/manual/tutorial/generate-test-data/)

{% highlight js %}
for (var i = 1; i <= 25; i++) db.testData.insert( { x : i } )
db.testData.find()
{% endhighlight %}

在一般情況下我們會使用 `db.tableName.insert()` 然而 `db` 指的是目前所在的資料庫 。如果要快速在不同的資料庫，集合中建立資料，官方文件也提供 function 的方式：

{% highlight js %}
function insertData(dbName, colName, num) {
  var col = db.getSiblingDB(dbName).getCollection(colName);
  for (i = 0; i < num; i++) {
    col.insert({x:i});
  }
  print(col.count());
}
{% endhighlight %}

一般 mongo shell 中使用查詢資料，並不是一次傳回所有的資料。 MongoDB 的運作機制是他會先傳回一個指標物件，這個物件一次只會先包含 20 筆資料，如果你要檢視更多資料則輸入 `it` 繼續。而這個 20 筆資料是可以設定的，執行 `DBQuery.shellBatchSize` 可以看到目前的值，要修改的話就給他一個值 `DBQuery.shellBatchSize = 25`。

讓我們直接透過官方教學的步驟來體驗一下 mongo

#### 1. 在 mongo shell 環境下執行

{% highlight js %}
var c = db.testData.find();
{% endhighlight %}

c 會取得一個指標物件。

#### 2. 透過迴圈列出所有的資料

{% highlight js %}
while ( c.hasNext() ) printjson( c.next() )
{% endhighlight %}

另外當你只輸入 `c` 的時候會輸出這個指標物件的內容，接著會提示如果要繼續檢視下面的內容就輸入 `it` 。

#### 3. 使用陣列方式操作指標物件

{% highlight js %}
var c = db.testData.find();
printjson( c [ 4 ] );
{% endhighlight %}

## 查詢指定的記錄(documents)
MongoDB 本身擁有非常豐富的查詢機制讓我們可以過濾找到需要資料。如果你本身已經俱有 SQL 相關技能那可以參考這兩篇教學[SQL 到彙總(Aggregation)對應表](http://calvert.logdown.com/posts/159915-sql-to-aggregation-mapping-chart)[SQL 到 MongoDB 對應表](http://calvert.logdown.com/posts/159792-sql-to-mongodb-mapping-chart)。透過傳遞一個物件參數，我們可以找到 `testData` 中的特定筆資料。例如：

{% highlight js %}
db.testData.find({ x : 18 });
{% endhighlight %}

## 從集合中取得一筆資料

{% highlight js %}
db.testData.findOne();
{% endhighlight %}

## 限制查詢數量
為了程式的效能問題你可以限制每次查詢傳回的資料數量。

{% highlight js %}
db.testData.find().limit(3);
{% endhighlight %}


## 資料文件(documents)
在 MongoDB 中我們稱一筆資料為 documents 類似于關聯式資料庫中的一筆 record，MongoDB 儲存資料的格式類似 JSON，使用鍵值對應。 documents 類似程式語言中的 structures 關聯一個 key 和 value。透過 key 值就可以對應找到 value，在其他程式語言中也有類似功能的東西。例如 dictonaries, hashes, maps 等等。關於這個 document 正式的名稱稱為 BSON。它是一個二進制格式的 JSON 檔案。

## 集合(collections)
接著 MongoDB 把這些 documents 存在一個 collections 裡面。一個 collection 是一個群組用來關聯這些 documents。它們共用一個通用的索引。collections 類似于關聯式資料庫的 table。

# 資料庫操作

## 查詢
讓我們直接透過一個實際的範例來大概了解關於 MongoDB 查詢一筆特定資料的語法：

{% highlight js %}
db.users.find({age: {$gt: 18}}).sort({age: 1})
{% endhighlight %}

`db` 目前使用的資料庫。
`users` 資料表。
`find()` 查詢 = select。
`{age: {$gt: 18}}` 條件 = age 這個欄位要 > 18 。
`sort` 排序。
`{age:1}` 使用 age 這個欄位來排序，`1` 的用意是正向排序 ASC，如果要反向排序則用 `-1` DESC 。

有了整體基本架構的了解之後相信就能快速上手了。

## 資料修改
資料修改包含了 CRUD 的操作就是新增，讀取，更新，刪除。在 MongoDB 中這些操作是針對單一集合。
跟 SQL 一樣更新或刪除一樣是可以指定條件。
例如下面的例子建立一筆資料：

{% highlight js %}
db.users.insert({name: 'Andy', age: 18});
{% endhighlight %}
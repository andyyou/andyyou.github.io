---
title: 正確使用 Laravel Eloquent
date: 2020-09-08 15:05:57
categories: Program
tags:
  - laravel
---

自 Laravel 5.3 之後，看到過許多專案的 Eloquent 關聯與查詢都不是使用相對有效率的用法。這些情況之所以存在是因為有不同的方法可以取得同一個資料集，而且多數看起來非常像。除非您非常了解底層做了什麼，不然判斷哪個方法比較好也不是那麼容易。

<!-- more -->

例如在 `$blog->posts` 和 `$blog->posts()` 之間的選擇。

 `$blog->posts` 會讀取資料庫，讀取跟 `$blog` 相關的文章，然後建立一個 `Post` Model 的 `Collection`。這個操作是很耗費效能的，因爲需要為每個相關的 Model 建立物件實例並賦值。

首先，您應該要先問問自己 - 是否需要 Model 的物件實例？一般來說，您可能只需要幾個欄位的資料，除非您需要欄位自動轉型之類的例如 `Carbon` 日期型別的資料，不然您應該直接從資料庫取得資料就好。

`$blog->posts()` 的話會回傳 `Builder` 而不會直接執行實際查詢。到此則是該再次問問自己 - 我需要整個資料集合嗎？查詢可以在更具體限縮一點嗎？比較好的方式是盡可能在對資料庫查詢多一點操作，在 PHP 少一點。舉例來說 `where` 操作在資料庫會比在 `Collection` 做 `where` 快。

讓我們繼續看一些範例。



### 不要使用 Collection 來計算數量

`$blog->posts->count()` 會先建立 `Post` Model 的資料集合，包含所有屬性欄位資料，然後才回傳 Model 的數量。如果 `posts` 已經載入了，那用這個方式計算當然沒什麼問題，但如果之前沒載入過那就會浪費很多效能。

`$blog->posts()->count()` 則會組合查詢的 SQL 然後從資料庫取得一個數字回來。沒有建立 Model，也沒有屬性。我們就只要數量而已。不是嗎？

假如是一個 5000 筆資料的查詢，第二個作法大概會比第一個作法快 20 倍左右。不過這會跟您的系統環境有關，注意到查詢的數量越大效率的差異會更加極端。



### 如果您只需要第一筆資料，不要讀取整個集合

這是一個顯而易見的錯誤，但我認為需要再提一次。

`$blog->posts->first()` 跟之前提到的一樣，它會讀取並建立整個集合的物件，然後您只需要第一筆。

如果您只需要第一筆資料那麼您應該使用 `$blog->posts()->first()` 如此在查詢中就會使用 `OFFSET` 和 `LIMIT` 而且只會回傳建立一個 Model。

重點是要理解 Model 的方法在調用後回傳了什麼。



### 如果 Builder 能做到，請不要在 Collection 使用 `where` 

`$blog->posts->where('author', 1)` 會先建立所有的 Model，然後遍歷這個集合找到符合條件 `author` 為 1 的 Model，再建立另一個新的集合，最後回傳資料給您。

我們在資料庫使用的任何索引都將無效，因為這裡是靠 PHP 完成查詢的。

`$blog->posts()->where('author', 1)->get()` 的話，所有具體的查詢都是在資料庫完成的，如果有索引的話，會加快效率，並且只會回傳符合的資料，就不會建立多餘的物件。



### 不要在 Collection 使用 `pluck`，使用 Builder 取代

Laravel 專案隨處可見 `pluck`。我們很多時候只是要某個欄位的值，舉例來說我們需要 Blog 的 `post_id` 作為 Key 和 `name` 的資料。

完成上述需求的其中一種方式就是使用 `$blog->posts->pluck('name', 'id')` 但再一次一樣會建立 Collection，也會建立所有列表資料的 Model 物件實例。實際上我們不需要 Model，只需要某個欄位的資料而已，此時我們可以利用 Builder 完成。

另外，`$blog->posts()->pluck('name', 'id')` 看似跟上面幾個點非常像，但行為卻不同。調用 `pluck` 一樣會傳遞給 Builder ，然後執行 `SELECT name, id` 的查詢，接著使用資料庫取得的資料建立一陣列。

前面提到，使用 Builder 不會利用 Model 的方法變更物件實例的欄位資料，因爲根本沒有 Model 的物件實例。

此時如果 `name` 是動態的或者使用 Model 的 `getNameAttribute` 方法變更，那麼利用 Builder 取得資料的 `pluck` 將會和 Model 的資料不一樣。請特別注意。

大多數類似的狀況下，使用 Builder 可能產生很大的不同。因此建議您應該多思考一下關於框架背後執行的操作。

### 參考資料

* [Using Laravel’s Eloquent Efficiently](https://codeburst.io/how-to-use-laravels-eloquent-efficiently-d46f5c392ca8)
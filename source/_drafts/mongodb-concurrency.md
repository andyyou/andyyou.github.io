---
title: 關於 Mongodb 的併發機制
categories: Database
tags: [mongodb, javascript, database]
---

Mongodb 可以讓多個客戶端同時讀取與存取同一筆資料。為了確保一致性，其使用了保護鎖的機制和併發控制監測（concurrency control）來防止多個使用者同時修改同一筆資料。總體來說，這些機制是為了確保發生在同筆資料的寫入行為處於`完成執行`或`不執行`的兩種結果，最終我們需要的就是客戶端不會看到不一致的資料。

# 保護鎖機制

> 開始之前建議您先看看關於[各種鎖的介紹](http://jackyshih.pixnet.net/blog/post/6154337-sql-server-lock-%E6%9E%B6%E6%A7%8B%E8%AE%80%E5%BE%8C%E5%BF%83%E5%BE%97)

Mongodb 使用了 multi-granularity locks 多層級鎖的機制，這使其可以在資料庫或資料表層級執行鎖定，讓各種儲存引擎實作自己資料表層級的併發控制機制。而 `WiredTiger` 引擎可以到單筆資料層級。

Mongodb 使用了 `reader-writer lock` 讓同時讀取的使用者共享同一個資源，例如一個資料庫或資料表，不過在 `MMAPv1` 引擎下只有一個人能執行寫入的操作。
即提供共享鎖模式（S)給讀取操作，獨佔鎖模式（X)給寫入操作，企圖共享鎖（IS）和企圖獨佔鎖（IX）模式則是提供另一個層級的鎖。

舉例來說，當我們要寫入資料時則會鎖定一個資料表（collection）執行 X，同時對應的資料庫和全域會執行 IX，同個資料庫可以同時處在 IS 和 IX 模式，但 X 不可以跟任何模式相容，S 可以和 IS 相容。

![](http://imgur.com/a/9Wbef)

# 參考
* [concurrency](https://docs.mongodb.com/manual/faq/concurrency/)
* [microtime](https://github.com/wadey/node-microtime)
* [Multiple granularity locking](https://en.wikipedia.org/wiki/Multiple_granularity_locking)
* [stream](https://medium.freecodecamp.com/node-js-streams-everything-you-need-to-know-c9141306be93)
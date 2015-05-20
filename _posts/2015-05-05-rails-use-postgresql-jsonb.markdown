# Rails 使用 PostgreSQL jsonb


http://nandovieira.com/using-postgresql-and-jsonb-with-ruby-on-rails

PostgreSQL 9.4 版引入了一個新的資料型別 (column type)
叫做 `jsonb` ，其功能是要讓我們能夠在關聯式資料庫儲存 documents 類型的資料

`jsonb` 和 `json` 型別從功能面看來的確是一樣的東西，但是在底層儲存方式的實作上是不同的

使用 jsonb 的好處是您可以輕鬆的整合關聯與非關聯的資料，充分發揮各自的優勢。例如像 MongoDB documents 的儲存方式具有比較高的效能。 documents 類型的意思就是相關連的資料會被封裝成一個文件檔

# 理解 json 和 jsonb 的差異


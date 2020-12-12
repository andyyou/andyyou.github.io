---
title: 使用 PG copy 升級 Heroku Postgres 資料庫
tags:
  - heroku
  - postgres
categories: Cloud
date: 2017-09-29 15:53:36
---


> 原文：[Upgrading Heroku Postgres Databases](https://devcenter.heroku.com/articles/upgrading-heroku-postgres-databases#upgrade-with-pg-copy)

本文為 Heroku 官方文件的閱讀 + 翻譯筆記，最新的資料請參考官方網站。

這篇文章將會介紹如何升級 Heroku Postgres 資料庫。這裡所要說的是關於變更資料庫的 `plan` （付費方案的部分），以及升級資料庫版本。
關於資料庫的變更只能夠使用 [Heroku CLI](https://cli.heroku.com/) 指令介面來完成。

升級/變更一個正在運作的 Heroku Postgres 資料庫是一件特別需要注意的操作。

Heroku 共有三種變更方式。在所有的情況下，變更資料庫方案時，應用程式會需要停止服務一點時間，此時無法寫入任何資料。

| 升級方式                | 需求說明                                     |
| ------------------- | ---------------------------------------- |
| PG copy             | 可用於所有升級的情況包含從 Hobby 方案變更到其他方案。也可用於升級 Postgres  的版本。 |
| Follower Changeover | 變更正式環境資料庫的使用方案，資料庫版本維持一致。僅可用於 Standard，Premium，Private 或 Enterprise 方案的資料庫。需花費幾小時準備 follower ，期間應用程式仍可運作，切換所需的停機時間小於 1 分鐘。 |
| pg:upgrade          | 升級大型資料庫版本。僅可用於 Standard，Premium，Private 或 Enterprise 方案的資料庫。 |

<!--more-->

# 適用方式

升級的方式取決於下面幾種因素：

* 如果使用的 Postgres 版本在 9.3 之前並且從未使用 `pg:copy`，應使用 `pg:copy` 升級。Postgres 9.3 提供了資料損壞檢核機制，使用 `pg:copy` 將會避免損壞資料，一旦完成 `pg:copy` 便可接續使用 `pg:upgrade` 來升級。
* 如果使用的 Postgres 版本大於 9.3 ，可以直接使用 `pg:upgrade` 升級。
* 如果使用的 Postgres 版本大於 9.3 ，但 [bloat](https://devcenter.heroku.com/articles/managing-vacuum-on-heroku-postgres#determining-bloat) 值過大時，建議使用 `pg:copy`，`pg:copy` 會重新建立資料表和索引，並移除 bloat。但如果停機時間是比較重要的考量點則使用 `pg:upgrade`。


```bash
# 查詢 bloat
$ heroku plugins:install heroku-pg-extras
$ heroku pg:bloat DATABASE_URL --app <app_name>
```

> `pg:upgrade` 針對資料庫為中等數量的關聯與結構的情況，如果您的資料庫包含了上千的資料結構請與 `postgres@heroku.com` 聯繫。

# 使用 PG copy

PG copy 採用的是原始 PostgreSQL 備份與還原的功能，而不是直接把備份檔複製到硬碟上。

## 使用案例與時間

PG copy 大約需要每 GB 3 分鐘的時間，雖然資料量是主要因素，但大型的資料庫方案具備更快速的 I/O。

## 指令

```bash
# 1. 建立新的資料庫
$ heroku addons:create heroku-postgresql:hobby-basic --app <app_name>
# （選用）標準型以上的資料庫需要花費點時間，可使用通知指令
$ heroku pg:wait
# 2. 啟用維護模式
$ heroku maintenance:on --app <app_name>
# （選用）確保沒有任何程序存取資料庫
$ heroku ps:scale worker=0
# 3. 複製資料（備註：HEROKU_POSTGRESQL_SILVER_URL 參數是根據您所建立的新資料庫給的值）
$ heroku pg:copy DATABASE_URL HEROKU_POSTGRESQL_SILVER_URL --app <app_name>
# 4. 切換資料庫
$ heroku pg:promote HEROKU_POSTGRESQL_SILVER_URL --app <app_name>
# 5. 關閉維護模式
$ heroku maintenance:off --app <app_name>
```

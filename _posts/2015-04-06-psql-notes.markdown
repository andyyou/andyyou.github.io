---
layout: post
title: 'Psql 學習記錄'
date: 2015-04-06 20:18:00
categories: Database
---

# 何謂 `psql`
`psql` 是讓我們可以操作 Postgres SQL 的指令介面。其包含著一大票的參數設定，不過這篇記錄只會紀錄我工作時比較常用到的部分

OK! 首先是當我們安裝好 Postgres 之後怎麼連線

# 連線

* -h 代表要連線的 host，以本機來說就是 localhost
* -U 要登入的帳號 使用 --username 等價
* -W 登入帳號的密碼 使用 --password 等價
* -w 不使用密碼
* -p PORT 號 --port 等價 (預設: 5432)
* -d 資料庫名稱 --dbname= 等價

>小提醒：縮寫的參數後的值只需要隔一個空白則可，但全名(--dbname, --username) 可以加上 `=` 或不加。但縮寫不可以加上 `=`

{% highlight bash %}
$ psql -h localhost -U andyyou database_name

# 完整版
psql "dbname=your_dbname host=your_host user=andyyou password=12345678 port=5432 sslmode=require"
{% endhighlight %}

>如果您想使用 GUI 的話 `PGCommander` 可能是目前最好用的工具。不過這邊我們希望把指令學起來，如此當下次需要在遠端伺服器執行任務時就可以派上用場。

# 登入後確認當前使用者

{% highlight bash %}
$ select current_user;
{% endhighlight %}

# 離開

{% highlight bash %}
$ \q

# 或者 CTRL + d
{% endhighlight %}

# 求助
如果記不住指令的時候只要記得下 `--help` 參數即可

{% highlight bash %}
# 指令介面下
$ psql --help

# 已經登入 postgres 之後
$ \?
$ \help
{% endhighlight %}

# 操作資料庫
一般來說只要記住 `\?` 進到指令介面在查要用的指令即可。不過這邊還是把我常用的列出來


{% highlight bash %}
# 修改密碼
$ alter user user_name with password 'new_password';

# 列出所有資料庫
$ \l

# 列出所有 Tables
$ \d

# 顯示表格資訊
$ \d table_name

# 列出 table schema
$ \d+ [tablename]

# 列出使用者
SELECT * FROM "pg_user";

# 列出所有角色 + 權限
$ \du

# 列出該 db schema 與角色的權限
$ \dn+

# 列出當前使用者
SELECT current_user;

# 授權
$ grant all on schema public to public;

# 移除 db (OSX)
$ rm /usr/local/var/postgres/postmaster.pid
$ dropdb rbase_test

# 在 SQL 中移除 db
$ drop database if exists [database name]

# 列出 Table/View 權限
$ \dp

# 建立使用者
$ CREATE USER andy with password 'p@ssw0rd';
$ createuser test --interactive -P # 含建立密碼 + 權限

# 連線資料庫/切換資料庫
$ \c database

# 建立新資料庫
$ CREATE DATABASE mydb;

# 刪除 table
DROP TABLE IF EXISTS table;

# 建立 table 範例
CREATE TABLE films (
  code        char(5) CONSTRAINT firstkey PRIMARY KEY,
  title       varchar(40) NOT NULL,
  did integer NOT NULL,
  date_prod   date,
  kind        varchar(10),
  len         interval hour to minute
); 

# 插入資料
INSERT INTO films VALUES
('UA502', 'Bananas', 105, DEFAULT, 'Comedy', '82 minutes');
 
INSERT INTO films (code, title, did, date_prod, kind)
VALUES ('T_601', 'Yojimbo', 106, DEFAULT, 'Drama'),
('HG120', 'The Dinner Game', 140, DEFAULT, 'Comedy');

# 備份 / 還原 使用 Dump
# 備份
$ pg_dump dbname > outfile
$ pg_dump dbname | gzip > filename.gz
 
# 還原
$ psql dbname < infile
$ createdb dbname && gunzip -c filename.gz | psql dbname
 
# 更新
$ update auth_user set is_superuser = 't' where username='abhiomkar';

# 顯示所有 roles 名稱(SQL)
$ select rolname from pg_roles;

# 查看 table 權限
$ \z

# 移除 db (OSX)
$ rm /usr/local/var/postgres/postmaster.pid
$ dropdb rbase_test

{% endhighlight %}

# pg_hba.conf 在哪裡？

/var/lib/pgsql/data/pg_hba.conf (Linux 預設路徑)
/usr/local/var/postgres/pg_hba.conf (OSX 預設路徑)

# 善用 `find` 指令

* 查詢檔案名稱
{% highlight bash %}
$ find / -name 'pg_hba.conf'
{% endhighlight %}

* 查詢檔案名稱 (不區分大小寫)
{% highlight bash %}
$ find / -iname 'pg_hba.conf'
{% endhighlight %}

* 查詢檔案名稱 (指定目錄)
{% highlight bash %}
$ find /usr -name 'pg_hba.conf'
{% endhighlight %}

* 只搜尋檔案
{% highlight bash %}
$ find /usr -name 'pg_hba.conf' -type f
{% endhighlight %}

* 只搜尋目錄
{% highlight bash %}
$ find /usr -name 'postgres' -type d
{% endhighlight %}

# 忘記密碼的解法

#### 1. 找到 `pg_hba.conf` 

#### 2. 備份一份起來

#### 3. 修改設定 

~~~~~
local all all trust
~~~~~

#### 4. 重啟

一般在 OSX 下我習慣都用 `brew` 可以多裝一套 [brew services](https://robots.thoughtbot.com/starting-and-stopping-background-services-with-homebrew)

{% highlight bash %}
$ brew services start postgresql
$ brew services restart postgresql
$ brew services stop postgresql
{% endhighlight %}

#### 5. 現在任何 User 登可以登入了

{% highlight bash %}
$ psql -U postgres
{% endhighlight %}

#### 6. 重設定密碼

{% highlight bash %}
$ ALTER USER user_name with password 'new_password';
{% endhighlight %}


#### 7. 換回原來設定

#### 8. 重啟

# 無法刪除 db

DETAIL:  There is 1 other session using the database.

麻煩請找找是否有 GUI 軟體或其他 Session 正在連線。

# 無法連線的狀況
錯誤訊息

~~~~~
psql: could not connect to server: Connection refused
  Is the server running locally and accepting
  connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
~~~~~

OSX 環境下解法

{% highlight bash %}
$ rm /usr/local/var/postgres/postmaster.pid
{% endhighlight %}
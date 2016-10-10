---
layout: post
title: 'PHP 環境 OSX 10.9'
date: 2013-10-29 20:04:00
categories: System
tags: php, osx
---

更新 Mavericks 小牛之後會發現 `httpd.conf` 被取代了。不過舊的檔案放在 `/private/etc/apache2/httpd.conf.pre-update`

<!--more-->

# Apache

OSX 內建有 Apache 不過以前可以在 `系統偏好設定`->`共享` 開啟。但是在新版 10.9 這個部分被移除了。
所以要開啟 Apache 必須要使用指令

~~~bash
$ sudo apachectl start 			// 啓動
$ sudo apachectl stop  			// 停止
$ sudo apachectl restart 		// 重啟
$ sudo apachectl -t 			  // 測試 conf 是否正確
{% endhighlight  %}

# 自定 Apache 設定

預設的網站目錄在 `/Library/WebServer/Documents/`。

在開啟 Apache 之後您可能會需要一些設定。
* 建立一個 `/www` 目錄
* 將測試網站檔案放在該目錄底下
* 替工作區加入虛擬目錄
* 設定一個本地端的測試網址

首先建立一個 `/www` 目錄

~~~bash
$ cd /
$ mkdir www   // 這邊可能需要 sudo 權限
~~~

接著建立一個工作目錄

~~~bash
$ mkdir local.example.com
~~~

設定 `http.conf` 這個檔案完整路徑 `/private/etc/apache2/http.conf` 是用來設定關於 Apache 的組態。
這邊您可以使用 vi 或其他編輯器，我使用 Sublime Text 。

~~~bash
$ subl /private/etc/apache2/http.conf
~~~

開啟檔案後有幾個地方要修改如下：
* 170 行將 DocumentRoot 修改為

~~~apacheconf
DocumentRoot "/www"
~~~

* 197 行修改目錄路徑

~~~apacheconf
<Directory "/www">
~~~

* 217 行修改 `AllowOverride` 為 `All`

~~~apacheconf
AllowOverride All
~~~

* 429 行將匯入 `http-vhosts.conf` 註解移除

~~~apacheconf
Include /private/etc/apache2/extra/httpd-vhosts.conf
~~~

我們接著加入關於虛擬目錄的設定，設定檔的路徑在 `/private/etc/apache2/extra/httpd-vhosts.conf` 當你開啟這個檔案的時候會發現裡面已經有一些範例資料。下面是一個設定的範例，實際需求請自行修正。

~~~apacheconf
<VirtualHost *:80>
  DocumentRoot "/www/local.example.com"
  ServerName local.example.com
  ErrorLog "/private/var/log/apache2/local.example.com-error_log"
  CustomLog "/private/var/log/apache2/local.example.com-access_log" common
</VirtualHost>
~~~

當你做完這些修正您可以使用 `sudo apachectl -t` 檢查一下是否有錯誤。
最後我們修改 `/private/etc/hosts` 增加一個本地用的網址

				127.0.0.1      local.example.com

# PHP

PHP 也是已經內建在 OSX 10.9 只要修改 `httpd.conf` 就可以開啟功能。

~~~bash
$ subl /private/etc/apache2/http.conf
~~~

* 118 行 解開載入模組的註解

~~~apacheconf
LoadModule php5_module libexec/apache2/libphp5.so
~~~

* 231 行 加入預設 index.php

~~~apacheconf
DirectoryIndex index.html index.php
~~~

最後呢，我們需要讓 Apache 知道要處理 `.php` , `.phps` 檔案格式。
一樣打開 `http.conf` 在最下面加上程式碼片段

~~~apacheconf
    #PHP Settings
    <IfModule php5_module>
      AddType application/x-httpd-php .php
      AddType application/x-httpd-php-source .phps
    </IfModule>
~~~

別忘記每當你變更 `http.conf` 都要重啟 Apache 才會生效。
接著可以在剛建立的目錄底下建立一個 `test.php`

~~~php
<?php phpinfo(); ?>
~~~

測試環境是否成功。

# MySQL

### 安裝

~~~bash
$ brew install mysql
$ unset TMPDIR
$ mysql_install_db --verbose --user=`root` --basedir="$(brew --prefix mysql)"
$ mysql.server start
$ mysqladmin -u root password 'newpassword'

$ mkdir -p ~/Library/LaunchAgents
$ cp /usr/local/Cellar/mysql/5.5.20/com.mysql.mysqld.plist ~/Library/LaunchAgents/
$ launchctl load -w ~/Library/LaunchAgents/com.mysql.mysqld.plist
~~~

升級也許會覆蓋掉您原本 `php.ini` 的設定，為了取得 MySQL 的一些工作狀態，我們需要修改一些設定

* 478 行 開啟錯誤

~~~~~
display_errors = On
~~~~~

* 986 行 加入 `MySQL socket` 路徑

~~~~~
pdo_mysql.default_socket=/tmp/mysql.sock
~~~~~

參考連結
[Start mySQL server from command line on Mac OS Lion](http://stackoverflow.com/questions/7927854/start-mysql-server-from-command-line-on-mac-os-lion)
[Get Apache, MySQL, PHP and phpMyAdmin working on OSX 10.9 Mavericks](http://www.coolestguidesontheplanet.com/downtown/get-apache-mysql-php-and-phpmyadmin-working-osx-109-mavericks)
[Installing phpMyAdmin on Mac OSX 10.8 Mountain Lion & 10.7, 10.6](http://www.coolestguidesontheplanet.com/installing-phpmyadmin-on-mac-osx-10-7-lion/)
[OS X Mavericks and Apache](http://brianflove.com/2013/10/23/os-x-mavericks-and-apache/)

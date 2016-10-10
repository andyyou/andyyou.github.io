---
layout: post
title: 'OSX EL - Install Apache, MySQL, PHP Note '
date: 2015-10-04 05:30:00
categories: System
tags: php, osx
---

# Use Apache OSX 10.11

OSX 10.11 仍然內建 Apache

<!--more-->

```
$ sudo apachectl start
$ sudo apachectl stop
$ sudo apachectl restart
$ apachectl configtest # 測試設定檔
$ httpd -v # Apache version
```

# 設定 document root

OSX預設有兩個網頁根目錄

* System Level - 所有使用者使用同一個全域根目錄 `/Library/WebServer/Documents/`
* User Level - 每個使用者有自己的根目錄 `~/Sites`

> 從 10.7 之後 Apple 不再提供介面來管理必須要透過指令介面設定, 而使用 User Level 可以避開很多權限的問題所以一般建議使用 User Level

###  User Level 設定

```
$ mkdir ~/Sites
# 加入 `[username].conf`
$ cd /etc/apache2/users
$ sudo vi [username].conf
$ ls -ahl
# -rw-r--r--   1 root  wheel  298 Jun 28 16:47 username.conf
```

`[username].conf` 使用者網站設定

```
<Directory "/Users/username/Sites/">
AllowOverride All
Options Indexes MultiViews FollowSymLinks
Require all granted
</Directory>
```

```
$ sudo vi /etc/apache2/httpd.conf
# 解開註解
# LoadModule authz_core_module libexec/apache2/mod_authz_core.so
# LoadModule authz_host_module libexec/apache2/mod_authz_host.so
# LoadModule userdir_module libexec/apache2/mod_userdir.so
# LoadModule include_module libexec/apache2/mod_include.so
# LoadModule rewrite_module libexec/apache2/mod_rewrite.so
# LoadModule php5_module libexec/apache2/libphp5.so
# Include /private/etc/apache2/extra/httpd-userdir.conf

# 處理另外一個設定檔
$ sudo vi /etc/apache2/extra/httpd-userdir.conf
# 解開註解
# Include /private/etc/apache2/users/*.conf
# 因此, 我們一開始建立的 /etc/apache2/users/[username].conf 就能夠使用

$ sudo apachectl restart # 重啟套用設定

# 造訪 http://localhost/~username/ 即可看到網站, 而網頁檔案放置在 ~/Sites
```

設定檔的 Include 關聯 `/etc/apache2/httpd.conf` 解開 `Include /private/etc/apache2/extra/httpd-userdir.conf`
之後便能夠使用 `/etc/apache2/extra/httpd-userdir.conf` 針對使用者目錄來設定
而 `httpd-userdir.conf` 解開 `Include /private/etc/apache2/users/*.conf` 之後所有在 `/etc/apache2/users`
的 `.conf` 設定就會被套用

## System Level 設定

設定允許 `.htaccess` 來複寫預設值

```
$ sudo vi /etc/apache2/httpd.conf
# AllowOverride All

# 解開註解
# LoadModule rewrite_module libexec/apache2/mod_rewrite.so
# LoadModule php5_module libexec/apache2/libphp5.so
$ sudo apachectl restart
```

# 安裝 MySQL

```
$ brew install mysql
# 需要開機啟用的話
$ ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents

# homebrew suggested, notice before setting need to stop mysqld
$ unset TMPDIR
$ mysql_install_db --verbose --user=`whoami` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp

# 加入設定檔
$ cp -v $(brew --prefix mysql)/support-files/my-default.cnf $(brew --prefix)/etc/my.cnf

# 預設 MySQL 沒有密碼執行下列指令
$ $(brew --prefix mysql)/bin/mysql_secure_installation

# 移除重新安裝
$ brew remove mysql
$ brew cleanup
$ launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
$ rm ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
$ sudo rm -rf /usr/local/var/mysql

# 透過 brew services 操作
# https://github.com/Homebrew/homebrew-services
$ brew tap homebrew/services # install brew services

$ brew services start mysql
$ brew services stop mysql
$ brew services restart mysql
$ brew services list

# 登入 MySQL
$ mysql --host=localhost --user=root -p

# 建立使用者
mysql> CREATE USER '[username]'@'localhost' IDENTIFIED BY '[password]';
mysql> SELECT User,Host FROM mysql.user;
mysql> GRANT ALL PRIVILEGES ON * . * TO '[username]'@'localhost';
mysql> FLUSH PRIVILEGES;

# 設定使用者密碼
mysql> SET PASSWORD FOR '[username]'@'localhost' = PASSWORD('[password_here]');
```

[詳細建立角色權限](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql)

# phpMyAdmin 安裝

[下載 phpMyAdmin](https://www.phpmyadmin.net/downloads/) 解壓縮並移至 document root 目錄

```
$ cd phpmyadmin
$ mkdir config
$ chmod o+w config
# Visit http://localhost/~[username]/phpmyadmin/setup
# After save file move config.inc.php to phpmyadmin folder
```

# 問題

* apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName

[解法](http://blog.miniasp.com/post/2012/06/23/apache2-Could-not-reliably-determine-the-server-fully-qualified-domain-name-using-for-ServerName.aspx)

```
$ sudo vi /etc/apache2/httpd.conf
# 修改 ServerName 隨意指定一個網址
$ apachectl configtest
# > Syntax OK
```


* ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

```
$ ps -ef | grep mysql
$ kill -9 [id]

# Check /tmp/mysql.sock exists
# 檢查設定檔 $(brew --prefix)/etc/my.cnf
$ touch /tmp/mysql.sock # 預設
$ brew services start mysql
```

* Warning: mysqli::mysqli(): (HY000/2002): No such file or directory

```
$ cd /var
$ sudo mkdir mysql
$ ln -s /tmp/mysql.sock mysql/mysql.sock
$ brew services restart mysql
```

[解法](http://stackoverflow.com/questions/4219970/warning-mysql-connect-2002-no-such-file-or-directory-trying-to-connect-vi)


# 參考資源

* [Starting and Stopping Background Services with Homebrew](https://robots.thoughtbot.com/starting-and-stopping-background-services-with-homebrew)
* [Install MySQL on Mac OSX using Homebrew](http://blog.joefallon.net/2013/10/install-mysql-on-mac-osx-using-homebrew/)
* [Get Apache, MySQL, PHP and phpMyAdmin working on OSX 10.11 El Capitan](http://coolestguidesontheplanet.com/get-apache-mysql-php-and-phpmyadmin-working-on-osx-10-11-el-capitan/)

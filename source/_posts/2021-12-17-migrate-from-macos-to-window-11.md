---
title: 從 macOS 轉 window 11 開發者新手指南
date: 2021-12-17 10:43:26
tags:
  - windows
  - wsl
  - ubuntu
categories: System
post_asset_folder: true
---


## 基本設定

* 安裝字體 Operator Mono Lig
* 安裝  VS Code
* [Install WSL](https://docs.microsoft.com/en-us/windows/wsl/install)
* [設定 WSL 開發環境](https://docs.microsoft.com/zh-tw/windows/wsl/setup/environment)

<!-- more -->

```sh
# Windows Terminal 相關

$ sudo apt-get update
$ sudo apt-get upgrade

# 使用檔案總管開啟
$ explorer.exe .
# 複製 ssh keys

$ eval `ssh-agent -s`
$ ssh-add [SSH_KEY]

# C:\ 路徑
$ cd /mnt/c
# Windows 下讀取 Linux 目錄 \\wsl$\

# Windows Terminal setting.json
# C:\Users\[USERNAME]\AppData\Local\Packages\Microsoft.WindowsTerminal_[HASH]\LocalState\settings.json

# sudo 免密碼
$ echo "YOUR_USERNAME ALL=(ALL:ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/will

# 安裝 zsh + oh-my-zsh
$ sudo apt-get install zsh
$ sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

# vim 基本設定
$ git clone https://github.com/amix/vimrc.git ~/.vim_runtime

# 檢查 ubuntu 版本
$ lsb_release -a

# 基本工具
$ sudo apt-get install wget curl unzip

# 列出已安裝套件
$ apt list --installed
```

* [Windows Terminal Themes](https://windowsterminalthemes.dev/)
* [Windows Subsystem for Linux (WSL) 終極開發人員配置 - 2018 版](https://blog.miniasp.com/post/2018/06/15/My-Windows-Subsystem-for-Linux-WSL-Setup-2018)
* [MacType - windows 字體蘋果化](https://github.com/snowie2000/mactype/releases)
  * [各種語言工具可參考 macOS setup 查詢](https://sourabhbajaj.com/mac-setup/)


## 資料庫	

* [資料庫安裝教學](https://docs.microsoft.com/zh-tw/windows/wsl/tutorials/wsl-database)
* [安裝詳細教學](https://j.mp/3yjxr6B)

```sh
# 安裝 Postgre SQL
$ sudo apt install postgresql postgresql-contrib

# 用於檢查您資料庫的狀態
$ sudo service postgresql status
# 以開始執行您的資料庫
$ sudo service postgresql start
# 用來停止執行您的資料庫
$ sudo service postgresql stop
# WSL 開啟自動啟動 Postgre SQL
$ sudo systemctl enable postgresql

# 預設管理員 postgres / secret 設定密碼
$ sudo passwd postgres
# 連線資料庫
$ sudo -u postgres psql
$ psql -h localhost -U postgres [DATABASE]

# 授權
$ grant all on schema public to public;

# 中斷連線
$ \q
# 或者 CTRL + d

# 建立使用者
$ CREATE USER andy with password 'p@ssw0rd';

# 建立新資料庫
$ CREATE DATABASE [DATABASE_NAME];
$ ALTER USER [USERNAME] WITH SUPERUSER;

```

## Nodejs

```sh
# 安裝 nvm
$ sudo apt-get install build-essential libssl-dev

# https://github.com/nvm-sh/nvm

$ nvm ls-remote
$ nvm install node
```

## Docker Desktop

```sh
# Powershell
$ wsl -l -v

# 安裝 Docker Desktop
# https://www.docker.com/products/docker-desktop
```

下載安裝之後，在右下角 Icon 右鍵設定確認**使用 WSL 2 引擎**和**整合**，細節參考[設定 Docker](https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-containers)

* [Remote development in Containers](https://code.visualstudio.com/docs/remote/containers-tutorial)

## PHP 

```sh
# https://www.cloudbooklet.com/how-to-install-php-8-on-ubuntu/
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:ondrej/php
$ sudo apt update
$ sudo apt install php8.0

# (選用) 安裝 composer
$ sudo apt-get install php8.0-cli unzip zip
# 如果 Laravel 缺少函式庫，請補上缺少的。
$ sudo apt-get install php8.0-mbstring php8.0-common php8.0-pgsql php8.0-zip php8.0-gd php8.0-intl php8.0-curl php8.0-xsl php8.0-zip

# Composer 手動安裝
# 官網 https://getcomposer.org/
# 下載教學：https://getcomposer.org/download/
```

## Heroku CLI

如果您直接安裝 windows 版本遭遇下面錯誤

```
@echo: command not found
```

概略是因為部分函式庫沒有安裝 windows 版本，為了讓事情單純可以直接安裝在 WSL 下

```sh
$ curl https://cli-assets.heroku.com/install-ubuntu.sh | sh
```

其他參考﹔[Issue#600](https://github.com/heroku/cli/issues/600)

## 推薦軟體

* Typora
* Adobe GenP
* Fork
* ngrok
* Fences 3
* Seer
* [PowerToys](https://github.com/microsoft/PowerToys)

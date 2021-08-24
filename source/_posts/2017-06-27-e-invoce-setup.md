---
title: 電子發票系統安裝
tags:
  - e-invoice
  - ubuntu
categories: Tools
date: 2017-06-27 17:53:21
---


# 前言

{% asset_img 1.png %}

支援電子發票，營業人必須使用 `Turnkey Client 連線工具` 來執行發票開立、發票作廢、發票註銷、折讓開立、折讓作廢等（傳送/接收發票資料檔案）。

本篇為 `Turnkey` Client on Ubuntu 的安裝筆記。

<!--more-->

## 使用環境需求

### 系統

* Window 或 Linux
* RAM 2GG+
* HD 80G+
* CPU 2+ core
* Linux
  + Ubuntu 10.4+/Redhat ES 5.4 (須含 xwindow)
* Windows
  + WinXP/Win7/Win2003 Server+

### 資料庫

* PostgreSQL 8.0+
* Oracle 10+
* MySQL 5
* MS SQL Server 2003+

### JDK

僅支援 JSK 1.6.x


# 系統安裝

## 安裝 Ubuntu 16.0.4 Server

選擇系統語系，使用英文即可。

{% asset_img 2.png %}

開始安裝

{% asset_img 3.png %}

選擇時區，如果有使用排程要特別注意

{% asset_img 4.png %}
{% asset_img 5.png %}
{% asset_img 6.png %}

設定鍵盤排列類型

{% asset_img 7.png %}

是否偵測鍵盤排列類型

{% asset_img 8.png %}

輸入主機名稱

{% asset_img 9.png %}

輸入使用者名稱/帳號

{% asset_img 12.png %}

設定密碼

{% asset_img 13.png %}

是否要加密使用者目錄: No

{% asset_img 14.png %}

硬碟格式化

{% asset_img 15.png %}
{% asset_img 16.png %}
{% asset_img 17.png %}
{% asset_img 18.png %}
{% asset_img 19.png %}

詢問是否有Proxy設定。可以直接按 Enter 跳過

{% asset_img 20.png %}

是否自動更新

{% asset_img 21.png %}


為了能夠遠端連線，選擇安裝了 OpenSSH Server

{% asset_img 22.png %}

是否要建立開機選單。如果電腦中擁有雙重系統，就要注意一下，因為建立後有可能導致無法進入原本已安裝好的系統。如果 Ubuntu 是電腦中唯一的系統，那就一定要選擇Yes。這樣Ubuntu Server才打的開。

{% asset_img 23.png %}


## VNC 與 XWindow

VNC(Virtual Network Computing) 為一種連線方式，讓我們可以使用 GUI 圖形化介面操作遠端伺服器。VNC 可以讓我們管理遠端主機的檔案，軟體，設定等等。由於 turnky 軟體需要 xwindow 介面，因此為了遠端管理我們需要安裝 VNC 與 XWindow。

## 安裝桌面環境和 VNC Server

大部分的 Linux Server 預設並不會安裝圖形化介面，所以第一步我們需要安裝桌面環境。這裡我們使用輕量化的 xwindow - `XFCE4` 搭配 `TightVNC`

```bash
$ sudo apt-get update
$ sudo apt-get install xfce4 xfce4-goodies tightvncserver
```

接著使用 `vncserver` 指令完成 VNC Server 初始化設定

```bash
$ vncserver
```
設定完存取密碼之後會詢問是否需要建立 `view-only` 密碼。注意如果要設定密碼和上一組存取密碼不可相同。

如果設定錯誤可使用下列指令修改密碼

```
$ vncpasswd
```

## 設定 VNC Server

接下來，我們得告訴 VNC Server 在啟動時該執行那些指令，這些指令會存放在家目錄下 `~/.vnc/xstartup` 的設定檔。在我們第一次執行 `vncserver` 時就被建立了，但我們需要為 Xfce 做些調整。

VNC 第一次設定時預設會使用 5901 port。在 VNC 中則代表 `:1`，例如 `:2` `:3` 分別為 5902，5903 簡單說 `:X` = `5900+X`。
因為我們需要修改設定所以第一步我們要先停止 VNC

```bash
$ vncserver -kill :1
# Killing Xtightvnc process ID 17648
```

在開始設定之前，我們先備份一份設定檔

```bash
$ mv ~/.vnc/xstartup ~/.vnc/xstartup.bak
```

設定檔

```
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```

第一行 `xrdb $HOME/.Xresources` 告訴 VNC 的 GUI framework 讀取 Server 使用者的 `.Xresources` 檔案。這個檔案可以讓使用者調整特定圖形化介面，終端機顏色，字體，等等設定。第二行單純告訴 server 執行 Xfce。

同時為了確保 VNC Server 可以使用這個檔案我們需要修改檔案權限

```bash
$ sudo chmod +x ~/.vnc/xstartup
```

重啓 VNC Server

```bash
$ vncserver
```

指令整理

```bash
# 啟動
$ vncserver

# 停止
$ vncserver -kill :1 # 停止 5901 port
$ vncserver -kill :2 # 停止 5902 port

# 修改密碼
$ vncpasswd

# 設定檔
# ~/.vnc/
```

## 測試 VNC 遠端連線

首先，為了連線安全與方便性我們可以在本地端建立 SSH 轉發連線，就是將 localhost 與遠端 IP 資訊繫結起來。

```
$ ssh -L 5901:127.0.0.1:5901 -N -f -l turnkey <server_ip>
```

之後就可以使用 `localhost:5901` 來連線。

## OSX VNC Client

* [VNC Viewer](https://www.realvnc.com/download/viewer/)

## Browser, Failure to Execute

```bash
$ sudo apt-get install netsurf-gtk

# Application -> Settings -> Preferred Application -> Web Browser -> netsurf

# OR 安裝 Firefox
$ sudo add-apt-repository ppa:ubuntu-mozilla-daily/ppa
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install firefox
```

## 建立 VNC Service File

下一步我們要設定 VNC Server 為系統服務。如此一來就可以像其他服務一樣使用 `service` 指令來啟動、停止、重啓。

建立 `/etc/systemd/system/vncserver@.service`

> 注意 `User` `PIDFile` 的部分要換成您的 username。

```bash
$ sudo vi /etc/systemd/system/vncserver@.service
```

設定檔內容如下

```
[Unit]
Description=Start TightVNC server at startup
After=syslog.target network.target

[Service]
Type=forking
User=turnkey
PAMName=login
PIDFile=/home/turnkey/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target

```

```bash
# 重載新 Unit 設定檔
$ sudo systemctl daemon-reload

# 啟用 Unit
$ sudo systemctl enable vncserver@1.service

# 啟動
$ sudo systemctl start vncserver@1

# 狀態
$ sudo systemctl status vncserver@1
```

# 安裝 JDK

```bash
# 下載 jdk6
# http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html
# 由於 Orcale 下載連結需要登入，可以使用 sftp 或 scp
$ scp ./jdk-6u45-linux-x64.bin turnkey@<ip>:/home/<your_username>

$ chmod u+x jdk-6u45-linux-x64.bin
$ ./jdk-6u45-linux-x64.bin
$ sudo mv jdk1.6.0_45 /opt
$ sudo update-alternatives --install /usr/bin/java java /opt/jdk1.6.0_45/bin/java 1
$ sudo update-alternatives --install /usr/bin/javac javac /opt/jdk1.6.0_45/bin/javac 1
$ sudo update-alternatives --install /usr/bin/jar jar /opt/jdk1.6.0_45/bin/jar 1

# 其他備註 - 安裝 jdk8
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer

# 選擇 java 版本
$ sudo update-alternatives --config java

# 確認版本
$ java -version
$ javac -version
```

# 安裝 PostgreSQL

```bash
$ sudo apt-get install postgresql postgresql-contrib
```

預設，PostgreSQL 使用`角色`的概念來進行驗證與授權。某種程度上類似於 Unix-Style 的帳號，但 PostgreSQL 沒有區分群組和使用者，而是使用角色的方式。

預設安裝之後 Postgre 使用 `ident` 的驗證方式，意味著只要`符合條件` Linux／Unix 系統的帳號都可以存取。例如一個 Postgre 的角色和 Linux/Unix 的 username 一樣就可使用該角色的權限登入。

## 切換 postgres 帳號

預設安裝流程中幫我們建立了一個使用者帳號 `postgres`，這個帳號就可以存取預設 Postgre 的角色。為了使用該角色，現在我們需要切換換系統帳號。

```bash
$ sudo -i -u postgres
# -i 登入
# -u 指定帳號
```

接著，使用 `psql` 存取 Postgre

```bash
$ psql
```

```
# 登入
$ $ psql -h localhost -U andyyou database_name

# 確認當前使用者
psql=# select current_user;

# 離開
psql=# \q

# 幫助 - 指令介面下
psql=# psql --help

# 幫助 - 已經登入 postgres 之後
psql=# \?
psql=# \help

# 直接使用帳號執行 psql
$ sudo -u postgres psql
```

* [psql 入門](http://andyyou.github.io/2015/04/06/psql-notes/)

## 建立角色

要建立角色需要帳號有權限，目前我們只能使用 `postgres`。我們可以使用 `createuser` 指令 搭配 `--interactive` 來建立新角色和資料庫使用者。

如果已經切換帳號 `sudo -i -u postgres`

```bash
# 建立包含密碼的角色（Turnkey 強制需要密碼）
$ createuser --interactive --pwprompt

# > psql
# 列出角色
psql=# \du
# 列出使用者
psql=# SELECT * FROM "pg_user";
# 列出資料庫
psql=# \l
# 顯示連線資訊
psql=# \conninfo
```

如果沒有切換則

```bash
$ sudo -u postgres createuser --interactive --pwprompt
```

接著指令會提供一系列選擇，依據需求來建立帳號和角色。

## 建立新資料庫

除了預設建立的資料庫 `postgres` 外，另一個預設行為就是 PostgresSQL 在登入時預設存取跟帳號同名的資料庫。

假如我們剛剛建立了一個角色叫作 `turnkey` 預設登入時會存取名為 `turnkey` 的資料庫。我們可以使用 `createdb` 來建立資料庫，如果後面沒有加入名稱預設會使用帳號名稱。


```bash
$ sudo -u postregres createdb einvTurnkey
```

## 使用新帳號與角色

基於 `ident` 驗證機制，我們必須要有一個 Linux 的系統帳號來存取角色和資料庫。假如我們沒有符合的 user 可以使用 `adduser` 來建立。

```bash
$ sudo adduser andyyou
$ sudo -i -u andyyou
$ psql
```

如果需要連線到不同的資料庫

```bash
$ psql -d postgres
```

一旦登入成功，我們可以使用下面指令來確認登入資訊

```bash
# port 資訊也在上面
psql=# \conninfo
```

## PostgreSQL 安裝流程 for Turnkey

postgresql 分別有使用者和角色，使用 `createuser` 可以一起建立，使用 `createdb` 可建立資料庫。因為 `ident` 驗證的關係所以我們需要一組系統帳號。

```bash
# Turnkey 安裝流程

# 1. 建立 turnkey role
$ createuser --interactive --pwprompt
# 2. 如果沒有系統帳號
$ sudo adduser turnkey
# 3. 切換帳號
$ sudo -i -u turnkey
# 4. 建立資料庫
$ createdb einvTurnkey
# 設定 /etc/postgresql/<version>/main/postgresql.conf
# 設定 /etc/postgresql/<version>/main/pg_hba.conf
# 5. 重啓服務
$ sudo /etc/init.d/postgresql reload
# 6. 稍後匯入 .sql
$ psql -f PostgreSQL.sql
```

## 中文字 issue

```bash
# 檢視是否支援語系
$ locale -a 
# 如果不支援
$ sudo dpkg-reconfigure locales
# 更新
$ sudo apt-get dist-upgrade
# 安裝字體
$ sudo apt-get install xfonts-intl-chinese
$ sudo apt-get install fonts-arphic-bkai00mp fonts-arphic-bsmi00lp

# （可選）安裝 Google 字體
# 確認系統安裝 fontconfig 套件
$ sudo dpkg -l | grep fontconfig
# 沒有的話則安裝
$ sudo apt-get install fontconfig 
$ mkdir ~/.fonts 
$ cd ~/.fonts
$ wget https://github.com/google/fonts/archive/master.zip
$ unzip master.zip 
$ fc-cache -fv 
```

# 安裝 Turnkey

```bash
# 下載並解壓縮
$ sudo apt-get install unzip
$ wget http://117.56.24.205/EINVTurnkey1.4.8-linux.zip
$ unzip EINVTurnkey1.4.8-linux.zip
$ cd ~/EINVTurnkey1.4.8-linux/

# 1. 匯入 sql (本文使用關聯式資料庫)
$ psql -f DBSchema/PostgreSQL/PostgreSQL.sql

# 注意下列指令須在 xwindow 下執行
# 備註：Application -> settings -> window manager -> keyboard -> Super + Tab Clean -> Fix tab issue
# 2. 設定 Turnkey
$ cd linux
$ chmod +x runFirst.sct einvTurnkey.sct
$ ./runFirst.sct
$ ./einvTurnkey.sct

# 預設密碼：ADMIN

# 注意：這些 scripts 包含相對路徑，所以不可使用 profile。
# runFirst.sct： 執行環境設定
# einvTurnkey.sct: 執行 Turnkey
# runUpgrade.sct: 執行更新程式
# setenv: 設定環境變數
# StartBatchEINVTurnkey.sct: 啟動 Turnkey 背景執行功能
# StopBatchEINVTurnkey.sct: 關閉 Turnkey 背景執行功能
```

到這邊我們已經完成 Turnkey 的安裝，但還沒完成設定。

# 軟體憑證 PFX

說明：使用 Turnkey 需要使用憑證認證，硬體（卡片）或軟體憑證 PFX。

* [官方文件](https://www.einvoice.nat.gov.tw/APTRNKY/einvoice_softcert_user_manual.pdf)
* [官方網站](https://www.einvoice.nat.gov.tw/APTRNKY/PFXTool.htm)

## 流程

1. 至[PFX 網頁](https://www.einvoice.nat.gov.tw/APTRNKY/PFXTool.htm)產生`csr`
2. 憑證申請
  + [GTestCA 測試憑證申請(工商測試憑證 - 公司及分公司
)](http://gtestca.nat.gov.tw/03-02.html)
  + [經濟部工商憑證管理中心](http://moeaca.nat.gov.tw/nonic.html)

> 正式憑證請參考官方文件

## 測試憑證申請步驟

連線至[PFX 網頁](https://www.einvoice.nat.gov.tw/APTRNKY/PFXTool.htm?CSRT=11407652690091276046 )

{% asset_img pfx-1.png %}

產生 csr

{% asset_img pfx-2.png %}

備份到 txt

{% asset_img pfx-4.png %}

連線至[GTestCA 測試憑證申請 - 我要申請非 IC 卡類測試憑證](http://gtestca.nat.gov.tw/03-02.html)，新版網站與官方的文件圖片不同。

{% asset_img pfx-5.png %}

可以使用 `工商測試憑證 - 公司及分公司` 或 `一站式專屬測試`，但要注意 `CA金鑰與簽章演算法` 不可以選`新`的。

{% asset_img pfx-6.png %}
{% asset_img pfx-7.png %}

如果憑證無法使用則匯出 base64 格式

{% asset_img pfx-8.png %}
{% asset_img pfx-9.png %}
{% asset_img pfx-10.png %}
{% asset_img pfx-11.png %}
{% asset_img pfx-12.png %}

貼上 base64 的編碼之後產生憑證。

{% asset_img pfx-13.png %}
{% asset_img pfx-14.png %}

## 備份金鑰

{% asset_img pfx-15.png %}
{% asset_img pfx-16.png %}
{% asset_img pfx-17.png %}
{% asset_img pfx-18.png %}
{% asset_img pfx-19.png %}
{% asset_img pfx-20.png %}
{% asset_img pfx-21.png %}
{% asset_img pfx-22.png %}
{% asset_img pfx-23.png %}

## 上傳憑證至主機

```bash
$ sftp turnkey@<ip>
$ put <pfx>
```

## Turnkey 設定

最後開啟 turnkey 將 `系統管理` 與 `檔案接收` 逐步設定。

# 參考

* [電子發票介紹](https://www.google.com.tw/url?sa=t&rct=j&q=&esrc=s&source=web&cd=7&cad=rja&uact=8&ved=0ahUKEwj4hfabwdDUAhXCwLwKHWACAbEQFghBMAY&url=https%3A%2F%2Fwww.ntbca.gov.tw%2Fetwmain%2Fdownload%3Ffname%3DETWF001_20140620122918642.pdf&usg=AFQjCNHa7w2ZUXHf_rNWC37ZOmnKW5oWtg)
* [財政部電子發票整合服務平台](https://www.einvoice.nat.gov.tw/)
* [二代電子發票平台](https://www.einvoice.nat.gov.tw/APTRNKY/index.html)
* [VNC 安裝教學](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-16-04)
* [Ubuntu 安裝 JDK](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-ubuntu-with-apt-get)
* [Ubuntu 安裝 PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
* [Ubuntu JDK 1.6](https://gist.github.com/senthil245/6093389)
* [Debian 支援中文](http://wiki.debian.org.hk/w/Make_Debian_support_Chinese)
* [Ubuntu 安裝 Google 字體](https://www.phpini.com/linux/ubuntu-install-google-fonts)
* [jdk 6 安裝](https://community.nxp.com/docs/DOC-98441)



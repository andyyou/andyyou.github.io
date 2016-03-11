---
layout: post
title: 'RaspberryPi with node 安裝筆記'
date: 2016-03-08 19:00:00
categories: web
---

安裝完 NOOBS 之後...

# 設定

{% highlight bash %}
> raspi-config # 設定語系，時區
> startx # 開啟圖形化介面

# 更新軟體
> sudo apt-get -y update
> sudo apt-get -y upgrade
> sudo apt-get -y dist-upgrade

# 更新韌體
> rpi-update
> reboot

# 安裝遠端桌面
> sudo apt-get install xrdp # Microsoft Remote Desktop 即可連線

# 安裝酷音
> sudo apt-get install scim-chewing

# 安裝中文字體
> sudo apt-get install ttf-wqy-microhei ttf-wqy-zenhei xfonts-wqy

# 安裝 node-arms
> wget http://node-arm.herokuapp.com/node_latest_armhf.deb
> sudo dpkg -i node_latest_armhf.deb

# (安裝 node 錯誤)
dpkg: regarding node_latest_armhf.deb containing node:
 nodejs-legacy conflicts with node
  node (version 4.2.1-1) is to be installed.
  node provides node and is to be installed.

dpkg: error processing archive node_latest_armhf.deb (--install):
 conflicting packages - not installing node
Errors were encountered while processing:
 node_latest_armhf.deb

# (除錯)
> sudo apt-get remove nodered
> sudo apt-get remove nodejs nodejs-legacy
> sudo apt-get remove npm   # if you installed npm

# 安裝 PhantomJS
> sudo apt-get install g++ flex bison-doc bison gperf ruby ruby-dev perl libsqlite3-dev sqlite3 libfontconfig1-dev icu-doc libicu-dev libfreetype6 libssl-dev libpng12-dev libjpeg8-dev ttf-mscorefonts-installer fontconfig build-essential chrpath git-core openssl
> git clone git://github.com/ariya/phantomjs.git
> cd phantomjs
> git checkout 2.1.1
> ./build.py
> sudo chmod -x ~/phantomjs/bin/phantomjs
> sudo chmod 775 ~/phantomjs/bin/phantomjs
> sudo ln -s /home/pi/phantomjs/bin/phantomjs /usr/bin/

{% endhighlight %}

# dpkg 常用指令

{% highlight bash %}
> dpkg -l package_name # 列出該 package 相關資訊
> dpkg -l | less # 列出系統中所有安裝的軟體
> dpkg -L package_name # 列出該 package 所有檔案擺放位置
> dpkg -S file_name # 搜尋 file 所屬 package
> dpkg -i package_name # 安裝軟體
> dpkg -r package_name # 移除軟體
> dpkg -x package_name.deb target_dir # 解 .deb 檔案成數個檔案
> dpkg -i --force-overwrite-i package_name # 強制安裝軟體
> dpkg -i --force-all package_name # 強制安裝軟體
> dpkg -r --purge --force-deps package_name # 強制移除軟體
> dpkg --get-selections # 列出系統中所有安裝的軟體
> dpkg --pending --remove # 移除多餘的軟體
{% endhighlight %}


# 參考

* [裝置](https://www.raspberrypi.org/wp-content/uploads/2012/04/quick-start-guide-v2_1.pdf)
* [基本安裝含無線網路](http://tonyhack.familyds.net/wordpress/?p=3463)
* [Raspberry Pi 安裝中文輸入法與字型](http://blogger.gtwang.org/2014/12/raspberry-pi-chinese-input-method.html)
* [安裝 Node 除錯，含 v1, v2](https://www.raspberrypi.org/forums/viewtopic.php?f=66&t=130217)
* [安裝 Node 含開機自動執行設定](http://weworkweplay.com/play/raspberry-pi-nodejs/)
* [常用軟體](https://netduinoplusfun.wordpress.com/2012/07/03/some-useful-packages-for-the-raspberry-pi/)
* [編譯 PhantomJS](https://www.bitpi.co/2015/02/11/compiling-phantomjs-on-raspberry-pi/)
* [安裝 PhantomJS](https://www.bitpi.co/2015/02/10/installing-phantomjs-on-the-raspberry-pi/)

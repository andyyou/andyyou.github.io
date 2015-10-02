---
layout: post
title: 'Bower 簡介'
date: 2013-08-29 14:09:00
categories: Javascript F2E
---

Bower 是一套管理網頁套件的工具，他提供了一種通用且不受限制的方案來解決管理前端相關的套件如：jquery。
支援的系統非常廣泛，並沒有太多相依性的東西。開發者也可以透過他管理套件的相依性和升級等等的問題。
有了它，就不用到處去下載套件檔案(jquery, bootstrap)。

安裝
---
Bower 相依於 Node 和 npm 安裝指令如下

{% highlight bash %}
$ npm install -g bower
{% endhighlight %}

使用方法
---
大部份的資訊，都可以在 `bower help` 提供的說明中找到，一旦安裝完成就可以開始使用了。
Bower 是一組指令集，他們不需要使用 root 權限。如果你真的要限制權限
則使用 `--allow-root` 

安裝套件
---
透過 bower.json 來安裝管理，bower.json 就像一組清單用來幫你記錄管理想要安裝的套件和版本，先把清單些完之後執行下面的指令就能一口氣安裝完畢。

{% highlight bash %}
$ bower install
{% endhighlight %}
        
安裝在本地專案

{% highlight bash %}
$ bower install <package>
# ex: bower install jquery
{% endhighlight %}

指定版本

{% highlight bash %}
$ bower install <package>#<version>
{% endhighlight %}

<package> 可以是下面任何一種值
---
 1. 套件名稱 ex: `jquery`。可以透過 bower search jquery 來尋找相關的套件名稱。
 2. git 路徑 `git://github.com/someone/some-package.git`。
 3. 本地的 git 檔案庫。
 4. 縮寫的路徑 someone/some-package 預設是(github 上面的專案)。
 5. 網址指向一個 zip 或者 tar 檔案。
 
查詢已經安裝的套件
---

{% highlight bash %}
$ bower list
{% endhighlight %}

搜尋
---

{% highlight bash %}
$ boewr search [keyword]
{% endhighlight %}

安裝完後使用套件
---
最簡單的方式就是直接使用預設路徑

{% highlight html %}
<script src="/bower_components/jquery/index.js"></script>
{% endhighlight %}

對於更複雜的情況，你可能會需要使用腳本，或者使用一個載入模組。Bower 只是一個管理工具，此時可以使用其他的工具 - 如 RequireJS - 這將幫助你做到這一點。

移除
---
{% highlight bash %}
$ bower uninstall <package-name>
{% endhighlight %}
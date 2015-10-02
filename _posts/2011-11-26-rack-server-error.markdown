---
layout: post
title: '處理 Rack Server Error'
date: 2011-11-26 04:05:00
categories: Ruby, Rails
---

## 錯誤訊息：

~~~~~
initialize: Address already in use
~~~~~

## 解決方式：

查詢程序ID

{% highlight bash %}
$ lsof -iTCP:3000
$ lsof |grep 3000
{% endhighlight %}

刪除卡住程序

{% highlight bash %}
$ kill -9 00000  # 0000 = ID
{% endhighlight %}

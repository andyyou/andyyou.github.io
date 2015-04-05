---
layout: post
title: 'Linux/Windows RSA Key 常用指令'
date: 2011-11-16 05:30:00
categories: Linux
---

{% highlight bash %}
$ ssh-keygen -t rsa -C "youraccount@com.tw" #建立金鑰
$ ssh-add ~/.ssh/id_rsa #加入金鑰
$ ssh-add -l #列出金鑰清單
{% endhighlight %}

Error狀況: Could not open a connection to your authentication agent

{% highlight bash %}
$ ssh-agent bash
$ ssh-add
{% endhighlight %}

Permissions 0644 for '/c/Users/HelloWorld/.ssh/id_rsa' are too open. 

{% highlight bash %}
$ cd ~/.ssh
$ chmod 700 id_rsa
{% endhighlight %}

---
layout: post
title: 'Get gem path'
date: 2011-11-16 23:20:00
categories: Ruby
---

*取得Gem環境資訊*

{% highlight bash %}
$ gem env
{% endhighlight %}

*取得正在引入gem資訊*

{% highlight ruby %}
require 'rubygems'
require 'cucumber'
gem_root = Gem.loaded_specs['cucumber'].full_gem_path
gem_lib = File.join(gem_root, 'lib')
{% endhighlight %}


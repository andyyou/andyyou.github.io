---
layout: post
title:  "Jekyll - Psych::Nodes (NameError)"
date:   2015-04-04 14:49:00
categories: jekyll
---

## Jekyll 遇到 uninitialized constant Psych::Nodes (NameError)

當 jekyll 的環境遇到 `Ruby 2.1.x` `Psych 2.0.10` 會出現不相容問題，所以請

{% highlight bash %}
$ gem uninstall psych
{% endhighlight %}

[參考](https://github.com/dtao/safe_yaml/issues/72)
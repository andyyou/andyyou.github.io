---
layout: post
title: 'How to get plist?'
date: 2014-08-04 09:22:00
categories: ios
---

{% highlight objc %}
NSString *path = [[NSBundle mainBundle] pathForResource:@"YOUR_PLIST_FILE" ofType:@"plist"];
NSDictionary *dictionary = [[NSDictionary alloc] initWithContentsOfFile:path];
{% endhighlight %}
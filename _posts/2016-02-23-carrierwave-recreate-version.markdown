---
layout: post
title: '開發時期，CarrierWave 重產不同 version 圖片'
date: 2016-02-23 00:00:00
categories: rails
---

當增加不同 version 的尺寸需要對已上傳的圖片從新產生新尺寸的圖片時

{% highlight bash %}
> rails c
{% endhighlight %}

輸入

{% highlight bash %}
Attachment.all.each do |att|
  att.image.recreate_versions!
  att.save!
end
{% endhighlight %}

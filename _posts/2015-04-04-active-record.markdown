---
layout: post
title:  "ActiveRecord 雜記"
date: 2015-04-04 12:00:00
categories: Ruby Rails
---

## 優化 Model count

{% highlight ruby %}
class Like < ActiveRecord::Base
  belongs_to :guestbook_entry, counter_cache: true
end
{% endhighlight %}

## 其他小筆記
* Rails 慣例 date -> `created_on`, datetime -> `created_at`

* 如果 Model 的單偶數錯誤了就到 `config/initializers/inflections.rb` 去修改

{% highlight ruby %}
ActiveSupport::Inflector.inflections do |inflect| 
  inflect.irregular 'tax', 'taxes'
end
{% endhighlight %}

或者直接改 Model 本身

{% highlight ruby %}
class Sheep < ActiveRecord::Base 
  self.table_name = "sheep"
end
{% endhighlight %}

* 要換 key 的名稱，4 版後直接調整 schema

{% highlight ruby %}
create_table :products, id: false do |t|
  t.string :sku, primary_key: true

  t.timestamps
end
{% endhighlight %}

## rails console 下的一些指令

{% highlight ruby %}
$ Order.column_names
$ Order.columns_hash["column_name"]
{% endhighlight %}
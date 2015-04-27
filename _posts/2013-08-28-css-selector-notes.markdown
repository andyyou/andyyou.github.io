---
layout: post
title: 'CSS 選擇器筆記'
date: 2013-08-28 10:25:00
categories: CSS
---
#### *
通用選擇器 -> 套用所有元素

{% highlight css %}
* {color: yellow;}
{% endhighlight %}

#### <h1> 
類型選擇器 

{% highlight css %}
h1 {color: red;}
{% endhighlight %}

#### id

{% highlight css %}
#container {color: skyblue;}
{% endhighlight %}

#### class

{% highlight css %}
.span2 { color: hotpink;}
{% endhighlight %}

#### 屬性
過濾對應標籤元素中的屬性 `<a rel="friend" href="http://www.w3c.com/">w3c</a>`

{% highlight css %}
a[rel='friend'] {color: blue;}
{% endhighlight %}   

#### a[attr='value'] 
全部都要符合。

#### a[attr*='value']
只要有包含 value 都會被選中。

#### a[attr^='value'] 
只要開頭有都會被選中。

![Alt text](http://i.imgur.com/k1AvINu.png)

#### a[attr$='value'] 
只要結尾有 value 都會中。

![Alt text](http://i.imgur.com/eMLGRE9.png)

#### a[attr~='value'] 
以空格隔開的字串清單中，符合的就會中。

#### a[attr|='value']
以 `-` 隔開的字串清單中，如果開頭包含字串 value 。需要注意的是下圖 f 沒有被選中。

![Alt text](http://i.imgur.com/MmzbsSb.png)

#### div p
div 裡面的 p 都會被選中。

#### div > p
只有 div 內第一階的子元素會被選中。

#### div + ul
跟 div 同階層並緊跟著 div 後面的 ul 才會被影響。

#### div ~ p div 同階層後面所有的 p。

#### a, em 套用同一個設定。

---
layout: post
title: 'React 複製元件'
date: 2014-09-15 21:40:00
categories: Reactjs
---
## 複製元件
在少數的情況下，某個元件可能想要變更不屬於自己的 `props`(例如: 修改 `this.props.children` 的 `className` )。
或者是複製多個被傳入的元件。`cloneWithProps()` 是這件事變的可能。

{% highlight js %}
ReactComponent React.addons.cloneWithProps(ReactComponent component, object? extraProps)
{% endhighlight %}

複製淺層的 `component` 然後合併 `extraProps` 。 Props 被使用和 `transferPropsTo()` 一樣的方式合併，所以類似像 `className` 屬性就會被整合進去。

> 注意:
  `cloneWithProps` 不會轉移 `key` 屬性至複製的元件中。如果您希望保存 `key`，請使用 `extraProps` 物件

{% highlight js %}
var clonedComponent = cloneWithProps(originalComponent, { key : originalComponent.props.key });
{% endhighlight %}

---
layout: post
title: 'line-height 問題筆記'
date: 2016-03-08 19:00:00
categories: Program
tags: css
---

# line-height 歸納

* line-height 有五種屬性可用，預設是 normal 他會拿 font-size * 1.2
  + normal
  + number ex: 1.2, 1.8 => 倍數
  + unit ex: 18px, 2em, 3rem, etc...
  + percentage ex: 90%
  + inherit => line-height from parent

<!--more-->
# inline box

~~~
containing-box
  line-boxes (會看一行中的 line-height 誰最高來決定，所有內容共用此水平垂直線)
    inline-box (影響高的設定：line-height, 中文每一個單`字`是一個 inline-box，英文的話則是一個 word 為一個 inline-box), (設定為 inline 的 tag)
      content-area (影響高的設定：font-size, 隱形的區塊，純文字沒有圖片的情況下 box-model 的高決定權在 line-height)
        character
~~~

* 影響 box 高度的規則只有 height 和 line-height，一個 tag 如果沒設 height 的話最後決定高度的一定是 line-height
* 要說 font-size 也是因為預設 line-height = font-size * 1.2
* 使用 inherit 時，用百分比效能較差，用數值最佳
* font-size: 12px, line-height: 0 在只有文字的情況下，parent tag 不會被撐開，但是放入圖片就會被撐開。[codepen](http://codepen.io/andyyou/pen/eZZEJB)


# 實現多行垂直置中

外層包一個如下屬性的 container 即可

~~~
line-height: [height];
font-size: 0;
~~~

# 資源

* [MUKI - 深入 CSS 之 line-height 應用](http://muki.tw/tech/css-line-height/)
* [深入常見 vertical-align 問題](http://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/)
* [css行高line-height的一些深入理解及应用](http://www.zhangxinxu.com/wordpress/2009/11/css%E8%A1%8C%E9%AB%98line-height%E7%9A%84%E4%B8%80%E4%BA%9B%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E5%8F%8A%E5%BA%94%E7%94%A8/)

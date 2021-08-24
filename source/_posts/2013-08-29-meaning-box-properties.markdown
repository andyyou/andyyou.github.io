---
layout: post
title: 'Box 屬性意義'
date: 2013-08-29 10:29:00
categories: Program
tags: [css]
---

* `display: block` : 前後會插入換行，寬會填滿父元素，高則是配合內容。可以用 width, height 設定。
  預設: h1, p, div

* `display: inline` : 不會換行，會連成一行。寬高都是依照內容，且不能使用 width, height。
  預設: span, a, em

* `display: inline-block` : 跟 inline 一樣，但能夠使用 width, height 設定寬高。

* `display: list-item` : 變成 marker + block。

* `display: run-in` : 如下圖 h2 被指定 `run-in` 它就會變成 inline 後續的元素如果是 block 就會如圖一樣合體。但是後續的元素如果是 inline 或者套用了 float, position ，那麼 h2 就會是 block。 `註: Firefox 和 IE7 目前都不支援 run-in`

<!--more-->

![](http://i.imgur.com/5PjwcM2.png)

![](http://i.imgur.com/ewaSw1j.png)

* `display: none` : 隱藏

* `display: table` : 表格

				<style type="text/css">
				#container {
				overflow: hidden;
				display:table;
				width: 100%;
				}
				.row {display:table-row;}
				.row div {display: table-cell;}
				.a {background:#00ff00;}
				.b {background:#ff00ff;}
				.c {background:#0000ff;}
				</style>
				<div id="container">
					<div class="row">
 						<div class="a">
							test<br />test<br />test<br />test<br />test<br />test<br />test
						</div>
						<div class="b">
							test
						</div>
						<div class="c">
							test
						</div>
					</div>
				</div>

* `display: compact` : 可以插入在後續 block 的 margin 區域。

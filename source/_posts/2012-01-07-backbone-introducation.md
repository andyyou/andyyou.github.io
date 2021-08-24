---
title: Backbone 簡介
date: 2012-01-07 08:00:00
categories: Program
tags: [javascript]
---


![Alt text](http://backbonejs.org/docs/images/backbone.png)

前言
---

這篇文章不是要深入探討關於BackboneJS的使用，純粹是因為最近Javascript MVC Framework
一直推陳出新。但像小弟資質愚鈍其實剛知道這東西的時候完全不知道她在干嘛。
這篇文章只是大略用範例來做些基本的說明。

<!--more-->

What is BackboneJS ?
--------------------

[Backbone](http://backbonejs.org/) 是一個為前端設計的JavaScript框架。 不同於jQuery專注於簡化DOM的操作和事件繫結。
Backbone 提供結構化來達到分離資料模型和DOM。就像是MVC框架分離View，Model，Controller。
它讓複雜的JavaScript應用程式更簡單的開發和維護。

為什麼使用Backbone?
-----------------

在jQuery裡，我們可能會使用一連串的事件來指派資料到DOM像以下範例:

```js
//宣告一個資料物件
var article = {
  author:"Joe",
  content: "testing"
};
//透過jQuery Selector選到ID為article的DOM然後繫結事件放入資料等操作
$('#article').click(function(event){
  $(this).find('.content').text(article.content);
});
```

接著，看看Backbone. 它提供Model Class和View Class這兩個Backbone最主要的元素。

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>Hello Backbone</title>
	  <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
    <script src="http://ajax.cdnjs.com/ajax/libs/json2/20110223/json2.js"></script>
    <script src="http://ajax.cdnjs.com/ajax/libs/underscore.js/1.1.6/underscore-min.js"></script>
    <script src="http://ajax.cdnjs.com/ajax/libs/backbone.js/0.3.3/backbone-min.js"></script>
	<style type="text/css" media="screen">
		.content{width:300px;display:block;border:1px solid #ccc;}
		#article{width:300px;display:block;}
	</style>
</head>
<body>
	<script>

	$(function(){
		//宣告一個Article的Model，概念上就好像設定 Article 是一個Class的感覺。
		var Article = Backbone.Model;

 		//new 一個 article 模型實體包含資料(內容當然可以取自JSON或其他來源)
 		//其實上述的兩個步驟通常也可以用 Backbone.Model.extend 的寫法
		var article = new Article({
  			author:"Joe",
  			content: "testing"
		});

 		/*
 		核心觀念
 			定義一個 ArticleView 概念上還是有點像是在設計ArticleView 這個Class
 		*/
		var ArticleView = Backbone.View.extend({
			  //el = element 設定對應的DOM
  			el: $('#article'),
  			//初始化 initialize: 和 el: , events: 可以想成是new一個物件時的建構子等事件。
  			initialize: function(){
  				//將自訂動作的function和這個DOM綁定,意思就是當其他事件呼叫updateContent 它會自動去影響this
  				//這邊有一點要先搞清楚這個功能是Underscore JS 提供的。
  				//這邊的功能是當綁定的事件被觸發會影響被綁定的物件。
    			_.bindAll(this,'updateContent');
    			//當Model改變時就呼叫updateContent. 可以試著將上面註解掉觀察其不同
    			this.model.bind('change',this.updateContent);
  			},
  			//事件綁定
  			events: {
    			"click .content" : "updateContent"
  			},
  			//自訂函式 可以理解成操作的動作
  			updateContent: function(){
    			this.$('.content').text(this.model.get("content"));
  			}
		});
 		//上面設計好的Class 就new 一個出來 帶入model資料。
		var articleView = new ArticleView({model:article});
		//這邊是測試直接改變model 顯示會不會改變 這邊可以把_.bindAll 註解觀察不同點.
		$('#fire').click(function(){
			article.set({content: 'Change Model'});
		});
	});

	</script>
	<div id="article">Click Me <div class="content">Initialize</div> </div>
	<button id="fire">Change</button>

</body>
</html>
```

結論
----

到這邊為止,大致上對於Backbone做的事情有些概略的認識.簡單來說Backbone就是提供一個框架讓我們把既存的JavaScript Code 結構上可以寫成MVC的架構。
對於功能單純的程式頁面來說這樣做好相反而會增加大量程式碼，但當頁面功能越來越複雜的時候。
好處是其他接手維護的人可以用既定的規則去理解部分的功能。像最上面jQuery的寫法。可能免不了要重頭
看到尾。

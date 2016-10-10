---
layout: post
title: 'KineticJS 建立範圍選取功能'
date: 2013-11-28 14:17:00
categories: Program
tags: javascript
---

# KineticJS 介紹
KineticJS 是一套針對 canvas 設計的函式庫，使得我們在操作使用 canvas 的時候相對簡單易懂。
這篇文章將會教您如何透過 KineticJS 在螢幕上（canvas）建立一些物件，然後可以透過拖拉選取範圍。

<!--more-->

# 背景
如果您還不了解 HTML5 canvas 標簽，請先閱讀[HTML5 Canvas Tutorials](http://www.html5canvastutorials.com/)。如果您還不知道什麼是 KineticJS 請先至[官網](http://kineticjs.com/) 閱讀基本教學。
這篇文章是針對 KineticJS v4.7.4。

# 程式碼說明
在這個範例中，將會在 canvas 建立三個方塊，然後可以選取它們。首先呢，您必須在 html 中有一個 container

~~~html
<div id='container'></div>
~~~

接著建立 stage 和 layer ，這部分如果您不能理解請先參考基本教學。簡單的說明，透過 Kinetic 來操作 canvas
我們一般會建立一個 stage 和 layer ，透過 layer 包含各種圖形物件，stage 包含 layer 分層的方式來組織 canvas

~~~js
var stage = new Kinetic.Stage({
  container: 'container',
  width: 100,
  height: 100
});
var layer = new Kinetic.Layer();
~~~

為了完成我們的功能，第一個技巧是在整個 layer 放入一張全滿透明的方形物件，這物件是用來判斷當使用者
點擊滑鼠的時候可以開始進行拖曳以及放開的時候做些對應的處理。

~~~js
var rectBackground = new Kinetic.Rect({
  x: 0,
  y: 0,
  height: stage.attrs.height,
  width: stage.attrs.width,
  fill: 'transparent',
  draggable: false,
  name: 'rectBackground'
});
~~~

現在我們可以加入上面說的方形物件。

~~~js
DrawBlocks();
function DrawBlocks() {
  var x, y, heigth;
  x = 90;
  y = 10;
  size = 40;
  CreateBlock(x, y, size, size, 'green');

  x = 150;
  y = 80;
  CreateBlock(x, y, size + 20, size + 60, 'red');

  x = 110;
  y = 170;
  CreateBlock(x, y, size, size, 'blue');
  layer.draw();
}

function CreateBlock(x, y, height, width, color) {
  var grpBlk = new Kinetic.Group({
  	x: x,
    y: y,
    height: height,
    width: width,
    name: color,
    draggable: true
  });
  var blk = new Kinetic.Rect({
    x: x,
    y: y,
    height: height,
    width: width,
    fill: color,
    name: color + ' block'  
  });
  grpBlk.add(blk);
  blk.setAbsolutePosition(x, y);
  grpBlk.setAbsolutePosition(x, y);
  layer.add(grpBlk);
  return grpBlk;
}
~~~

這個範例讓我們可以透過拖拉滑鼠建立一個選取方塊，為了做到這點我們需要一些變數的協助。

~~~js
var arSelected = new Array(); // 陣列是用來保存被選取到的 block 的名稱。
var bDragging = false;	      // 當程式正在透過滑鼠拖拉計算方塊的大小時避免程式又從新執行。
var bHaveSelBox = false;      
var rectSel = null;           // 最終看到的選取方形
var initX = 0;                // 初始的 x, y 坐標。
var initY = 0;
~~~

現在我們需要介紹一些處理事件，第一件要做的事情是我們需要截取當滑鼠點擊的那一瞬間。

~~~js
rectBackground.on('mousedown', function(evt) {
  bDragging = true;
}
~~~

接著，當滑鼠開始移動拖拉，我們需要重新產生選取方塊。

~~~js
stage.getContent().addEventListener('mousemove', function(e) {
	if (bDragging)
  {
  	SetSelRectPosition(e);
  }
});

var bInHere = false; // 防止事件重新被執行。
function SetSelRectPosition(e) {
  if (bDragging && !bInHere)
  {
  	bInHere = true;
    var canvas = layer.getCanvas();
    var mousepos = stage.getPointerPosition();
    var x = mousepos.x;
    var y = mousepos.y;

    if (!bHaveSelBox) {
    	initX = x;
      initY = y;
      rectSel = new Kinetic.Rect({
      	x: initX,
        y: initY,
        height: 1,
        width: 1,
        fill: 'transparent',
        stroke: 'black',
        strokeWidth: 1
      });
      layer.add(rectSel);
      layer.draw();
      bHaveSelBox = true;
    } else {
    	var height = 0;
      var width = 0;
      var newX = 0;
      var newY = 0;

      if (x > initX)
        newX = initX;
      else
        newX = x;
      if (y > initY)
        newY = initY;
      else
        newY = y;

      height = Math.abs(Math.abs(y) - Math.abs(initY));
      width = Math.abs(Math.abs(x) - Math.abs(initX));

      rectSel.setHeight(height);
      rectSel.setWidth(width);
      rectSel.setX(newX);
      rectSel.setY(newY);
      layer.draw()
    }
  }
  bInHere = false;
}
~~~

接著當使用者放掉滑鼠按鍵時，我們需要計算哪些物件被選取了然後把它們的名稱放到 arSelected 陣列。
稍後我們就可以拿這個清單去做高亮等處理。

~~~js
stage.getContent().addEventListener('mouseup', function(e) {
  if (bDragging) {
    bDragging = false;
    GetOverlapped();
    if (rectSel != null)
      rectSel.remove();

    rectSel = null;
    bHaveSelBox = false;
    layer.draw();
  }
});

function GetOverlapped()
{
  if (rectSel == null) {
    return ;
  }
  var iHeight = 0;
  var iWidth = -1000;
  arSelected.length = 0;
  initX = 10;
  initY = 10;
  var arGroups = layer.getChildren();
  for (var i=0; i<arGroups.length; i++) {
    var grp = arGroups[i];
    if (grp.attrs.name != rectSel.attrs.name && grp.attrs.name != rectBackground.attrs.name grp.attrs.name != 'btn' && grp.attrs.name != 'highlightBlock') {
      var pos = rectSel.getAbsolutePosition();
      var selRecXStart = parseInt(pos.x);
      var selRecXEnd = parseInt(pos.x) + parseInt(rectSel.attrs.width);
      var selRecYStart = parseInt(pos.y);
			var selRecYEnd = parseInt(pos.y) + parseInt(rectSel.attrs.height);
      var grpXStart = parseInt(grp.attrs.x);
			var grpXEnd = parseInt(grp.attrs.x) + parseInt(grp.attrs.width);
			var grpYStart = parseInt(grp.attrs.y);
			var grpYEnd = parseInt(grp.attrs.y) + parseInt(grp.attrs.height);

      if ((selRecXStart <= grpXStart && selRecXEnd >= grpXEnd) && (selRecYStart <= grpYStart && selRecYEnd >= grpYEnd))
			{
				if (arSelected.indexOf(grp.getName()) < 0)
				{
					arSelected.push(grp.getName());

					var tmpX = parseInt(grp.attrs.x);
					var tmpY = parseInt(grp.attrs.y);

					var rectHighlight = new Kinetic.Rect({
						x: tmpX,
						y: tmpY,
						height: grp.attrs.height,
						width: grp.attrs.width,
						fill: 'transparent',
						name: 'highlightBlock',
						stroke: '#41d6f3',
						strokeWidth: 3
					});

					layer.add(rectHighlight);
				}
			}
    }
  }
}
~~~

最後當使用者在背景點一下要取消選取，或者選取單一方塊

~~~js
stage.getContent().addEventListener('mousedown', function(e) {
  if(arSelected.length > 0)
  {
    var name = '';
    if (e.shape != undefined)
      name = e.shape.attrs.name;
    if (e.targetNode != undefined)
      name = e.targetNode.attrs.name;
    if (name != 'btn')
      RemoveHighlights();
  }
});

function RemoveHighlights()
{
	var arHighlights = layer.get('.highlightBlock');
	while (arHighlights.length > 0)
	{
		arHighlights[0].remove();
		arHighlights = layer.get('.highlightBlock');
	}
	arSelected.length = 0;
}
~~~

在這個範例中我們會在增加一個按鈕可以用來取得關於被選取物件的資料

~~~js
x = 85;
y = 250;
var grpGetSelectedButton = CreateButton(x, y, "Get Selected");
grpGetSelectedButton.on("click", function (evt) { ShowSelected(); });

function CreateButton(x, y, text)
{
	var grpButton = new Kinetic.Group({
		x: x,
		y: y,
		height: 30,
		width: 135,
		name: 'btn',
		draggable: true
	});

	var blkButton = new Kinetic.Rect({
		x: x,
		y: y,
		height: 30,
		width: 135,
		fill: 'Violet',
		name: 'btn'
	});

	var txtButton = new Kinetic.Text({
		x: x + 2,
		y: y + 2,
		fontFamily: 'Calibri',
		fontSize: 22,
		text: text,
		fill: 'black',
		name: 'btn'
	});

	grpButton.add(blkButton);
	grpButton.add(txtButton);
	grpButton.setAbsolutePosition(x, y);
	blkButton.setAbsolutePosition(x, y);
	txtButton.setAbsolutePosition(x + 2, y + 2);

	layer.add(grpButton);

	return grpButton;
}

function ShowSelected()
{
	var str = "";
	for (var i = 0; i < arSelected.length; i++)
	{
		str += arSelected[i] + ", ";
	}
	if (str != "")
		str = str.substring(0, str.length - 2);

	alert(str);
}
~~~

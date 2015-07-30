---
layout: post
title: "理解 Bootstrap 3 Grid 的運作機制"
date: 2015-07-24 08:00:00
categories: Css
---

像 CSS 網格機制這樣的東西，通常被隱藏在背後。沒有人真的需要完全懂它是這麼運作的，就如同你會開車但不見得會作出一台汽車。
直到我們弄壞了設計或者遇到某些非常詭異複雜的狀況，例如找不出為什麼那些該留白的地方, margin, padding 等等都不照著你的想法運作。
這種狀況真的很令人挫折，尤其當你在介面動態產生一些內容的時候。

包含我自己在內我曾看過一些朋友非常困惑關於 bootstrap grid 的運作，這篇文章就快速且透過視覺話解釋 bootstrap grid 的運作。

# Container
關於 `container` 有兩個目的:
1. 在 responsive width 即寬是動態的機制下提供一個 width 的限制，當 responsive size 改變，container 就會改變。 rows 和 columns 都是靠百分比來設定的並且依據 container 來調整其尺寸，也因此要變動的就只有 container 。
2. 設定 padding ，所以內容不會直接壓在瀏覽器的邊緣上，在 bootstrap 中 container 在兩邊各提供了 15px 的 padding

![](http://www.helloerik.com/wp-content/uploads/image-1.png)

# Row

row 則是放 column 的位置，理想的情況下 column 加總會等於 12 。由於所有的 column 都是 `float: left` 所以 row 的角色就是一個列的容器，此外 row 也處理掉一些因為 float 而產生的怪異行為，例如和下一排疊在一起等等。

row 有一個比較特別的地方就是其左右具有 `margin` `-15px` ，如同下圖藍色的部份。 加上 row 樣式的 div 元素通常會被限制在 container 裡面包含上圖粉紅色的區塊。
本質上 row 的 margin -15px 抵消了 container 的 padding 15px。至於為什麼要這麼做，因為這牽扯到 column 的運作，等等我們將會解釋為什麼。

記住，千萬別要在 container 外面直接使用 row。

![](http://www.helloerik.com/wp-content/uploads/image-2.png)

# Column
column 本身具有 15px 的 padding 如下圖黃色的部分。 這意味著 column 的邊緣的確是和 row 的邊緣切齊，同時因為 container 和 row 的 15px 抵消所以 column 也和 container 切齊，但是 column 的 15px 會讓其內部的內容留下 15px 的間距，造成 column 和 column 之間也就是內容之間的間距是 30px。

![](http://www.helloerik.com/wp-content/uploads/image-3.png)

# Column 內的內容
column 本身的 padding 讓我們的內容產生間距，最終這三者搭配出如下圖的結構

![](http://www.helloerik.com/wp-content/uploads/image-4.png)

# Nesting 巢狀結構
當你擁有一個 container 搭配 row 和 column 組成網格系統，接著你可以在 column 內部再依照同樣的結構排版，現在 row 會幫你抵消 column 的 15px<
所以你不需要在 column 內部再加入 container。

![](http://www.helloerik.com/wp-content/uploads/image-5.png)

注意千萬不要再 column 在使用 container 

所以抵消 15px 的原因是因為要讓 column 和 container 的行為一致。最終保持內容之間的間距就是 30px 

![](http://www.helloerik.com/wp-content/uploads/image-6.png)

# Offset
offset 看起來很神奇但實際上非常單純，他只不過是加上 `margin-left`

![](http://www.helloerik.com/wp-content/uploads/image-7.png)

# Push 和 Pull
讓我們開始講解 push/pull 當我們因應 Responsive 尺寸需要反轉佈局，尤其是為了重新排列在不同裝置的佈局，有了 push/pull 您就可以打破 HTML 元素排列必須要從上至下，從左至右的規則

通常這邊是最容易搞混的地方，一個 push 實際上是加上 left 屬性而 pull 則是 right，因為這些 column 都是 float 元素並且被包含在在 relative 的容器裡，column 本身也是 `position: relative` 最後就可以透過 left/right 來改變位置，例如 col-sm-push-4

![](http://www.helloerik.com/wp-content/uploads/image-8.png)

使用 push/pull 會有一個要注意的問題就是他會造成 column 被重疊，且不會像一般 col-* 搭配 offset 一樣
所以如果您 push 了一個 column 你就要自己控制該 row 的 column ，一個建議是要改變方向的時候就一整個 row 都換同一個方向，因為變成設定 position 所以每個 column 的位置都要自己控制。

![](http://www.helloerik.com/wp-content/uploads/image-9.png)

# 一些機制背後的動機

* Container：container 有一個虛設的 15px padding ，這是從 RC1 版本之後才修改的，在這之前整個 body 都有 15px 的 padding 造成沒有套用 bootstrap 規則的元素不會貼齊邊緣，這導致全寬設計，通常是背景色設定會變得不好做還要加上額外的樣式。

* Row：設有 -15px margin 抵消 container 的 padding 讓 row 可以貼齊 container 真正的邊緣

* Column：有 15px padding 讓所有的內容跟邊緣保持 15px 的間距，而 column 和 column 之間則是 30px，如此一來就不需要像 960, blueprint 等網頁柵格系統一樣對第一個和最後一個元素作特殊處理

* Nested Column：行為跟 column 一致同時也讓內部使用 row 的行為和 container 一樣

* Offset：如果欄位要推移也不需要再加入空白的 column 直接利用計算好的 margin-left 來偏移

* Push/Pull：用來改變排版順序，通常只有 Offset 和基本用法不能處理的時候才會用到這個方式。

# 常見的問題

## 缺少 container
關於 container 的第一個問題，就是它本身不具備 container 的意義，對於 row 來說並沒有 padding，row 因為 -15px 的關係會超出父元素。
在一個 `overflow: hidden` 的 container 或靠近瀏覽器邊緣的情況下，通常是造成 column 不對齊或奇怪的水平捲軸出現的兇手。

![](http://www.helloerik.com/wp-content/uploads/image-10.png)

## 缺少 row 
另外一個類似的問題也發生在如果沒有適當的使用 row 樣式，則外產生多餘的 padding，因為破壞的設計所以 float 的 column 就會不照原來的規則對齊邊緣。
此外如果您試圖在這種狀況下使用巢狀結構，會得到多一倍的 padding<

![](http://www.helloerik.com/wp-content/uploads/image-11.png)

## container 內部放入其他東西
在 container 第一層的其他子元素行為會像 column，此時不會抵消 container 的 15px padding ，但你不能直接在 container 內馬上接 container 。
常見的錯誤用法就是在 container 或 column 內部又使用 container 

![](http://www.helloerik.com/wp-content/uploads/image-12.png)

## Offset/Push/Pull
offset 本身用起來沒什麼太大的問題，就只是增加 margin-left ，不過 push/pull 則需要注意，因為他是直接改位置，如果設太大 column 會破版跑出 container 以外。

![](http://www.helloerik.com/wp-content/uploads/image-13.png)

上面大概就是常見的 Grid 問題。基本上在 CSS 世界裡沒什麼是絕對的 bootstrap 只是開發者在經驗累積下的產物，設計了一系列的慣例，正確的跟著文件做就不會產生錯誤。


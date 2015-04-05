---
layout: post
title: 'CSS 選擇器筆記'
date: 2013-08-28 10:25:00
categories: CSS
---
1. `*` 通用選擇器 -> 套用所有元素
				* {color: yellow;}

2. `<h1>` 類型選擇器 
				h1 {color: red;}

3. `id`
				#container {color: skyblue;}

4. `class`
				.span2 { color: hotpink;}

5. `屬性` -> 過濾對應標籤元素中的屬性 `<a rel="friend" href="http://www.w3c.com/">w3c</a>`
				a[rel='friend'] {color: blue;}
        
6. `a[attr='value']` 全部都要符合。
7. `a[attr*='value']` 只要有包含 value 都會被選中。
8. `a[attr^='value']` 只要開頭有都會被選中。
![Alt text](http://i.imgur.com/k1AvINu.png)

9. `a[attr$='value']` 只要結尾有 value 都會中。
![Alt text](http://i.imgur.com/eMLGRE9.png)

10. `a[attr~='value']` 以空格隔開的字串清單中，符合的就會中。

11. `a[attr|='value']` 以 - 隔開的字串清單中，如果開頭包含字串 value 。需要注意的是下圖 f 沒有被選中。
![Alt text](http://i.imgur.com/MmzbsSb.png)

12. `div p`  div 裡面的 p 都會被選中。

13. `div > p` 只有 div 內第一階的子元素會被選中。

14. `div + ul`  跟 div 同階層並緊跟著 div 後面的 ul 才會被影響。

15. `div ~ p` div 同階層後面所有的 p。

16. `a, em` 套用同一個設定。




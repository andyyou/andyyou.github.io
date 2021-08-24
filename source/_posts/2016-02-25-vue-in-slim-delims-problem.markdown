---
layout: post
title: 'Vue.js in Slim 語法的小問題 Slim::Parser::SyntaxError'
date: 2016-02-25 12:00:00
categories: Program
tags: [vuejs, slim, javascript, RoR]
---

當我們使用 `Vue.js` 搭配 `slim` 時(事實上 `Angular` 應該也有相同的問題)時

~~~slim
div id="app"
  p {{message}}

javascript:
  new Vue({
    el: '#app',
    data: {
      message: "Hello, Vue.js"
    }
    })
~~~

立馬收到`Slim::Parser::SyntaxError`的錯誤訊息。

但是改成這樣卻又正常

~~~slim
div id="app" {{ message }}
~~~

好啦！答案很明顯了就是我們有地方寫錯，讓 slim engine 誤會了。

這邊紀錄一下解法：

### 補上屬性

第一個最簡單的方式就是幫 `p` 補上隨意一個屬性

~~~slim
div id="app"
  p class="" {{ message }}

javascript:
  new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue.js',
    }
  })
~~~

### 加上 [], (), {} 任何一種

~~~slim
div id="app"
  p () {{ message }}

javascript:
  new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue.js',
    }
  })
~~~

### 使用 |

~~~slim
div id="app"
  p
    | {{ message }}

javascript:
  new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue.js',
    }
  })
~~~

### 修改設定

上面的解法都是因為 slim 預設會把 `{}` `()` `[]` 和 tag 後面接的 `property=value` 當作屬性(attributes)來解析。
所以我們只要把 `{}` 拿掉就正常了。

新增或修改 `config/initializers/slim.rb` 加入

~~~ruby
Slim::Engine.set_options :attr_list_delims => {'(' => ')', '[' => ']'}
~~~

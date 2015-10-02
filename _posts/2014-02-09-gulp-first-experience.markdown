---
layout: post
title: 'Gulp 初體驗'
date: 2014-02-09 20:16:00
categories: Javascript F2E
---
# Gulp 初體驗
從轉用 Grunt 以來其實我沒有遇到太多問題，加上大部份的 Framework 都把 Task 寫得好好的，對我來說用就好了。特別要客製的地方大概也都是小改一下別人的 Gruntfile 。
這篇記錄沒有要特別去分析 Gulp 。介紹在這篇[The streaming build system Gulp](http://blog.wu-boy.com/2013/12/streaming-build-system-gulp/)就解釋得蠻清楚的了。
只不過在這不想外出的下雨天稍微用看看 Gulp。以下記錄非常單純，只是透過 Gulp 來編譯 Coffee, Jade，使用一下 watch 功能體驗一下。
透過實作我覺得比較容易理解：

## 安裝

{% highlight bash %}
$ npm install -g gulp
{% endhighlight %}

## 建立一個簡單的測試專案

{% highlight bash %}
$ mkdir gulp-test # 建立目錄
$ npm init # 建立 package.json
$ npm install gulp gulp-util gulp-jade gulp-coffee gulp-watch
{% endhighlight %}


## 建立 gulpfile.js
這隻檔案的功能跟 Grunt 的 `Gruntfile.js` 功能上是一樣的就是組織任務的地方。
下面這段程式碼雖然已經被驗證有瑕疵，但在這邊只是為了體驗一下概念還是先保留後面會再補上比較好的做法。

{% highlight js %}
var gulp = require('gulp');
var gutil = require('gulp-util');
var jade = require('gulp-jade');
var watch = require('gulp-watch');
var coffee = require('gulp-coffee');

gulp.task('default', function () {
  gulp.src('./*.coffee')
    .pipe(watch())
    .pipe(coffee())
    .pipe(gulp.dest('./'))
  ,
  gulp.src('./*.jade')
    .pipe(watch())
    .pipe(jade())
    .pipe(gulp.dest('./'))
});
{% endhighlight %}

## 建立測試用 Jade

{% highlight html %}
doctype html
html
  head
    title Hello Gulp & Jade
    script(src='test.js')
  body
    h1 Cool, Getting Started
{% endhighlight %}

## 列出可執行的任務

{% highlight bash %}
$ gulp -T
{% endhighlight %}

![](http://i.imgur.com/7BzD0Wj.png)

## 執行任務

{% highlight bash %}
$ gulp
{% endhighlight %}

# 比較好的做法
根據官方的範例，其實 watch 應該是直接使用內建的 api 就好了。下面把試用的一段程式碼提供給大家參考

{% highlight js %}
// 載入函式庫
var gulp = require('gulp');
var gutil = require('gulp-util');
var jade = require('gulp-jade');
var watch = require('gulp-watch');
var coffee = require('gulp-coffee');
// 定義路徑
var paths = {
  coffee: ['*.coffee'],
  jade: ['*.jade']
};
// 編譯 coffee script 任務
gulp.task('coffee', function () {
  gulp.src(paths.coffee)
    .pipe(coffee())
    .pipe(gulp.dest('./'))
});
// 編譯 jade 任務
gulp.task('jade', function () {
  gulp.src(paths.jade)
  .pipe(jade())
  .pipe(gulp.dest('./'))
 gutil.log('log here...ok') // 可以使用 gutil 輸出資料
});

var watcher = gulp.task('watch', function () {
  // 建立完成任務後的 callback
  var done = function (evt) {
    console.log('File ' + evt.path + ' was ' + evt.type + ', running tasks...');
  };

  var coffeer = gulp.watch(paths.coffee, ['coffee'], done);
  // 觸發時綁定一個 event
  coffeer.on('change', function () {
    gutil.log('start changing...');
  });

  gulp.watch(paths.jade, ['jade']);
});

gulp.task('default', ['coffee', 'jade', 'watch']);
{% endhighlight %}

## 心得
用起來其實蠻直覺的，不過小試了一下覺得第一文件有點不完整。有些小細節沒有內建功能，例如 gulp -T 對於指令竟然沒有地方可以放描述(我在文件上沒找到)。寫起來很爽，不過我覺得對專案來說除了寫 `gulpfile` 以外的其他開發者應該會覺得這些 task 提示的訊息怎麼這麼少。當然這些是可以加的，只不過初步用起來會覺得少蠻多東西的。XD


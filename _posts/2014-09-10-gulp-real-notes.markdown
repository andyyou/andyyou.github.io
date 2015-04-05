---
layout: post
title: 'Gulp 使用問題與記錄'
date: 2014-09-10 20:14:00
categories: Javascript F2E
---

這篇文章除了記錄一下最後只是堪用的解法，如果有高手知道更好的做法煩請賜教。

話說小弟今天要來寫一些 React 的測試專案，就想說那就搭上 Gulp 吧！原本的目的很單純就只是用 Gulp compile 一些常用的 meta language，像是 Less, Jade 外加上 Jsx。也不是什麼很大的專案所以架構上其實就是在 React 的 StartKit上面再開個自己的測試目錄。

一開始也都用的很開心，然後就發現：在啟動 server 支援 livereload 後，當從 `src` 目錄刪除檔案，`dist` 目錄的檔案竟然沒被刪掉！！！(好啦！我知道正確來說應該是要先輸出到 .tmp 在 copy)。
說明一下我想完成的目標就是：

* 一個簡單的 Server 支援 watch & livereload。
* 存檔後直接把 src 底下的檔案編譯至對應的 dist 目錄下。
* 接著我希望 dist 目錄下的檔案要正確的對應，意思是如果我新增/刪除一個檔案，那 dist 也要新增/刪除。

直覺反應這也不是什麼大問題，那就補上 clean 的套件就好，一開始沒注意到 gulp-clean 有非同步的小問題因為我是要刪 dist 而不是 src 所以直覺得拆成兩個 task，想說我任務都有照順序先 clean 在 compile 為什麼一下噴 Error，一下又正常，再加上一開始不想要用全部清掉這種方式。

於是就讓我 Google 到這一篇 [Delete feature request](https://github.com/floatdrop/gulp-watch/issues/4h) ，因為下面有人提到使用 `gulp-filter` 的方式，接著我就將 `watch` 的 task 部分換成

{% highlight js %}
gulp.task('default', function () {
    watch('css/**/*.css').pipe(gulp.dest('./dist/'));
});
{% endhighlight %}

這種寫法，並補上 filter ，本來以為要打完收工的時候，卻發現我對 Node 很多東西觀念太薄弱，我不會替 `vinyl-fs` 物件綁上 event，也不知道怎麼根據 pipe() 來的檔案資訊來切換目錄，且有人提到可以用 `gaze` 的方式我試了半天也宣告失敗。附帶一提當你使用上面這種 watch 的寫法時其實 log 的資訊比較清楚。

最後差強人意的 `gulpfile` 在下面，最後如果有興趣要測的可以用 [Github](https://github.com/AndyYou/gulp-example) 測試環境在 `playground` 目錄下

{% highlight js %}
var gulp = require('gulp'),
    connect = require('gulp-connect'),
    less = require('gulp-less'),
    react = require('gulp-react'),
    watch = require('gulp-watch'),
    jade = require('gulp-jade'),
    clean = require('gulp-clean');


/**
* Compilers
*/
gulp.task('less', ['clean-css'], function () {
  gulp.src('playground/src/styles/less/*.less')
      .pipe(less())
      .pipe(gulp.dest('playground/dist/css'));
});


gulp.task('clean-css', function () {
  gulp.src('playground/dist/css/*', {read: false}).pipe(clean({force: true}));
});


gulp.task('jsx', ['clean-js'], function () {
  gulp.src('playground/src/scripts/jsx/*.jsx')
      .pipe(react())
      .pipe(gulp.dest('playground/dist/js/'));
});


gulp.task('clean-js', function () {
  gulp.src('playground/dist/js/*', {read: false}).pipe(clean({force: true}));
});


gulp.task('jade', ['clean-html'], function () {
  gulp.src('playground/src/templates/**.jade')
      .pipe(jade())
      .pipe(gulp.dest('playground'));
});


gulp.task('clean-html', function () {
  gulp.src('playground/*.html', {read: false}).pipe(clean({force: true}));
});


/*********************************************************/


/**
* Web Server
*/
gulp.task('server', function () {
  connect.server({
    root: ['playground'],
    livereload: true
  });
});


gulp.task('livereload', function () {
  watch(['playground/*.html', 'playground/dist'])
      .pipe(connect.reload());
});


gulp.task('watch', function () {
  gulp.watch('playground/src/styles/less/*.less', ['less']);
  gulp.watch('playground/src/scripts/jsx/*.jsx', ['jsx']);
  gulp.watch('playground/src/templates/**.jade', ['jade']);
});


/*********************************************************/

/**
* Mixin feature of usage
*/
gulp.task('default', ['less', 'jsx', 'jade', 'server', 'livereload', 'watch']);
{% endhighlight %}


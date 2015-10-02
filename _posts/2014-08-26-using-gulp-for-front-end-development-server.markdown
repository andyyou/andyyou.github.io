---
layout: post
title: '使用 Gulp 為前端開發伺服器'
date: 2014-08-26 19:59:00
categories: Javascript F2E
---

因為 `Gulp` 這個任務執行工具最近越來越流行，所以想說試著用它來做些事情，其實它跟 Grunt 一樣可以拿來處理很多事情像是整合 Javascript 檔案，壓縮圖片等等。如果你之前從來沒聽過 Gulp 這裡建議您先閱讀 [英文教學](http://code.tutsplus.com/tutorials/managing-your-build-tasks-with-gulpjs--net-36910), [Gulp 初體驗](http://andyyou.logdown.com/posts/178360-gulp-first-experience)。
在這篇文章中，我們將學習使用 Gulp 來完成一個簡單的小型伺服器以協助您平日的前端開發，當然這也支援 `livereload`。

## 從前的方式
假想一下當你想開發一個 SPA (單頁式應用程式)，這個應用程式的進入點是 `index.html` ，最後我們的目標是透過 `localhost` 來存取這個頁面。為什麼一定要用 `localhost` 因為一些安全機制你不能直接透過 `file:///` 的檔案來執行 `ajax`，在過去或許你會安裝一個 Apache 或 Nginx 伺服器，學會這招！你將不再需要為了開發把自己的機器弄髒了。(當然 Grunt 也能辦到請[參考](http://andyyou.logdown.com/posts/177346-grunt-init))

## 更好的方式
今時今日，Javascript 幾乎實作了所有的東西，包含一個 Web Server，其中最流行的一個叫做 [Connect](https://github.com/senchalabs/connect)，我們將使用它透過一個 Gulp 的擴充套件 `gulp-connect` 來完成我們的目的。
接下來的文章我們將會學習如何設定一個 Web Server，所以我假設你已經懂了基本的 Gulp 和 關於如何使用 `gulpfile.js`。

## 初始化設定

{% highlight bash %}
$ mkdir spa-website
$ cd spa-website
$ npm init
$ npm install --save-dev gulp
$ npm i -D gulp-connect  # 縮寫
{% endhighlight %}

現在我們可以來定義 Web Server 的任務了，我們的 `gulpfile.js` 看起來會像下面

{% highlight js %}
var gulp = require('gulp'),
  connect = require('gulp-connect');
 
gulp.task('webserver', function() {
  connect.server();
});
 
gulp.task('default', ['webserver']);
{% endhighlight %}

超級直覺的吧！現在你可以啟動一個 Web Server ，透過在 terminal 執行 gulp 指令，然後開啟瀏覽器輸入 `localhost:8080` 如果你在目錄中有 index.html 你就能看到網頁。這個小型的 Web Server 是掛載在 gulpfile.js 放置的目錄之上也就是 `localhost:8080` 的根目錄，換個角度說吧！通常我們會在一個目錄組織自己的 nodejs 專案，而這個 gulpfile.js 要放在專案的根目錄，因為你會需要 `require` `node_module` 裡面的 gulp，同時也方便你組織路徑。一旦你執行 `gulp` 指令這個 Server 就會啟動直到你使用 `Ctrl` + `c` 結束這個任務。
如果你已經有經驗的話那也可以直接查閱 [Github](https://github.com/schickling/gulp-webserver-article) 看看範例。每一個範例包含了所有待會實作的結果，你只需要下 `npm install` 安裝所有相依的套件即可使用。

## 加入 `livereload` 
如上面所看到的使用 gulp 建立一個 Web Server 是非常簡單的。下一步讓我們繼續來加入 `livereload` 支援，這樣一來當我們更新頁面的內容瀏覽器就會自動更新就不用一直手動更新頁面。
我們需要做兩件事讓 gulp 的 Web Server 支援 livereload ，第一件事情是設定 Web Server 就是我們的 gulp-connect 支援 livereload ，接著我們必須通知 `livereload` 何時要更新頁面。
第一個步驟非常簡單直接設定 `livereload: true` 設定 `server` 這個任務如下

{% highlight js %}
gulp.task('server', function () {
  connect.server({
    livereload: true,
  });
});
{% endhighlight %}

第二個步驟則視您的需求，在這個範例我們會設定自動編譯 LESS 檔案為 CSS 然後把更新的內容提交給瀏覽器。讓我們來看看這個範例的部分。我們需要一個偵測器，這個東西的用途是判斷 LESS 檔案是否被變更，接著這個偵測器會去觸發 LESS 編譯的任務，然後就會輸出 CSS 檔案，接下來這個檔案應該要透過 `livereload` 注入。

根據我們的需求我們會需要使用 `gulp-less` 這個外掛是用來編譯 LESS 的，同樣的你需要透過 `npm install --save-dev gulp-less` 去安裝這個套件。而所謂的偵測器已經附加在 gulp.js 裡面了，我們這個應用程式的架構大致上像這樣

~~~~~
├── node_modules
│   └── ...
├── styles
│   └── main.less
├── gulpfile.js
├── index.html
└── package.json
~~~~~

剛剛我們提到了偵測器，我們這邊用的就是 `gulp.watch()` 這個方法，他的功用就如上面說的當觀察到某個檔案產生變化，通常就是你存檔的瞬間，它就可以去觸發你設定的任務。在這個範例中就是一個 `watch` 的任務，Gulp.js 會一直監聽所有在 `styles` 目錄中符合 `*.less` 的檔案(就是任意名稱且附檔名為 less 的檔案)，接著當他看到檔案改變了就要去觸發 `less` 任務，這邊我們編譯的就是 `main.less` 這個 LESS ，每一次執行完編譯的動作，其結果將自動被注入到瀏覽器。
關於 `gulpfile.js` 看起來會像下面這樣

{% highlight js %}
var gulp = require('gulp'),
    connect = require('gulp-connect')
    less = require('gulp-less');

gulp.task('server', function () {
  connect.server({
    livereload: true
  });
});

gulp.task('less', function () {
  gulp.src('styles/*.less')
      .pipe(less())
      .pipe(gulp.dest('styles'))
      .pipe(connect.reload());
});

gulp.task('watch', function () {
  gulp.watch('styles/*.less', ['less']);
});

gulp.task('default', ['less', 'server', 'watch']);
{% endhighlight %}

完成設定之後回到 `Terminal` 執行 `gulp` 然後在瀏覽器開啟 `localhost:8080`。我們現在可以對 `LESS` 做一些修改然後存檔，在這一瞬間 gulp 已經完成編譯並且 refresh 您的瀏覽器了，注意到這邊這個方式並不需要額外安裝 `livereload` 的瀏覽器擴充元件。`livereload` 以不同的方式運作。

## 調整設定
注意上面的 gulpfile.js 只是一個小小的示範，以讓你可以快速理解如何建立一個 `gulp` 搭配 `gulp-connect` 支援 `livereload` 的 Web Server。我非常建議您可以搜尋一些其他的套件將它們組合出一些其他的任務，你應該試著重組這些任務結構，並且試試非內建的 `gulp-watch` 套件，它可以讓你處理剛剛變更過的檔案，隨著你要處理的任務越來越多這將會非常有幫助。讓我們簡單地用實際範例說明一下吧！你會發現除了 `LESS` 變更會自動更新，如果你是變更 HTML 那 livereload 並不會有任何反應，為什麼？因為你並沒有觀察 html ，所以你可能會像下面這樣

{% highlight js %}
var gulp = require('gulp'),
    connect = require('gulp-connect'),
    less = require('gulp-less');

gulp.task('server', function () {
  connect.server({
    livereload: true
  });
});

gulp.task('less', function () {
  gulp.src('styles/main.less')
      .pipe(less())
      .pipe(gulp.dest('styles/'))
      .pipe(connect.reload());
});

gulp.task('html', function () {
  gulp.src('*.html')
      .pipe(connect.reload());
});

gulp.task('watch', function () {
  gulp.watch('styles/*.less', ['less']);
  gulp.watch('*.html', ['html']);
});

gulp.task('default', ['less', 'server', 'watch']);
{% endhighlight %}

這個時候如果改用 `gulp-watch` 那麼設定檔就會像下面這樣

{% highlight js %}
var gulp = require('gulp'),
    connect = require('gulp-connect'),
    less = require('gulp-less'),
    watch = require('gulp-watch');

gulp.task('server', function () {
  connect.server({
    livereload: true
  });
});

gulp.task('less', function () {
  gulp.src('styles/*.less')
      .pipe(watch(function (f) {
        return f.pipe(less())
                    .pipe(gulp.dest('styles/'))
                    .pipe(connect.reload());
      }));
});

gulp.task('html', function () {
  gulp.src('*.html')
      .pipe(watch(function (files) {
          files.pipe(connect.reload());
      }))
});
gulp.task('default', ['less', 'html', 'server']);
{% endhighlight %}

你就不需要再多一個 watch 的任務了。兩種方式您都可以使用端看您的偏好。

## 設定 hostname 和 port 號
關於 `gulp-connect` 他其實有很多設定選項，在這個範例中我們將設定一個 hostname 和使用 80 port 這樣一來你就可以少打一點字

{% highlight js %}
connect.server({
	post: 80,
  host: gulp.dev
})
{% endhighlight %}

先別急著執行，為了讓你設定的 hostname 能夠運作你必須要先將這個網域名稱加到你的 `hosts` 

~~~~~
127.0.0.1	gulp.dev
~~~~~

然後執行 `sudo gulp` 必須要有 sudo 權限才能使用 80 port 

## 其他進階的功能
你甚至可以使用 `connect` 套件在同一時間產生多個 Web Server 服務，這對您的開發也是有幫助的舉例來說你可以執行一個開發用的 Server 然後同時執行整合測試。
`gulp-connect` 也允許您使用多個目錄為根目錄，舉例來說當你使用 `CoffeeScript` 你會希望把一些編譯的 Javascript 檔案放在 .tmp 暫存目錄然後把這個目錄 mount 上來當根目錄用，如此你就不會把你的專案目錄弄髒了。

## 重構
在前面的範例我們只不過小試了一下用 gulp 來編譯 LESS 然後使用 gulp-connect 搭配 livereload ，實際上我們可以做的更好。
當混合多種編譯和 `livereload` 到同一個任務的時候有可能會有一些問題和導致整個設定越來越混亂，所以讓我們來把他們分離並且讓偵測器在檔案一變更的時候就直接做些處理，為了達到這個目的，我們會需要使用剛剛提到的 `gulp-watch`。

然後我們加入編譯 CoffeeScript 的任務，下面這個新的設定結構將會更加清楚。
首先我們先安裝需要的套件

{% highlight bash %}
$ npm i -D gulp-watch
$ npm i -D gulp-coffee
{% endhighlight %}

安裝好之後 `require` 這些套件函式庫，在下面這些步驟中我假設你已經有一些 `.coffee` 的檔案在 `scripts` 目錄，如果沒有你可以到[這邊](https://github.com/schickling/gulp-webserver-article/tree/master/03-livereload-refactored)下載。重構後的 gulpfile.js 如下

{% highlight js %}
var gulp = require('gulp'),
    connect = require('gulp-connect'),
    less = require('gulp-less'),
    watch = require('gulp-watch'),
    coffee = require('gulp-coffee');

gulp.task('server', function () {
  connect.server({
    root: ['.', '.tmp'],
    livereload: true
  });
});

gulp.task('livereload', function () {
  gulp.src(['.tmp/styles/*.css', './tmp/scripts/*.js', '*.html'])
      .pipe(watch())
      .pipe(connect.reload());
});

gulp.task('less', function () {
  gulp.src('styles/main.less')
      .pipe(less())
      .pipe(gulp.dest('.tmp/styles'));
});

gulp.task('coffee', function () {
  gulp.src('scripts/*.coffee')
      .pipe(coffee())
      .pipe(gulp.dest('.tmp/scripts'));
});

gulp.task('watch', function () {
  gulp.watch('styles/*.less', ['less']);
  gulp.watch('scripts/*.coffee', ['coffee']);
});

gulp.task('default', ['less', 'coffee', 'server', 'livereload', 'watch']);
{% endhighlight %}

這個版本最大的改變是我們增加了 `livereload` 任務，這個任務透過 gulp-watch 來觀察那些已經被編譯完成的檔案，只要他們有變動就幫忙 reload 。套件提供的 `watch()` 函式允許我們只重載那些變更的檔案，而內建的 gulp.watch() 將會重載所有檔案，而不是只有變更的檔案。
因為這個額外的任務我們不需要在每個任務後面加上 `.pipe(connect.reload())` ，所以我們達到了任務的關注點分離，比起全部混在一起這可能是比較好的做法。
我們同時也注意到那些被編譯的檔案不是存在對應的原始碼目錄中，他們被放在暫存的目錄裡 `.tmp` 這樣的好處是這些編譯的檔案不會污染 `styles`, `scripts` 目錄，並且在實務上我們通常會排除這個 `.tmp` 目錄進入我們的版控。但我們又直接把它 mount 到了根目錄如此一來程式碼路徑就不用變更了。

## 結論
你已經學會了關於 Gulp 如何設定為你的開發伺服器了。你可以結合這個技術到各種專案去，注意這個 Web Server 只適合當作本地開發機如果是產品，你還是應該使用像是 Nginx 這類的伺服器。
任何剛剛做到的任務你都可以使用 `Grunt` 達成。只不過 Gulp 設定起來對程式設計師比較直覺。
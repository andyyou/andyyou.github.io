---
layout: post
title: 'Grunt 系列(3) 範例實作'
date: 2013-10-01 19:13:00
categories: Javascript F2E Grunt
---

下面我們將透過一個使用了 5 個常用套件的 `Gruntfile` 來實作練習並討論關於 Gruntfile。

* [grunt-contrib-uglify](https://github.com/gruntjs/grunt-contrib-uglify) : 壓縮檔案
* [grunt-contrib-qunit](https://github.com/gruntjs/grunt-contrib-qunit) : 單元測試
* [grunt-contrib-concat](https://github.com/gruntjs/grunt-contrib-concat) : 檔案合併
* [grunt-contrib-jshint](https://github.com/gruntjs/grunt-contrib-jshint) : 檢查 Javascript 語法
* [grunt-contrib-watch](https://github.com/gruntjs/grunt-contrib-watch) : 觀察檔案變更

整個完整的 `Gruntfile` 在頁面的最下方，不過如果你按順序閱讀，這篇文章會一步一步的說明。

>備註：通常在使用套件前，比較方便的方式是透過下面指令來安裝，這樣一來也順便會更新 `package.json` 。一般來說執行 grunt 都是在開發階段所以參數會使用 --save-dev 就是設定在 `package.json` 的 `devDependencies` 中。

{% highlight bash %}
$ npm install grunt-contrib-uglify --save-dev 
{% endhighlight %}

首先要說明的是關於 `wrapper` 函式，簡單的來說所有關於 `grunt` 該執行的任務和相關設定都會被封裝在這個 function中。如下程式碼

{% highlight js %}
module.exports = function(grunt) {
    // 所有任務設定都會在這。
}
{% endhighlight %}

接著我們通常會在這個 function 中開始初始化我們的`設定檔物件`，就會透過 `grunt.initConfig` 把資料放到物件`{}`中，如下：

{% highlight js %}
{% raw %}
grunt.initConfig({
	// 屬性: "資料" 
});
{% endraw %}
{% endhighlight %}
        
為了能夠使用一些關於專案的資料，我們可以讀取 `package.json` 的資料到 `pkg` 屬性，於是我們就可以透過 `pkg` 來取得專案的資料，例如： `pkg.name` 就是我們在 `package.json` 設定的專案名稱。設定的方式如下

{% highlight js %}
pkg: grunt.file.readJSON('package.json');
{% endhighlight %}

到目前為止的設定就會如下：

{% highlight js %} 
module.exports = function(grunt){
	grunt.initConfig({
	   pkg: grunt.file.readJSON('package.json');
  });
}
{% endhighlight %}
        
現在，我們就可以針對每一個任務來作設定，在`設定檔物件`中，也就是在 `grunt.initConfig(\{\})` 的 `{}` 中所有存在的任務通常會對應一個屬性，它是透過相同的名稱來對應的。所以例如 `concat` 任務的設定就會在 `concat`  屬性中設定。下面就是關於 `concat` 任務的設定：

{% highlight js %}
concat: {
	options:{
		// 合併不同檔案時會在檔案和檔案之間加入 ; 
		separator: ';'
	},
	dist:{
		// 要合併的檔案
		src: ['src/**/*.js'],
		// 產生的檔案路徑
		dest: 'dist/<%= pkg.name %>.js'
	}
}
{% endhighlight %}

注意到這邊我們引用了專案名稱 `name` 屬性，透過 `grunt.file.readJSON` 我們把 `package.json` 讀入成為一個物件，並且賦予 `pkg` ，如此一來我們就可以很輕鬆地存取關於 `package.josn` 中的設定。 Grunt 內建一個簡單的樣板引擎，讓我們在設定檔中能夠輕鬆的內嵌變數，通常是屬性的值（例如 `pkg.name`）。上面任務的意思就是取得 `src/` 目錄底下所有 `.js` 結尾的檔案，然後合併成一個 `dist/[專案名稱].js` 。 

接著讓我們來看看 `uglify` 任務，它是用來壓縮 Javascript 的套件。

>dist = Distribution

{% highlight js %}
uglify: {
    options: {
				// 產生一段 banner 文字並加入到將產生檔案的一開始
				banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */\n'
		},
		// 這邊的 dist 是任務底下的一個目標（target）
		// 一般情況下執行 grunt uglify 所有的目標都會被執行。
		dist: {
				files: {
        		// 回憶一下上一篇 dist:src 前面是目的路徑，後面是來源路徑，注意來源是 concat 的屬性
    				'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
				}
    }
}
{% endhighlight %}
     
上面 `uglify` 任務取得 `concat` 合併後的檔案，並壓縮到 `dist` 目錄。讓我們復習一下上面 `files` 是使用檔案物件格式，檔案的設定方式有三種。

`qunit` 的設定則非常單純，只要把運行測試頁的位置設定即可。

{% highlight js %}
qunit: {
	files: ['test/**/*.html']
}
{% endhighlight %}
        
JSHint 的設定也很簡單如下
 
{% highlight js %}
{% raw %}
jshint: {
	// 檢查的檔案
	files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
	// 設定 JSHint 屬性查詢文件(http://www.jshint.com/docs/options/)
	options: {
		globals: {
			jQuery: true,
			console: true,
			module: true
		}
	}
}
{% endraw %}
{% endhighlight %}
        
JSHint 就是根據 files 的設定去檢查檔案，而 options 可以用來改寫預設的規則。如果你要使用預設值則不用設定。

最後我們還有一個 `watch` 任務

{% highlight js %}
{% raw %}
watch: {
		files: ['<%= jshint.files %>'],
		tasks: ['jshint', 'qunit']
}
{% endraw %}
{% endhighlight %}
        
當你使用 `grunt watch` 命令時，它就會去觀察 `jshint.files` 的檔案列表。就是 `['gruntfile.js', 'src/**/*.js', 'test/**/*.js']` 。當它偵測到指定的這些檔案有變更，他就會去執行任務 `tasks` 。在這邊我們設定了執行 `hshint` 和 `qunit` 兩個任務。
 
讓我們再次提醒，上面都是透過 `grunt.initConfig` 設定的組態，因此我們還是要載入這些套件。載入之前請先記得安裝。

{% highlight js %}
grunt.loadNpmTasks('grunt-contrib-uglify');
grunt.loadNpmTasks('grunt-contrib-jshint');
grunt.loadNpmTasks('grunt-contrib-qunit');
grunt.loadNpmTasks('grunt-contrib-watch');
grunt.loadNpmTasks('grunt-contrib-concat');
{% endhighlight %}

最後我們需要把常用的任務設定為 `default` 這樣一來我們以後只要下 `grunt` 指令就好，就不用在特別下 `grunt uglify` 之類的。

{% highlight js %}
// 註冊 test 任務之後只要執行 "grunt test" 就會跑 jshint 和 qunit
grunt.registerTask('test', ['jshint', 'qunit']);

// 註冊預設任務之後只要執行 "grunt" 就會跑下面陣列中的任務。
grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);
{% endhighlight %}

 下面就是完整的 `Gruntfile` ，通常我們使用 `.js` 或 `.coffee`
 
{% highlight js %}
module.exports = function(grunt){
	grunt.initConfig({
			pkg: grunt.file.readJSON('package.json'),
			concat: {
  				options: {
      				separator: ';'
  				},
  				dist: {
      				src: ['src/**/*.js'],
      				dest: 'dist/<%= pkg.name %>.js'
  				}
			},
			uglify: {
  				options: {
      				banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */\n'
  				},
  				dist: {
      				files: {
          				'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
      				}
  				}
			},
			qunit: {
  				files: ['test/**/*.html']
			},
			jshint: {
  				files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
  				options: {
      				globals: {
          				jQuery: true,
          				console: true,
          				module: true,
          				document: true
      				}
  				}
			},
			watch: {
  				files: ['<%= jshint.files %>'],
  				tasks: ['jshint', 'qunit']
			}
	});

	grunt.loadNpmTasks('grunt-contrib-uglify');
	grunt.loadNpmTasks('grunt-contrib-jshint');
	grunt.loadNpmTasks('grunt-contrib-qunit');
	grunt.loadNpmTasks('grunt-contrib-watch');
	grunt.loadNpmTasks('grunt-contrib-concat');

	grunt.registerTask('test', ['jshint', 'qunit']);
	grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);
};
{% endhighlight %}
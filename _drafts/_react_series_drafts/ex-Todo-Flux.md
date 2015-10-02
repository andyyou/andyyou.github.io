# Flux 與 React ToDoMVC
上面對於 Flux 概念的說明有些過於理論，接下來這篇文章我希望以實務為主軸帶領大家一步一步建立一個 Flux 的應用，在 nodejs 裡同樣
一件事有非常多的做法與選擇，而這邊我展示了其中一種。老實說學習 nodejs 有一部分困擾的地方
就是在 nodejs 的世界裡有著各式各樣的函式庫，套件與不同思維的做法。而我們能做的大概就是不斷的學習
理解找出最適合自己需求的解決方案。光配置任務這件事我們就至少有三種選擇
1. npm package.json
2. Grunt
3. gulp

而組織程式碼的風格又有 AMD, CommonJS 等等，您可以選擇 requirejs, Sea.js 等等。
這次我打算使用 Grunt 搭配 Broswerify。首先讓我們根據下面的步驟來建立我們基本的專案架構。

## 1. 建立 package.json

```
npm init
```

## 2. 修改 package.json 加上相依套件

```js
{
  "name": "flux-todo",
  "version": "0.0.1",
  "description": "TodoMVC with Flux",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "AndyYou",
  "license": "MIT",
  "devDependencies": {
    "grunt": "^0.4.5",
    "grunt-browserify": "^3.0.1",
    "grunt-contrib-clean": "^0.6.0",
    "grunt-contrib-connect": "^0.8.0",
    "grunt-contrib-watch": "^0.6.1",
    "grunt-react": "^0.9.0"
  },
  "dependencies": {
    "flux": "^2.0.1",
    "react": "^0.11.2"
  }
}
```

## 3. npm install 安裝所有相依套件

## 4. 建立 Gruntfile.js
我們將透過 Grunt 整合 borwserify 和 React。

```js Gruntfile.js
module.exports = function (grunt) {
  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),
    connect: {
      server: {
        options: {
          port: 8000,
          hostname: '*',
          base: '.',
          protocol: 'http'
        }
      }
    },

    watch: {
      options: {
        livereload: true
      },
      react: {
        files: ['js/app.js'],
        tasks: ['browserify']
      }
    },

    browserify: {
      options: {
        transform: [require('grunt-react').browserify]
      },
      react: {
        src: ['js/app.js'],
        dest: 'js/bundle.js'
      }
    }

  });
  grunt.registerTask('default', ['browserify', 'connect', 'watch']);
  grunt.loadNpmTasks('grunt-contrib-connect');
  grunt.loadNpmTasks('grunt-contrib-watch');
  grunt.loadNpmTasks('grunt-browserify');
}

```

完成基本的任務配置之後，我們先來配置這個專案的架構，一個 Flux 架構預設上會如下：

```
flux-todo
  |
  + ...
  + js
    |
    + actions
    + components // 所有 React 元件都放置在這個目錄
    + constants  // 為了讓事情保持單純這個範例將不使用 constants
    + dispatcher
    + stores
    + app.js
    + bundle.js
  + index.html
  + ...
```
於是我們如下建立這些目錄

![](http://i.imgur.com/Hp2heTt.png)

## 5. AppDispatcher

```
var Dispatcher = require('flux').Dispatcher;
var copyProperties = require('react/lib/copyProperties');
// copyProperties 將第二個參數合併到第一個參數物件
var AppDispatcher = copyProperties(new Dispatcher(), {
  handleViewAction: function (action) {
    this.dispatch({
      source: 'VIEW_ACTION',
      action: action
    });
  }
});

module.exports = AppDispatcher;
```

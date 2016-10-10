---
layout: post
title: 'NodeJS path'
date: 2015-12-17 05:10:00
categories: Program
tags: [javascript, nodejs]
---

範例筆記

```

/**
 * 詳細教學
 * http://www.tutorialspoint.com/nodejs/nodejs_path_module.htm
 */

// 正常化
path.normalize('/1/../2'); // => /2

// 把所有路徑整合在一起並且正常化
path.join('a', 'b'); // =>'a/b'
path.join('a', './b'); // => a/b'
path.join('a', '/b'); // =>'a/b'
path.join('/1', '/2/3', '../4'); // => /1/2/4


// 從第一個路徑，照著後面切換最後回傳絕對路徑
path.resolve('/from', '/to/path1', '/to/path2'); // => /to/path2
path.isAbsolute('/'); // => true

// 從 from 到 to 的相對路徑
path.relative('..', './A/B'); // => [project]/A/B

// 回傳該檔案所在的目錄
path.dirname('node_modules/bin/webpack'); // => node_modules/bin
path.dirname('./node_modules/bin/webpack'); // => ./node_modules/bin

// 回傳路徑最後的部分，包含副檔名
path.basename('node_modules/bin/webpack'); // => webpack
path.basename('/1/2/3.js'); // => 3.js

// 只取副檔名 + .
path.extname('/1/2/3.js'); // => .js

// 將路徑解析成物件
path.parse('/1/2/3.js'); // =>

// 從物件轉成字串
path.format({ root: '/', dir: '/1/2', base: '3.js', ext: '.js', name: '3' }); // => /1/2/3.js

```

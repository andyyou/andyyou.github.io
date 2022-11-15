---
title: 'Inertia - Server-side Rendering (SSR) 以 React 為範例'
date: 2022-04-11 15:18:28
tags:
  - javascript
  - react
  - laravel
categories: Program
---

> 本文為官方文件翻譯，如使用其他前端框架可參考[官方](https://inertiajs.com/server-side-rendering)

SSR 支援提前渲染造訪的頁面，並且回傳渲染的 HTML 到瀏覽器。這讓造訪者可以在資源完整載入之前看到並和頁面互動，另外提供的好處像是減少搜尋引擎分析建置索引的時間，優化 SEO。

<!-- more -->

> SSR 尚未支援 Svelte 橋接器。

## 運作機制

當 Inertia 偵測到在 Node.js 環境下運行時，會自動渲染 [頁面物件](https://inertiajs.com/the-protocol#the-page-object) 為 HTML 並回傳。

然而因為大部分 Inertia 應用程式使用像是 PHP 或 Ruby 語言建置，我們會需要使用另一個分開的 Node.js 服務來處理請求，才能渲染頁面回傳 HTML。

## 設定 Server-side Rendering

首先我們需要使用 npm 安裝必要的相依套件

```sh
$ npm i react-dom
```

同時需要安裝 `@inertiajs/server`，它提供了一個簡單的 HTTP 伺服器。雖然這個套件不是絕對必要的，它能避免使用變更自有 HTTP 服務所需，一般來說可以讓事情單純一些

```sh
$ npm i @inertiajs/server
```

接著我們需要建立 `resources/js/ssr.js` 檔案

```sh
$ touch resources/js/ssr.js
```

這個檔案會非常類似 `app.js` ，差別是這不是給瀏覽器使用的，而是 Node.js 下面為完整範例

```js
import React from 'react';
import ReactDOMServer from 'react-dom/server';
import { createInertiaApp } from '@inertiajs/inertia-react';
import createServer from '@inertiajs/server';

createServer((page) =>
  createInertiaApp({
    page,
    render: ReactDOMServer.renderToString,
    resolve: (name) => require(`./Pages/${name}`),
    setup: ({ App, props }) => <App {...props} />,
  })
);
```

> 請確認加入任何在 `app.js` 但這裡缺少的東西例如其他套件或自訂的 mixins。不過，並不是所有東西都需要加入，舉例來說 `InertiaProgress` 就可以被忽略，因為它不會在 SSR 模式被使用。
>
> 此外，不要在 `ssr.js` 使用拆分程式碼，這不會有任何幫助 (`mix.extract()`) 。當然客戶端 `app.js` 您完全可以使用。

> 預設 Inertia SSR 伺服器會使用埠 `13714` 。但您可以在 `createServer` 的第二個參數調整。

> Vue2 如果您使用 `PortalVue` 套件，它必須要在 `import {createRenderer} from 'vue-server-renderer'` 後面。

## 設定 Laravel Mix

為了讓我們的 Webpack 編譯的東西在 Node 正確執行，我們需要安裝 `webpack-node-externals`

```sh
$ npm i webpack-node-externals
```

然後，建置一個新的設定檔 `webpack.ssr.mix.js` 。因為目前使用的 Laravel Mix 不支援在 `webpack.mix.js` 提供多個設定。

```sh
$ touch webpack.ssr.mix.js
```

這裡提供一個範例。注意它看起來和 `webpack.mix.js` 非常類似，不同的地方在於它只編譯 JavaScript 不編譯 CSS。

確認定義的任何 `alias` 都要記得。接著使用 `webpackConfig()` 設定 `target` 為 `node` 和 `externals` 使用 `[webpackNodeExternals()]`

```js
const path = require('path')
const mix = require('laravel-mix')
const nodeExternals = require('webpack-node-externals')

mix
	.options({ manifest: false })
	.js('resources/js/ssr.js', 'public/js')
	.react()
	.alias('@': path.resolve('resources/js'))
	.webpackConfig({
    target: 'node',
    externals: [nodeExternals()],
  })
```

## 支援 Server-side Rendering

> 自動 SSR 渲染目前只支援 Laravel adapter。不過，我們正積極的開發 Rails adapter。

下一步我們需要確認 `@inertiaHead` 被包含在 `app.blade.php` 的 `<head>`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, maximum-scale=1.0"
    />
    <link href="{{ mix('/css/app.css') }}" rel="stylesheet" />
    <script src="{{ mix('/js/app.js') }}" defer></script>
    @inertiaHead
  </head>
  <body>
    @inertia
  </body>
</html>
```

最後，我們需要在 `inertia.php` 啟動 SSR。如果您在 `config` 目錄下海沒有這支檔案可以執行下面指令

```sh
$ php artisan vendor:publish --provider="Inertia\ServiceProvider"
```

然後設定 `ssr` 選項下的 `enabled` 為 `true`

```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Server Side Rendering
    |--------------------------------------------------------------------------
    |
    | These options configures if and how Inertia uses Server Side Rendering
    | to pre-render the initial visits made to your application's pages.
    |
    | Do note that enabling these options will NOT automatically make SSR work,
    | as a separate rendering service needs to be available. To learn more,
    | please visit https://inertiajs.com/server-side-rendering
    |
    */

    'ssr' => [

        'enabled' => true,

        'url' => 'http://127.0.0.1:13714/render',

    ],

// ...
```

## 建置應用程式

現在您有兩個建置流程 - 一個是客戶端, 一個是伺服器端

```sh
$ npx mix
$ npx mix --mix-config=webpack.ssr.mix.js
```

執行兩個建置指令並確認沒有任何錯誤。記住您現在建置的是一個 "Isomorphic" 同構應用程式，意思是您的應用程式同時可以在瀏覽器和伺服器上執行。

> 「如果兩個結構是同構的，那麼其上的對象會有相似的屬性和操作，對某個結構成立的命題在另一個結構上也就成立。」簡單來說，同構的意思指的是兩個實體是不相同的，但具有相似的操作行為。

## 執行 Node.js 服務

當檔案建置完成，您應可以使用下面指令執行

```sh
$ node public/js/ssr.js
```

一旦執行成功，您的應用程式應可支援 SSR 了。實際上您應該要刻意關閉 JavaScript 還能夠瀏覽應用程式。

## 客戶端 Hydration (僅支援 Vue)

搭配這些設定，Vue 會自動使用 hydrate 靜態標籤，而不是全部重新渲染。

然而客戶端 hydrate 要正常運作，伺服器產生的 HTML 必須要符合，否則會看到警告。

正常情況下您產生的頁面元件應該是一致的，如果您看到警告可以參考 Vue 官方文件的說明。

## 佈署設定

當佈署支援 SSR 應用程式時，您會需要同時執行 `app.js` 和 `ssr.js`。其中一個方式是更新 `package.json` 的 `prod`

```json
"prod": "mix --production && mix --production --mix-config=webpack.ssr.mix.js",
```

### Laravel Forge

要在 Forge 啟用 SSR 伺服器，需建立新的 daemon 執行 `node public/js/ssr.js`

記下 daemon ID 您需要在部署 script 中使用。每當您佈署應用程式時，您需要自動重新開啟 SSR 伺服器。將下面 script 加入

```sh
# Restart SSR server
$ sudo supervisorctl restart daemon-123456:daemon-123456_00
```

### Heroku

要在 Heroku 啟動 SSR 伺服器則要更新 `Procfile` 的 `web` 。先執行 SSR 伺服器在啟動一般伺服器。注意要完成這個操作您需要 `heroku/nodejs` buildpack。

```
web: node public/js/ssr.js & vendor/bin/heroku-php-apache2 public/
```

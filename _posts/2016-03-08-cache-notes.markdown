---
layout: post
title: 'Cache 筆記'
date: 2016-03-08 19:00:00
categories: web
---

* 瀏覽器 cache 是一種機制，透過特定狀態告知瀏覽器不需要重新下載檔案
* cache 最早使用 Expires 和 Pragma，現今主要使用 Cache-Control 來控制
* cache 大略流程，在實際下載檔案之前發生：
  0. cache 的實際處理機制存在瀏覽器中，也就是我們需要透過指令(cache-directives)告訴瀏覽器該怎麼處理
  1. 瀏覽器設定為不提供 cache ，完全忽略下面步驟直接請求資源
  2. 瀏覽器發送請求，此時可包含 cache-request-directive 這部分的資訊主要是拿來和 server 協商比對用的
  3. 伺服器判斷是否有更新並回傳通知 cache-response-directive
  4. 不需更新則回傳 304 Header
* 2 至 3 步驟用來判斷是否發出下載資源的請求，如果在 1 的狀況則發出請求，因為不帶任何 Date, Etag 資訊所以就會重新拉一份回來
![](http://i.imgur.com/4mjsAuc.png)
* 常用 directives 列表 - Cache-Control
  1. no-store 完全不存 cache
  2. no-cache 會存 cache 但是每次重新請求 = 無視 cache
  3. max-age 判斷是否要重新請求檔案，在一定秒數之內不需要重新請求，除非遇上 max-stale
  4. private | public 判斷該使用者能否使用
  5. max-stale 指定資源過期
  6. must-revalidate 定義上類似 no-cache 但是更嚴格，會跟原始 server 比對而不是 proxy 。如果 max-stale 過期整份文件包含子請求都會更新。實際運作則是交由瀏覽器自行判斷是否更新。
  7. s-maxage 用於 CDN 如果 max-age 也存在會覆寫 max-age
* 簡言之，除了瀏覽器直接關閉 cache 功能外，其他都需要和 server 端協商並透過 cache-directives 決定行為
* 除了 Cache-Control 還有 Date, Etag 驗證權杖, Last-modified 用來資訊判斷是否重新請求(現今多使用 Cache-Control)
  1. Etag - If-None-Match: [hash]
  2. Date(傳送), Last-modified(回覆) - If-Modified-Since
* Etag 格式分成寬鬆和嚴謹

~~~
ETag: "1234abcd" 嚴謹
ETag: W/"1234abcd" 寬鬆
~~~

* 在 client 端過期的資料本應要更新，但如果 server 端的資料也沒有更新，我們就沒有理由再去下載已存在於快取中相同的資料，因此 Etag 就是用來確認這件事
* 關於 Etag 我們唯一要做的就是確認 server 有提供此功能，之後 browser 會幫我們處理後續流程

~~~
client   -->   server
client   <--   server
          Etag: 'x234dff'
          Cache-Control: max-age=120 單位是秒
          ...
client   -->   server
          拿著資料去 server 比對
~~~

### CDN

CDN 就是卡在 server 與 users 之間的另外的 server 我們將網址的  Name Server 指向 CDN。CDN 會到我們的 server cache 資料，協助我們管理 DNS & Cache 的部分，也擋在我們服務之前所以偶而可以協助擋 DDoS

* [A Beginner's Guide to HTTP Cache Headers](http://j.mp/1QCzEho)
* [淺談 cache](http://www.alloyteam.com/2016/03/discussion-on-web-caching/) - ★★★★★ 推薦
* [Google - HTTP 快取](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)
* [Cache Control 與 ETag](https://blog.othree.net/log/2012/12/22/cache-control-and-etag/)
* [初探 HTTP 1.1 Cache 機制](http://blog.toright.com/posts/3414/%E5%88%9D%E6%8E%A2-http-1-1-cache-%E6%A9%9F%E5%88%B6.html)
* [w3c 規範](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)

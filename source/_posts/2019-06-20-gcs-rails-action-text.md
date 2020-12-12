---
title: Rails 6 Action Text 直接上傳圖片至 Google Cloud Storage 失敗
date: 2019-06-20 15:44:28
tags:
  - gcp
  - ruby
  - rails
categories: Cloud
---

直接先備份[參考解答](https://dev.to/morinoko/debugging-google-cloud-storage-cors-errors-in-rails-6-action-text-direct-upload-of-images-2445)。

<!-- more -->

小弟跟上文作者一樣在使用 Rails 6 的 Action Text 時發現圖片無法上傳 Google Cloud Storage，Rails 採用 Google App Engine 方式部署，上傳使用內建 Active Storage。

問題主要是遇到 cors 的錯誤，解法原文已有，本文只是自己的備份筆記，可參考原文更詳細。

```bash
$ gsutil cors get gs://my-bucket-name
```

無論您是否有取得設定或者遇到 `gs://my-bucket-name/ has no CORS configuration`，您都可以直接使用下面的方式把 `origin` 換掉設定即可。

> 輸入您的來源網址，您不一定是使用 App Engine

```json

[
  {
    "origin": ["https://app.hubspot.com"],
    "responseHeader": ["Content-Type", "Content-Md5"],
    "method": ["PUT", "GET", "HEAD", "DELETE", "OPTIONS"],
    "maxAgeSeconds": 3600
  }
]
```

完成 JSON 之後，下指令設定：

```bash
$ gsutil cors set cors.json gs://my-bucket-name
```

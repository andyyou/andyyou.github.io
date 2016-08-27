---
layout: post
title: "Facebook Bot 開發小技巧"
date: 2016-08-26 12:00:00
categories: js, html
---

## 流程

* 準備環境
* 使用 ngrok 取得 https 網址
* 建立與設定 Facebook App 與粉絲團
* 建立 webhook `GET` API
* 設定 Facebook App webhook
* 實作 webhook `POST` API 產生對話


## 準備

由於 Facebook bot 在執行 API 溝通的時候需要使用 SSL 協定即網址要是 `https`，為了在本地端開發方便。
因此我們需要使用 `ngrok` 來協助我們產生一個 `https` 的網址並傳回我們的機器。當然很多文章會介紹您使用 cloudflare 但這邊為了方便開發於是使用另外一種 `ssh tunnel` 的方式來解決

1. 註冊並登入 [ngrok](https://dashboard.ngrok.com/get-started)
2. 下載並安裝 `ngrok`
2. 完成安裝 ngrok 憑證的流程

```zsh
# 安裝可以下載檔案並放置到 /usr/lcoal/bin 或者使用 brew
$ brew cask install ngrok
# 登入後會看到 ngrok 提供的驗證指令
$ ngrok authtoken [your_token]

# 開始使用
$ ngrok http [port]
```

啟動之後畫面如下：

![Imgur](http://i.imgur.com/IBnruGZ.png)

## 建立 Facebook App 與粉絲專頁

* [官方教學](https://developers.facebook.com/docs/messenger-platform/product-overview/setup)

建立好 App 與粉絲團之後接下來為了`設定 webhook`，網址的部分請給定 `ngrok` 的網址，例如
`https://[hash_code].ngrok.io/webhook`。而 token 則是您自訂任意的字串，稍後需要在 API 中使用驗證。

因為設定 webhook 網址時，facebook 會發一個 request 給測試該網址是否正常運作，如果不正確的話，那就沒辦法進入下一階段了。
所以我們需要先將 `GET API` 建立完成

```js
app.get('/webhook/', function (req, res) {
  if (req.query['hub.verify_token'] === '<YOUR CUSTOM TOKEN>') {
    res.send(req.query['hub.challenge']);
  }
  res.send('Error, wrong validation token');
})
```

> 一旦設定了若要完全移除就是要讓 API 死掉 8H。

![Imgur](http://i.imgur.com/Px5x8Ae.png)

![Imgur](http://i.imgur.com/uaQxO5G.png)

上敘動作完成之後，我們就完成前置作業了。接著下來就是實作與機器人對談的部分

```js
var express = require('express')
var bodyParser = require('body-parser')
var request = require('request')
var app = express()
var token = '<YOUR_PAGE_TOKEN>'
var facebookApi = 'https://graph.facebook.com/v2.6/me/messages'

app.use(bodyParser.urlencoded({
  extended: true
}))
app.use(bodyParser.json())

app.get('/', function (req, res) {
  res.send('Hi, server')
})

app.get('/webhook/', function (req, res) {
  if (req.query['hub.verify_token'] === 'abcd') {
    res.send(req.query['hub.challenge']);
  }
  res.send('Error, wrong validation token');
})

app.post('/webhook/', function (req, res) {
  console.log(req.body.entry[0])
  var messages = req.body.entry[0].messaging

  for (var i = 0; i < messages.length; i++) {
    var event = messages[i]
    var sender = event.sender.id

    if (event.message && event.message.text) {
      var text = event.message.text
      sendTextMessage(sender, text)
    }
  }
  res.sendStatus(200)
})

app.listen(8080, function () {
  console.log('Listen on port 8080')
})

/**
 * Functions
 */

function sendTextMessage (sender, text) {
  messageData = {
    text: text
  }

  request({
    url: facebookApi,
    qs: { access_token: token },
    method: 'POST',
    json: {
      recipient: { id: sender },
      message: messageData
    }
  }, function (err, res, body) {
    if (err)
      console.log('Error sending message: ', err)
    else if (res.body.error)
      console.log('Error: ', res.body.error)
  })
}
```

到此我們就完成了一個簡單的 Facebook BOT 應用。

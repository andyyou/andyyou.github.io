---
title: RabbitMQ 學習筆記 - 安裝、入門、Work Queues
tags:
  - queue
  - rabbitmq
  - javascript
  - osx
categories: Program
date: 2017-06-08 18:13:11
---


本篇是關於 RabbitMQ 的入門學習筆記，內容從安裝到學習使用 Work Queue 的方式。能夠引導您快速入門。大部分的資料來自於官方的學習文件佐以實作時相關問題的資料補充。

<!--more-->

# OS X 使用 Homebrew 安裝 RabbitMQ

在開始安裝之前請先確保您的 brew 是最新版本：

```bash
$ brew update
```

接著安裝 RabbitMQ

```bash
$ brew install rabbitmq
```

# 執行 RabbitMQ Server

RabbitMQ server 的執行與操作指令會被安裝在 `/usr/local/sbin`。這個路徑並不會自動被加到我們電腦的 `PATH` 所以我們需要在 `.bash_profile` 或 `.profile` 加入 `PATH=$PATH:/usr/local/sbin`。

啟動 server 的指令為 `rabbitmq-server`。所有指令只需要使用者帳號的權限，不需執行 sudo。

> 如果您有使用 brew services 的話可以直接執行 `brew services start rabbitmq`。

# 預設使用者

仲介（Broker）預設會建立一個 `guest` 使用者，密碼是 `guest`。沒有設定的客戶端一般會直接使用這組帳密。
這組帳密只有當 Broker 在本機時可以使用，所以我們在從其他主機連線之前我們需要做些設定。所謂的 Broker 就是幫我們傳遞訊息的角色。

可以在[存取設定文件](https://www.rabbitmq.com/access-control.html)查詢到相關的資訊，如建立帳號，刪除 guest，甚至讓遠端可以存取 guest。

# OSX 設定系統限制

RabbitMQ 安裝程式預設的工作負載也許需要調整`系統限制`和`核心參數`，使其可以處理大量的併發連線和佇列（Queue)。最主要需要調整的設定就是開檔的最大數量，也就是 `ulimit -n`。大部分的作業系統預設值都不夠訊息仲介（Message Broker）使用，在一些 Linux 系統預設是 1024。
我們建議對於線上服務來說最少 `file descriptors` 是 `65536` 給 rabbitmq 使用，`4096` 應該可以滿足大部分開發時期使用。

通常有兩個地方限制關於開檔數：系統核心允許的最大開檔限制（kern.maxfilesperproc）和每個使用者的限制（ulimit -n），前者的設定會高於後者，也就是說 `kern.maxfilesperproc` 在有設定的情況下 ulimit 不可以超過它。

調整每個使用者限制的部分有幾個方式：

* 在開啟 RabbitMQ 之前執行 `ulimit -Sn 4096` （後續更新為該值，重開機或開新 shell 會回到預設）
* 修改 [rabbitmq-env.conf](http://www.rabbitmq.com/configure.html) 使其在 server 開啟之前執行 ulimit
* 透過 `launchctl limit /etc/launchd.conf` 調整最大開檔數量

注意：修改的設定不會影響已經執行的程式，調整 `launchctl limit` 需重開機。

更多關於使用 `sysctl` 設定 `kern.maxfilesperproc` 請參考 Apple 官方文件 [sysctl(8)](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/sysctl.8.html)。

> macOS Sierra 不作任何設定下預設 ulimit `file descriptors` 上限可調至 `10240(unlimited)` 使用者執行 `rabbitmq-server` 會使用 ulimit 的設定，但注意：如果您使用 brew services 則會採用 `launchctl limit` 即系統的設定。如果加入 `limit.maxfiles.plist` 調整系統限制，ulimit 最高不可超過該設定。為了讓後續開發時直接使用 brew services 我們建議直接調整 `launchctl limit`。另外在設定 soft limit 和 hard limit 時，如果 hard limit 小於 soft limit 那麼 `launchctl limit` 預設會變成 `maxfiles 256 unlimited`。

# macOS Sierra 調整 launchctl limit

新增 `/Library/LaunchDaemons/limit.maxfiles.plist`，並重新開機。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
        "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>limit.maxfiles</string>
    <key>ProgramArguments</key>
    <array>
      <string>launchctl</string>
      <string>limit</string>
      <string>maxfiles</string>
      <string>65636</string>
      <string>65636</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>ServiceIPC</key>
    <false/>
  </dict>
</plist>
```
# 查詢限制

我們一樣有許多方式來驗證 RabbitMQ 當前的限制是否如我們所設定。最簡單的方式就是透過[RabbitMQ management UI](https://www.rabbitmq.com/management.html)。

```bash
$ rabbitmq-plugins enable rabbitmq_management
# 瀏覽 http://localhost:15672
```

![](https://d2ppvlu71ri8gs.cloudfront.net/items/1z1O1c2I2e20072e3y1D/2017-05-28%2011-43-33.png)

再來就是使用指令的方式

```bash
$ rabbitmqctl status
```

兩者的資訊會是一致的。對於 macOS 的使用者則可以使用下列指令觀察使用者的限制。

```bash
$ launchctl limit
```

# 處理 node with name rabbit already running

```
$ rabbitmqctl stop

# OR

$ ps aux | grep epmd
$ ps aux | grep erl
$ kill -9 [pid]
```

# 入門

這一系列筆記主要是閱讀官方文件的記錄，內容涵蓋如何使用 RabbitMQ 建立基本的訊息，上面我們已經完成安裝了，接著我們就會使用 Javascript 系列來實作。
[官方教學](https://www.rabbitmq.com/getstarted.html)完整的提供各種語言的教學，如果您需要其他的語言請參考官方文件。

RabbitMQ 是個訊息仲介（Message Broker）上面第一次提到時，您恐怕是一頭霧水。簡單說就是它協助我們傳遞交換訊息。您可以把它想成是郵局，當我們把信件丟到郵筒時，我們幾乎可以確定郵差先生最終一定會幫我們把信件送給收件者。以這個比喻來說 RabbitMQ 就是郵筒、郵局、郵差的集合體。

唯一的差別是 RabbitMQ 不會真的寄出紙本，它使用二進制的文件類型來傳遞資料（Binary blobs）我們稱為 - Message。

當然 RabbitMQ 也有些術語是我們在使用之前最好先了解的。

`Producing` 就是傳送訊息。一個發送訊息的程式也被稱為 `producer` 

![](https://www.rabbitmq.com/img/tutorials/producer.png)

`queue` 佇列在 RabbitMQ 裡其實就是郵筒的意思。雖然訊息在 RabbitMQ 和我們的應用程式之間傳遞但它們只會被存放在 `queue`。`queue` 本身只會被主機的記憶體和硬碟所限制，本質上就是一個很大的訊息緩衝區。可以有很多個 producer 發送，但它們都會傳到同一個 `queue` 並且可以有多個 `consumer` 可以從 `queue` 取得資料。

![](https://www.rabbitmq.com/img/tutorials/queue.png)

`Consuming` 概略來說就是接收的意思。`consumer` 大致上說來就是一個等待接收訊息的程式。

![](https://www.rabbitmq.com/img/tutorials/consumer.png)

現在我們介紹了產生者（producer）、接收者（consumer）、和仲介（broker）。它們並不需要存在同一台主機上，確實在大多的程式中也不是都放在一個機器上。

# Hello, World
## 使用 amqp.node 

在這個段落我們將會用 javascript 撰寫兩隻很簡單的程式；producer 用來傳送訊息，consumer 用來接收訊息。這裡我們會略過一些 amqp.node API 的細節，專注在最基礎的地方 - 傳遞與接收 Hello World 訊息。

下圖的 `p` 代表我們的 producer 而 `c` 則是 consumer。中間的方塊則是我們的 `queue` 一個訊息的緩衝區，RabbitMQ 會為我們的 consumer 維護這些資料。

![](https://www.rabbitmq.com/img/tutorials/python-one.png)

> amqp.node 函式庫
> RabbitMQ 支援多種協定。本教學使用 AMQP 0-9-1，這是一個通用的訊息傳遞協定。同時 RabbitMQ 也支援了許多不同語言的 client 函式庫。這裡的 amqp.node client 就是 Nodejs。
> 首先，我們使用 npm 來安裝

```bash
$ npm install amqplib
```

安裝完成之後，我們可以來開始寫些程式

## 傳送

![](https://www.rabbitmq.com/img/tutorials/sending.png)

現在我們有個傳送者叫 `send.js` ，訊息的接收者為 `receive.js`。傳送者的任務就是連線到 RabbitMQ 然後送出訊息。
在 `send.js` 中，第一步我們需要載入 `amqp` 函式庫

```js
const amqp = require('amqplib/callback_api)
```

接著連線到 RabbitMQ 的伺服器

```js
amqp.connect('amqp://localhost', function (err, conn) {})
```

然後建立頻道，大部分 API 會在頻道內完成其任務。

```js
amqp.connect('amqp://localhost', function (err, conn) {
  conn.createChannel(function (err, ch) {

  })
})
```

為了傳送訊息，我們需要建立一個 `queue` 然後把資料傳到 queue 裡面。

```js
amqp.connect('amqp://localhost', function (err, conn) {
  conn.createChannel(function (err, ch) {
    var q = 'hello'

    ch.assertQueue(q, {durable: false})
    ch.sendToQueue(q, new Buffer('Hello, World!'))
    console.log('Sent Hello, World!')
  })
})
```

宣告 queue 的動作是冪等的，意思是只會在當 queue 不存在時才會建立，不然就使用同一個。
最後，我們需要關閉連線：

```js
setTimeout(function () {
  conn.close()
  process.exit(0)
}, 500)
```

> 傳送失敗
> 如果您是第一次使用 RabbitMQ 而且上面的範例您沒有看到 `Sent...` 訊息，可能的原因包含沒有足夠的硬碟空間給 broker，預設 RabbitMQ broker 需要 200 MB 的空間。檢查 broker 的 log 檔案，OSX 的話預設在 `/usr/local/var/log/rabbitmq`。我們可以先搜尋 rabbitmq-server 的目錄尋找 `rabbit-defaults` 來查看預設的設定。

```bash
$ which rabbitmq-server
# /usr/local/sbin
$ cat /usr/local/sbin/rabbitmq-defaults
# 預設 logs : /usr/local/var/log/rabbitmq
# 預設設定檔 : /usr/local/etc/rabbitmq
```

要減少關於硬碟限制的資訊請參考[官方文件](http://www.rabbitmq.com/configure.html#config-items)，相關設定檔的位置則[參考](https://www.rabbitmq.com/relocate.html)。

# 接收訊息

我們完成了我們的傳送程式。而我們的接收者則需要從 RabbitMQ 把訊息取出。不像發送者那麼只是單純發送一道訊息。接收程式需要持續的監聽是否有訊息。

![](https://www.rabbitmq.com/img/tutorials/receiving.png)

第一步跟發送程式一樣我們需要載入函式庫

```js
let amqp = require('amqplib/callback_api')
```

一樣需要建立連線和頻道，然後宣告 queue ，我們會從這個 queue 中取出訊息。

```js
amqp.connect('amqp://localhost', function (err, conn) {
  conn.createChannel(function (err, ch) {
    let q = 'hello'
    ch.assertQueue(q, {durable: false})
  })
})
```

注意到我們也同樣在這邊建立了 queue。因為我們可能在發動`傳送訊息程式`之前就啟動`接收訊息程式`，於是我們需要確保 queue 在我們嘗試取資料前就存在。
接著我們繼續來探討如何讓伺服器從 queue 派送訊息給我們。由於推送訊息的行為是非同步的，我們需要提供一個 callback ，當 RabbitMQ 推送訊息的時候我們就可以執行。這就是 `Channel.consume` 作的事情。

```js
console.log("Wating for message in %s. To exit press CTRL+C", q)
ch.consume(q, function (msg) {
  console.log("Received %s", msg.content.toString())
}, {noAck: true})
```

# 小結

現在我們可以執行兩個 scripts 了，在終端機裡我們先執行 `send.js` 接著執行 `receive.js`。接收程式會列出收到的訊息。

> 很多時候我們可能想知道 RabbitMQ 裡面有些什麼 queue 和訊息

```bash
$ rabbitmqctl list_queues
```

# 工作佇列（Work Queues）

在上面的教學中我們撰寫了兩隻程式利用具名佇列（named queue）發送與接收訊息。在這個段落我們將要建立 `Work Queue` 常用來派發耗時的任務到多個程序去處理。 Work Queues （Task Queues）的核心想法是為了要避免耗時的任務直接接在我們的操作流程上導致我們需要等待其完成才能到下一步。取而代之的是我們把這個耗時的任務加到排程之中讓它可以在稍後完成。我們需要將任務封裝當成一個訊息，然後把它送到 queue。接著工作程序會在背後慢慢的取出任務完成。當我們執行多個工作程序時，它們會一起消化 queues 裡面的任務。這樣的概念在網路應用程式中特別實用。

# 準備

在第一部分的教學中我們送了 `Hello, World!` 的訊息。現在我們將要傳送代表複雜任務的字串。這個教學中，我們並沒有事務上常見的例子像是修改圖片尺寸或是產生 pdf，所以讓我們使用 `setTimeout` 假裝這個任務非常耗費效能與時間，將字串中的點 `.` 模擬為複製度，每一個點假裝需要一秒工作時間。例如：`Hello...` 會需要 3 秒。

接下來讓我們來稍微修改之前的 `send.js` 範例，為了讓我們使用指令發送任意的訊息，這隻程式會把我們的任務加到 work queue 中，現在我們就叫它 `new_task.js`

```js
let q = 'task_queue'
let msg = process.argv.slice(2).join(' ') || 'Hello World!'
ch.assertQueue(q, {durable: true})
ch.sendToQueue(q, new Buffer(msg), {persistent: true})
console.log('Sent %s', msg)
```

舊版的 `receive.js` 也需要稍微修改。我們需要模擬耗時任務的部分。它的任務是從 queue 取出訊息然後執行任務，我們命名該檔案為 `worker.js`。

```js
ch.consume(q, function (msg) {
  var secs = msg.content.toString().split('.').length - 1
  console.log('Received %s', msg.content.toString())
  setTimeout(function () {
    console.log('Task Done')
  }, secs * 1000)
}, noAck: true)
```

# 循環制派發的任務

使用 Task Queue 的好處之一就是可以輕鬆的擴展平行的工作程序。假設我們累積了大量的任務需要執行，我們可以增加工作程序 workers 來協助我們。
首先，我們試試同時運行 2 個 `worker.js` 它們兩者都會取得 queue 的訊息，但具體是如何呢？

現在我們有 3 個 console，兩個執行 `worker.js` 這兩個 console 讓我們先分別稱為 C1, C2。
在剩下的 console 我們要用來傳送任務

```bash
$ node ./new_task.js First maessage.
$ node ./new_task.js Second message..
$ node ./new_task.js Third message...
$ node ./new_task.js Fourth message....
$ node ./new_task.js Fifth message.....
```

接著我們來看看我們的 workers（C1，C2）

```
## C1
$ node worker.js
# Wating for message in task_queue. To exit press CTRL+C
# Received First maessage.
# First maessage. Task Done
# Received Third message...
# Received Fifth message.....
# Third message... Task Done
# Fifth message..... Task Done
```

```
## C2
$ node worker.js
# Wating for message in task_queue. To exit press CTRL+C
# Received Second message..
# Received Fourth message....
# Second message.. Task Done
# Fourth message.... Task Done
```

預設，RabbitMQ 會把訊息依序傳給下一個接收者，每個接收者會平均收到差不多訊息。這種發送訊息的方式稱為循環制。

# 確認訊息

執行任務可能會需要一點時間。您可能會想知道假如`接收程式`收到訊息並開始一個耗時的任務但最後並未完成會如何？依據我們現在的程式碼，一旦 RabbitMQ 派發了訊息，它就會立即把訊息從記憶體移除。這個情況下如果我們停止了工作程序（Worker) 我們將會遺失正在處理的訊息。

但我們並不希望在工作程序發生異常或停止時遺失任何訊息，我們希望該任務可以派發給其他工作程序。為了確保訊息不會遺失 RabbitMQ 支援了確認訊息的機制 - 一個 `ack`(nowledgement) 確認通知需要從接收程式這邊送回 RabbitMQ 告訴 RabbitMQ 該訊息已被接受和處理完成了，接著 RabbitMQ 才會釋放和刪除該訊息。
一旦接收程式因為任何原因中斷例如：RabbitMQ 頻道關閉，連線中斷，TCP 連線失敗等等導致沒有回傳 `ack`，RabbitMQ 就知道該訊息還沒處理完成，就會把它再放回 queue 裡面，假如當下還有其他工作程序在運行，那 RabbitMQ 會立刻將這個訊息派給其他人，透過這個機制我們可以確保訊息不會遺失。

RabbitMQ 不存在逾時的狀況，即使處理需要非常長的時間，假如接收程式異常或斷線， RabbitMQ 會重新發送訊息。在上面的範例中`訊息確認通知`是被關閉的，現在我們要使用 `{noAck: false}`（您也可以移除參數）和 `ch.ack(msg)` 在適當時機發送通知。`{noAck: <Boolean>}` 設定算是一個保護機制的參數，不設定時我們可以隨時使用 `ch.ack` 一旦我們設定為 `{noAck: true}` 如果又呼叫 `ch.ack` 則會產生例外。

```js
ch.consume(q, function (msg) {
  let secs = msg.content.toString().split('.').length - 1

  console.log('Received %s', msg.content.toString())
  setTimeout(function () {
    console.log('%s Done', msg.content.toString())
    ch.ack(msg)
  }, secs * 1000)
}, {noAck: false})
```

使用這段程式碼我們可以確保即使訊息在處理階段我們用 CTRL+C 中斷工作程序，訊息也不遺失。

> 忘記發送確認通知
> 忘記 `ack` 是個很常見的錯誤，但後果卻很嚴重。當目前處理的工作程序斷線時，該訊息會再次被派發，這可能導致我們重複執行任務，且當我們無法釋放 `unacked` 的訊息時，RabbitMQ 將會使用越來越多的記憶體。
> 為了 debug 這類問題，您可以使用 `rabbitmqctl` 指令來列出 `messages_unacknowledged`。

```bash
$ rabbitmqctl list_queues name messages_ready messages_unacknowledged
```

# 訊息的持久性

我們已經學會如何確保當工作程序（接收程式)異常時訊息依然存在的處理機制，但這些任務訊息在 RabbitMQ Server 停止的時候依然會消失。
當 RabbitMQ 停止或異常崩潰時會失去我們傳入的 queue 和訊息，除非我們告訴 RabbitMQ 要保留這些資訊。兩件事可以確保我們的訊息不會遺失，我們需要註記 `queue` 和 `message` 為持久性的（Durable)。

第一步，我們可以通過宣告參數告訴 RabbitMQ 不要弄丟我們的 queue。

```js
ch.assertQueue('hello', {durable: true})
```

雖然這個設定是正確的，但現在無法作用，因為我們之前已經定義了一個 queue 叫 hello ，且其沒有開啟 `durable` 特性。RabbitMQ 不允許我們建立相同名稱的 queue 卻使用不同的設定。最簡單的解決辦法就是換個名稱。

```js
ch.assertQueue('task_queue', {durable: true})
```

注意要修改這個參數，發送程式和接收程式都必須要修改。到這邊我們確保了 `task_queue` 這個 queue 不會因為 server 停止或重啟而消失，現在我們要來處理訊息的部分，透過在 `Channel.sendToQueue` 加入 `persistent` 參數可讓訊息持續存在。

> 關於訊息的持久性
> 讓訊息具備持久特性並不能完全保證訊息不會遺失，雖然我們已經告訴 RabbitMQ 要將訊息存在硬碟上，但仍可能在非常短的時間之間遺失訊息，即 RabbitMQ 已接收訊息但還沒寫入硬碟，RabbitMQ 並非對所有訊息的操作都使用 `fsync` （確保所有對文件的修改都同步到硬碟上）- 它可能只是先被存在暫存區並不是寫入硬碟。
> 也就是說訊息的持久性並不能給我們完全保證，但這對與簡單的佇列任務需求已經足夠了，如果您需要更加完整的確認機制可以使用[發佈/傳送確認機制](https://www.rabbitmq.com/confirms.html)

# 均等派發

您可能已經注意到派發的機制並不是完全如我們所預期的，舉例來說有兩個 workder，奇數訊息任務比較吃效能，偶數訊息比較輕鬆，其中一個 worker 會一直在忙碌狀態，另一個幾乎沒有任何工作。其實，RabbitMQ 並不知道哪個 worker 的任務比較吃重，只是平均的分配任務。

會這樣是因為 RabbitMQ 只是發送訊息到 queue，它沒有為了`接收方`去判斷 `unacknowledged message` 未處理訊息的數量，就只是盲目的派發訊息。

![](https://www.rabbitmq.com/img/tutorials/prefetch-count.png)

為了克服這個問題我們可以使用 `prefetch` 方法並賦值為 `1`。這會告訴 RabbitMQ 一次不要給某個 worker 超過 1 個訊息，換句話說就是在 worker 處理完任務之前不要再送新任務給它。於是 RabbitMQ 就會把任務派發給其他 worker。

```js
ch.prefetch(1)
```

> 關於 queue 的容量
> 如果所有的工作程序（workers）都在忙碌，則訊息會被放在 queue 直到 queue 被塞滿。我們可以增加更多 workers 或提供其他解決方式。


# 小結

最後我們這一節的範例程式如下

```js
// new_task.js
const amqp = require('amqplib/callback_api)

amqp.connect('amqp://localhost', function (err, conn) {
  conn.createChannel(function (err, ch) {
    let q = 'task_queue'
    let msg = process.argv.slice(2).join(' ') || 'Hello World!'

    ch.assertQueue(q, {durable: true})
    ch.sendToQueue(q, new Buffer(msg), { persistent: true })
    console.log('Sent %s', msg)

    setTimeout(function () {
      conn.close()
      process.exit(0)
    }, 500)
  })
})
```

```js
// worker.js
const amqp = require('amqplib/callback_api')

amqp.connect('amqp://localhost', function (err, conn) {
  conn.createChannel(function (err, ch) {
    let q = 'task_queue'

    ch.assertQueue(q, {durable: true})
    ch.prefetch(1)
    console.log('Waiting for messages in %s. To exit press CTRL+C', q)
    ch.consume(q, function (msg) {
      let secs = msg.content.toString().split('.').length - 1

      console.log('Received %s', msg.content.toString())
      setTimeout(function () {
        console.log('%s Done', msg.content.toString())
        ch.ack(msg)
      }, secs * 1000)
    }, {noAck: false})
  })
})
```

* `durable` 參數是訊息被存放在硬碟，即使 RabbitMQ 重啟資料也不會遺失。
* 訊息確認 `ack` 的部分確保 worker 正確處理完資料。
* `prefetch` 讓工作程序可以平均分配任務。

這幾個機制讓我們可以確保任務佇列裡的任務被完成。如果您還想知道更多關於 Channel 的方法和訊息的屬性可以參考[amqplib 文件](http://www.squaremobius.net/amqp.node/channel_api.html)。



# 參考

* [各版本 OSX 調整 kern.maxfilesperproc](https://unix.stackexchange.com/questions/108174/how-to-persist-ulimit-settings-in-osx-mavericks)
* [Open File limit](https://docs.basho.com/riak/kv/2.2.3/using/performance/open-files-limit/)
* [RabbitMQ 官方](https://www.rabbitmq.com/tutorials/tutorial-one-javascript.html)


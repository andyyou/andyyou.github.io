---
title: "防禦 DDoS - 限制 IP 時間內大量請求 Nginx + fail2ban"
date: 2023-04-12 09:05:40
tags:
  - nginx
categories: System
---


## Nginx

<!-- more -->

> 如有使用 LB 或其他 Proxy 服務，測試規則請注意來源 IP是否為同一個 IP。

```
http {

    # limit_conn_zone 限制併發連線數量
    limit_conn_zone $binary_remote_addr zone=zone1:10m;
    # 前面有檔 LB 或 Proxy IP 來源可以換成 $http_x_forwarded_for
    # limit_conn_zone $http_x_forwarded_for zone=one1:10m;


    # limit_req_zone 限制請求頻率
    # binary_remote_addr 使用客戶端 IP 進行限制
    # zone=one:10m：建立 IP 儲存區 10MB，用於儲存請求頻率資料
    # rate=10r/s：表示該 IP 每秒請求次數 10 表示 10 次
    limit_req_zone $binary_remote_addr zone=zone2:10m   rate=10r/s;

}

server {
  # ...
  location / {
    # 套用 zone1 規則，限制併發連線數量為 2
    limit_conn  zone1  2;

    # 套用 zone2 規則，限制請求頻率
    # brust 如果請求超過規則，則延遲，可被延遲的數量上限
    # nodelay 超過頻率的話請求被延遲，延遲超過數量則終止，預設返回 503
    # 若要變更狀態可以使用 limit_req_status 419: Too Many Requests
    limit_req zone=zone2 burst=10 nodelay;
    limit_req_status 419;

    # ...
   }

}
```





## fail2ban

```sh
$ sudo apt install fail2ban

# 建立設定，例如：套用 filter，掃描的 log 路徑，限制時間等
# 不要直接變更 jail.conf 因為更新會調整該檔案
$ touch /etc/fail2ban/jail.local

# 建立過濾 regex 條件
$ touch /etc/fail2ban/filter.d/http-atk.conf

# 查詢被 ban 狀態, http-atk 為設定名稱
$ sudo fail2ban-client status http-atk
```

下面設定範例為針對 Nginx 存取日誌進行限制。

### jail.local

```
[http-atk]
enabled = true
port = http,https
filter = http-atk
logpath = /var/log/nginx/access.log
maxretry = 10
findtime = 1
bantime = 20
action = iptables[name=http-atk, port=http, protocol=tcp]
```

- `[http-atk]` CLI 調用規則時的名稱，慣例都小寫。
- `enabled` 限制規則是否啟用
- `port`
- `filter` 過濾條件檔案名稱。（filter 看檔名，cli 看 conf `[]` 名稱）
- `logpath` 掃描日誌路徑
- `maxretry` 容許次數
- `findtime` 容許次數的限制時間，預設不加單位為秒。例如 `findtime` 為 1，`maxretry` 為 5。1 秒允許 5 次。
- `bantime` 鎖定時間（秒）
- `action` 使用 iptables 鎖 IP

### http-atk.conf

過濾條件檔名可自訂。

```
[Definition]
failregex = ^<HOST> - - .*\"(GET|POST).*
ignoreregex =
```

## 資源

- [Gist](https://gist.github.com/andyyou/47ba30da0e56e1ffd372c88a9ec8a11d)
- [與 DDoS 奮戰：nginx, iptables 與 fail2ban ](https://gist.github.com/andyyou/47ba30da0e56e1ffd372c88a9ec8a11d)
- [Linux 遇到 nf_conntrack: table full, dropping packet 解法](https://blog.longwin.com.tw/2018/07/linux-nf-conntrack-table-full-drop-packet-2018/)
- [nginx单个ip访问频率限制 ](https://juejin.cn/post/6844903925133344776)
- [Per-IP rate limiting with iptables - Making Pusher](https://making.pusher.com/per-ip-rate-limiting-with-iptables/index.html)
- [Fail2ban 限定登入錯誤次數](https://hackmd.io/@nikerdy/SJtDbVXQS)
---
layout: post
title: 'Docker 入門筆記 - OSX'
date: 2016-04-15 12:00:00
categories: docker, system
---

首先需要先安裝 Docker Toolbox，工具包包含下面幾種工具

* Docker Machine
* Docker Engine
* Docker Compose
* Kitematic
* command-line environment shell
* Oracle VM VirtualBox

整個流程為

* 安裝 Docker Toolbox
* 在 container 中執行 image 檔
* 在 Docker Hub 瀏覽可用的 image 檔
* 建置自己的 image 並 push 到 Docker Hub


因為 Docker 的常駐服務程式(daemon)採用 Linux 規範的核心功能，所以我們不能直接在 OSX 中執行。
因此需要使用 `docker-machine` 來建立掛載一個虛擬機器，並且用這個虛擬機器來執行 Docker。

如果 Docker 安裝在 Linux 系統中那麼 Docker client, Docker daemon, 還有任何 Docker 容器都
可以直接在本機上執行。

而在 OSX 中 Docker daemon 預設是執行在一個輕量的 Linux VM，整個 VM 被配置在記憶體中，約佔 24MB
啟動時間約 5 秒。

# 執行

1. 建立或開啟 `docker-machine`

{% highlight zsh %}
# 建立
$ docker-machine create

# 刪除
$ docker-machine rm [name]

# 重建
$ docker-machine create --driver virtualbox default

# 常用
$ docker-machine [start|stop|restart|status]

# 設定檔在 ~/.docker/machine/machines/default

# 列表
$ docker-machine ls

# 連結 shell 到 default 機器
$ eval "$(docker-machine evn default)"

＃ 啟動 docker
$ docker run hello-world

# 顯示正在執行的 docker container
$ docker ps

# 安裝之後的正常開機流程
$ docker-machine start
$ eval "$(docker-machine env default)"
{% endhighlight %}

Docker machine 運行著 Docker Engine，Engine 的部分提供我們 Docker 的核心功能。我們可以使用 image 和 container。
container 意思就像是一個 instance，而 image 檔就像是 class。

`docker run hello-world` 中 `run` 是建立和執行一個 container，最後 `hello-world` 則是 image
一個 container 等於是一個基本的 Linux 系統，而 image 則是您希望安裝到這個 container 的軟體。

# docker 常用操作

{% highlight zsh %}
# 執行 image
# docker run [image_name] [arguments]
$ docker run docker/whalesay cowsay boo

# local images 列表
$ docker images

# docker run 流程 -> 檢查本地端是否有 image -> 沒有的話到 docker hub 下載 -> 有的話沿用
#   -> 只有在有更新的時候才會重新下載

# Dockerfile 不只可以指定環境該安裝什麼軟體，還可以指定該執行哪些指令

# 建置 image，切換到 Dockerfile 目錄
$ docker build -t docker-whale .

# docker 建置流程 ->
# 1. 首先確認所有需要的資料正確
#   Sending build context to Docker daemon
# 2. 載入 image
#   FROM docker/whalesay:latest
# 3. 執行安裝與指令

# 上傳 Docker Hub
# 0. 上傳的 image 需要用帳號當作 namespace 例如: andyyou/ubuntu
# 1. 打標籤或是在 docker build -t 的時候就設好
# 2. push 到 docker hub

# 打標籤
$ docker tag 32d9cb7369aa andyyou/node:latest
$ docker tag [image id] [repo_account/image_name]

# 登入
$docker login

# 推上 docker hub
$ docker push

# 刪除 local image，可用 id 或 name
$ docker rmi -f 7d9495d03763
$ docker rmi -f docker-whale

# 刪除 container
$ docker rm -f [container_id/container_name]
{% endhighlight %}


# Docker with Node, MongoDB

下面為如何設定 Docker 容器搭配 MongoDB 與 NodeJS 的簡短介紹。我們將一步一步的介紹關於設定與過程中可能遇到的問題。
在我們熟悉 Docker 搭配 Node, MongoDB 的操作之後，我們將具備足夠基本知識來使用 Docker

### MongoDB

{% highlight zsh %}
# 下載 Mongo image
$ docker pull mongo:latest

# 啟動/執行 MongoDB container
$ docker run -v "$(pwd)":/data --name mongo -d mongo mongod --smallfiles

# -v "$(pwd)":/data 會把當前目錄對應到 /data

# 列出正在執行的 container
$ docker ps

# 列出所有的 containers
$ docker ps -a

# 移除 container
$ docker rm [container_id]

# 建立一個新的 mongo container 連接到已經開啟的 mongo container 並執行連線
# --rm 是退出之後就移除
$ docker run -it --link mongo:mongo --rm mongo sh -c 'exec mongo "$MONGO_PORT_27017_TCP_ADDR:$MONGO_PORT_27017_TCP_PORT/test"'

# 連線到已在執行的 container，先連線在執行 mongo
$ docker exec -it 67d1db81df31 bash
$ mongo
$ db.col.insert({"a": 4})
$ db.col.find().pretty()
$ mongodump --db test --out /data/test-backup # 匯出 db
$ mongorestore --db test-restored /data/test-backup/test # 還原
{% endhighlight %}

### Node

{% highlight zsh %}
# 下載 node image
$ docker pull node:latest

# 執行
$ docker run -it --rm node # -it 參數表示保持 STDIN 和配置一個 TTY, --rm 當退出 node 的時候關閉 container

# 建立一個 container 並且 mounted 到 mongo
$ docker run -it --name node -v "$(pwd)":/data --link mongo:mongo -w /data -p 8082:3000 node bash

# 建立 nodeapp container
$ docker run --name nodeapp -v "$(pwd)":/data --link mongo:mongo -w /data/app/sandbox -p 8082:3000 -d node npm start

# 顯示目前 container 的狀態
$ docker logs nodeapp
$ docker logs -f nodeapp

# 顯示連線 IP
$ docker-machine config
{% endhighlight %}

# 資源

* [Getting started](https://docs.docker.com/mac/)
* [參考](https://docs.docker.com/engine/installation/mac/)
* [設定 AWS 和 Docker Cloud](https://docs.docker.com/docker-cloud/getting-started/link-aws/#limit-dockercloud-user-to-a-specific-ec2-region)
* [Debian 8 基本安裝](http://blog.programster.org/things-to-do-after-installing-debian-8-0/)
* [Mongo with Node 教學](http://ifdattic.com/how-to-mongodb-nodejs-docker/)
* [持續更新指令](https://gist.github.com/andyyou/b88d74fc180375b9ede9)
* [錯誤處理](https://github.com/docker/toolbox/issues/346)

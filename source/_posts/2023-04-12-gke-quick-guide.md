---
title: "[譯]部署應用程式到 GKE 叢集"
date: 2023-04-12 09:09:12
tags:
  - docker
  - k8s
categories: Cloud
---

##

本篇快速入門，我們將部署一個簡單容器化的 Web 應用程式到 Google Kubernetes Engine 叢集。您將學習如何建立一個叢集以及部署應用程式到叢集。

<!-- more -->

本文旨在讓您可以快速實作感受一下 K8S 的用法。

在開始之前遵循下列步驟啟動 Kubernetes Engine API

1. 在 Google Cloud Console 介面，選擇或建立專案
2. 確認專案的帳單功能已連結
3. 啟用 Artifact Registry 和 Google Kubernetes Engine API.

> 搭配[原文](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster)的連結可以快速找到設定連結

這裡我們將使用 Cloud Shell 簡化相關前置作業。它是一個 Shell 指令環境，支援管理在 Google Cloud 上的資源服務。Cloud Shell 預先安裝了 Google Cloud CLI 和 kubectl 指令。 `gcloud` 指令主要針對 Google Cloud ，而 `kubectl` 指令則針對 Kubernetes 叢集。

1. 登入 Google Cloud Console
2. 在右上角點擊 Active Cloud Shell 按鈕
```sh
$ gcloud projects list
$ gcloud config set project [PROJECT_ID]
```

一個叢集至少由一個 Cluster Control Plane machine (叢集控制面板機器) 和多個 Node (工作者機器) 組成。Node 就是 Compute Engine 虛擬機器，其執行 Kubernetes 程序使其成為叢集的一部分。



> Node 是 K8S 中的一台實體機或 VM。Pod 是 K8S 的基本單位即容器。而容器就例如我們 Dockerized 的應用程式，資料庫服務，Redis 服務等等。這裡我們先有個觀念即可，後續操作完在閱讀相關文件您會更容易理解。
>
> ![](https://miro.medium.com/v2/resize:fit:1344/format:webp/1*vJp5o7ABILiIapesES8j6g.png)



然後我們可以部署程式到叢集，程式具體會在 Node 中執行。

```sh
$ gcloud container clusters create-auto hello-cluster \
	--region=asia-east1
```

建立叢集之後我們需要取得憑證好讓我們可以操作叢集
```sh
$ gcloud container clusters get-credentials hello-cluster --region asia-east1
```
這個指令設定好可以使用 `kubectl` 操作叢集。
現在我們已經建立好叢集，可以部署容器化的應用程式了。為了聚焦在 K8S 我們部署一個簡單的範例 `hello-app`
GKE 使用 **Kubernetes 物件**來建立和管理叢集的資源。Kubernetes 提供了 Deployment 物件來部署。Service 物件則為我們的網頁程式定義從網路存取的規則和負載平衡器。

要在叢集內執行 hello-app，我們需要利用下面指令部署應用程式
```sh
$ kubectl create deployment hello-server \
--image=us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
```

`kubectl create deployment` 建立了一個名為 hello-server 的部署物件。然後 Deployment 的 Pod 會執行 hello-app 的映象檔。在上面的指令中
- `--image` 指定部署容器的 Image。在這個例子中指令會從 Artifact Registry 檔案庫讀取指定版本的 Image。

  在部署之後，您需要公開到網路上，那麼使用者才能夠存取。您可以通過建立一個 Service 來公開我們的應用程式。
```sh
$ kubectl expose deployment hello-server --type LoadBalancer --port 80 --target-port 8080
```
`--type LoadBalancer` 參數會建立一個 Compute Engine 的負載平衡器。`--port` 參數則是公開 Port 80 到目標 8080。

建立完成之後我們就可以利用下面的指令取得 IP 和相關資訊或刪除

```sh
$ kubectl get pods

$ kubectl get services
$ kubectl get service hello-server

$ kubectl delete service hello-server
$ gcloud container clusters delete hello-cluster --region asia-east1
```
---
title: 'GCP App Engine 設定固定 IP'
date: 2024-01-18 11:10:07
tags:
  - docker
  - gcp
categories: Cloud
---

## 快速設定 App Engine IP

<!-- more -->

```sh
# 1. 建立子網路
# 內網網段遮罩一定要 28，IP 請先至 VPC 檢查，須 10.128.x.x 以內
$ gcloud compute networks subnets create [SUBNET_NAME] \
--range=10.124.0.0/28 \
--network=[VPC_NAME] \
--region=[REGION]


# 2. 建立 VPC connector
$ gcloud compute networks vpc-access connectors create [CONNECTOR_NAME] \
--region=[REGION] \
--subnet=[SUBNET_NAME]


# 3. 調整 app engine app.yaml 並更新
# vpc_access_connector:
#   name: projects/[PROJECT_ID]/locations/[REGION]/connectors/[CONNECTOR_NAME]
#   egress_setting: all-traffic
$ gcloud depoly app


# 4. (可選)建立防火牆規則 詳情請參考
# https://cloud.google.com/appengine/docs/standard/connecting-vpc#network-tags
$ gcloud compute firewall-rules create [RULE_NAME] \
--action=ALLOW \
--rules=TCP \
--source-ranges=35.199.224.0/19 \
--target-tags=vpc-connector \
--direction=INGRESS \
--network=[SUBNET_NAME] \
--priority=0 \
--project=[PROJECT_ID]


# 5. 建立子網路路由器
$ gcloud compute routers create [ROUTER_NAME] \
--network=[SUBNET_NAME] \
--region=[REGION]


# 6. 保留固定 IP
$ gcloud compute addresses create [STATIC_IP_NAME] \
--region=[REGION]


# 7. 建立子網路路由器 NAT
$ gcloud compute routers nats create [NAT_NAME] \
--router=[ROUTER_NAME] \
--region=[REGION] \
--nat-custom-subnet-ip-ranges=[SUBNET_NAME] \
--nat-external-ip-pool=[STATIC_IP_NAME]
```

### 刪除服務

```sh
$ gcloud compute routers nats delete [NAT_NAME] --router=[ROUTER_NAME] --region=[REGION]
$ gcloud compute addresses delete [STATIC_IP_NAME] --region=[REGION]
$ gcloud compute routers delete [ROUTER_NAME] --region=[REGION]
$ gcloud compute networks vpc-access connectors delete [CONNECTOR_NAME] --region=[REGION]
$ gcloud compute networks subnets delete [SUBNET_NAME] --region=[REGION]
```

## 驗證設定影響練習

練習部署 Serverless 服務，驗證是否取得固定 IP。同時檢測在同一個 VPC 下 subnets 的影響。

### 快速部署 GKE 範例

建立一個簡易的 Python 專案或使用 [Google 教學範例](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster)

新增專案目錄和建立 `app.py`

```python
# app.py
import requests
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def home():
  	# 驗證外網取得的 IP
    response = requests.get('https://api.ipify.org?format=json')
    return response.json()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

建立 Dockerfile

```dockerfile
FROM python:3.8-slim

WORKDIR /app

COPY . /app

RUN pip install Flask requests

ENTRYPOINT ["python"]
CMD ["app.py"]
```

部署（前置作業請先註冊 GCP 帳號，取得權限，安裝 [gcloud](https://cloud.google.com/sdk/docs/install)）

```sh
# 1. 建立 GCP 專案與本地設定
$ gcloud projects list
$ gcloud config set [PROJECT_ID]


# 2. 未啟用過的話，可以從網頁介面啟動 Artifact Registry API, Kubernetes Engine API 或
$ gcloud services enable artifactregistry.googleapis.com
$ gcloud services enable container.googleapis.com

# 另外注意帳號授權
$ gcloud projects add-iam-policy-binding [YOUR_PROJECT_ID] \
--member="user:[USER_EMAIL]" \
--role="roles/container.admin"


# 3. 建立叢集
# region 例如 asia-east1
$ gcloud container clusters create-auto [CLUSTER_NAME] --location=[REGION]


# 4. 測試專案建置映象檔
$ cd [DEMO_PROJECT]
$ docker build --platform=linux/amd64 -t [DEMO_PROJECT_NAME] .
$ docker tag demo-app gcr.io/[PROJECT_ID]/[DEMO_PROJECT_NAME]:latest
$ docker push gcr.io/[PROJECT_ID]/[DEMO_PROJECT_NAME]

# 取得授權
$ gcloud container clusters get-credentials [CLUSTER_NAME] --location [REGION]

# 建立 Deployment
$ kubectl create deployment [DEPLOYMENT_NAME] --image=gcr.io/[PROJECT_ID]/[DEMO_PROJECT_NAME]:latest

# 開放阜
$ kubectl expose deployment [DEPLOYMENT_NAME] --type= LoadBalancer --port 80 --target-port 80

$ kubectl get pods

$ kubectl get service [DEPLOYMENT_NAME]

$ kubectl set image deployment/[DEPLOYMENT_NAME] [DEPLOYMENT_NAME]=gcr.io/[PROJECT_ID]/[DEMO_PROJECT_NAME]:latest
```

### 快速部署 App Engine 範例

- [Google 官方教學](https://cloud.google.com/build/docs/deploying-builds/deploy-appengine)

## 常用 gcloud 指令

```sh
# gcloud 常用指令
# 列出專案
$ gcloud projects list

# 查看當前設定狀態
$ gcloud config list

# 查看當前設定專案
$ gcloud config get-value project

$ gcloud config set project [PROJECT_ID]
$ export PROJECT_ID="$(gcloud config get-value project -q)"
$ gcloud config set compute/zone [REGION] # 台灣：asia-east1


# 查詢本機全部設定集
$ gcloud config configurations list
$ gcloud config configurations create [CONFIGURATION_NAME]
$ gcloud config configurations activate [CONFIGURATION_NAME]
# 修改配置集
$ gcloud config set project [PROJECT_ID]
$ gcloud config set compute/zone [ZONE]
$ gcloud config set account [ACCOUNT]

# 服務列表
$ gcloud services list --available
# 啟用服務
$ gcloud services enable [SERVICE_NAME]
```

## 參考資源

- [Deploy an app to a GKE cluster](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster)
- [Outbound IP addresses for App Engine services](https://cloud.google.com/appengine/docs/standard/outbound-ip-addresses)
- [Connecting to a VPC network](https://cloud.google.com/appengine/docs/standard/connecting-vpc#network-tags)
- [簡介 VPC 概念、架構與模式設定](https://mile.cloud/zh/resources/blog/vpc-introduction-network-setting-subnet_533)

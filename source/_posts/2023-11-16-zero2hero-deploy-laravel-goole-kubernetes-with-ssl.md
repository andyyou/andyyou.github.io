---
title: "Zero2Hero：部署 Laravel 10 到 GKE"
date: 2023-11-16 09:14:35
tags:
  - k8s
  - laravel
categories: Cloud
---


本文僅針對 macOS 環境，全文目標為部署一個 Laravel 10 專案至 GKE。

- 使用 GKE 運行 Laravel 應用程式
- 使用 Google ManagedCertificate 自動管理 SSL 憑證
- 使用 Cloudfare 作為反向代理並使用 SSL 完整或嚴格模式


<!-- more -->

## 安裝設定工具

開始之前，您需要建立一組 Google Cloud 的帳號，建立專案，並啟用相關 API 權限。

- [參考 Github Action 前段教學](https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-google-kubernetes-engine)

### 安裝 Gloud SDK （`gcloud`）- Google Cloud 管理指令介面

請先檢查系統環境，須支援 python 3.8 - 3.11。下面使用 `pyenv` 安裝 python 3.11.4。
若 macOS 系統版本符合您可以直接略過本章節。

```sh
$ brew install pyenv
$ echo 'eval "$(pyenv init -)"' >> ~/.zshrc
$ source ~/.zshrc
$ pyenv install 3.11.4
$ pyenv global 3.11.4
$ pyenv rehash

$ python -V
```

### 安裝 Google Cloud CLI

因為部署的目標是 GKE 因此須先安裝 **Google Cloud CLI 352.0+** 請依照[教學](https://cloud.google.com/sdk/docs/install)下載對應系統的檔案並設定。

常用指令如下：

```sh
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
```

### 安裝 Kubernetes 客戶端指令工具 （`kubectl`）

使用 `gcloud`  指令安裝 Kubernetes 指令工具 `kubectl`：

```sh
$ gcloud components install kubectl

# 更新
$ gcloud components update
```

- 其他資源 [Kubernetes 官方文件](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/)

## 建立 Kubernetes 叢集

```sh
$ gcloud container clusters create-auto [CLUSTER_NAME] \
	--region=[REGION]
# 台灣：asia-east1

# 查看叢集
$ gcloud container clusters describe [CLUSTER_NAME] --region=[REGION]
```

在建立叢集的時候有 `create-auto` 和 `create` 可以使用，`create-auto` 建立的是 **Autopilot** 是一種全自動化的叢集管理模式。`create` 建立的是標準模式叢集，主要差異為標準模式您可以擁有更多控制權。

接著，我們需要驗證叢集的權限，使用下面指令：

```sh
$ gcloud container clusters get-credentials [CLUSTER_NAME] --location [REGION]
```

這個指令會提供 `kubectl` 所需的相關設定。

檢查 `kubectl` 設定：

```sh
# 取得 kubectl 設定列表
$ kubectl config get-contexts

# 刪除 kubectl 設定
$ kubectl config delete-context [CONTEXT_NAME]

# ⚠️ 若 `gcloud` 切換專案或切換操作叢集請記得更新憑證
$ gcloud config set project [PROJECT_ID]
$ gcloud container clusters get-credentials [CLUSTER_NAME] --location [REGION]
$ kubectl config get-contexts
```

## 配置固定IP

預留一個適用於 Ingress 的**全域靜態IP**。

> 使用 Google Kubernetes Engine (GKE) 的 Ingress 資源時，通常需要使用全域靜態 IP 地址。這是因為 Ingress 作為一個 HTTP(S) 負載平衡器，通常需要能夠從任何地方接收到來的網路流量。全域靜態 IP 地址確保了這一點。

```sh
$ gcloud compute addresses create [NAME] --global
```

查詢剛建立的IP

```sh
$ gcloud compute addresses describe [NAME] --global --format="get(address)"
```


## 建立 GKE Namespace

Namespace 用於分隔應用，簡單的應用其實可以省略，但我們會使用 Secrets / ConfigMap，如果有多個相同類型的 app 可能導致 .env 混亂，因此需要 Namespace。

假設您有多個 Laravel 應用，每個應用都有自己的 `.env` 文件。在這種情況下，您可以為每個 Laravel 應用創建一個單獨的 Namespace。然後在每個 Namespace 中創建一個專用的 Secrets，來存儲該應用的配置。

建立 `namespace.yaml`：

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: [NAMESPACE]
```

執行

```sh
$ kubectl apply -f namespace.yaml
```


## 使用 .env 建立 GKE Secret

在 GKE Secret 中建立環境變數，如此 Docker 建置的時候就可以略過處理 `.env`

```sh
$ kubectl create secret generic [SECRET_NAME] --from-
env-file [ENV_FILENAME] -n [NAMESPACE]
```

`[SECRET_NAME]` 名稱後續在 _Deployment_ 的 `envFrom` 使用。


## Laravel 應用容器化

### Dockerfile

建立 `Dockerfile` for Laravel 10.x

```dockerfile
FROM php:8.2.12-fpm

WORKDIR /var/www/[PROJECT_FOLDER]
COPY . /var/www/[PROJECT_FOLDER]

COPY ./www.conf /usr/local/etc/php-fpm.d/www.conf
COPY ./php-fpm.conf /usr/local/etc/php-fpm.conf

RUN apt-get update \
    && apt-get install -yqq \
        g++ \
        libjpeg-dev \
        libpng-dev \
        libicu-dev \
        ssh-client \
        unzip \
        git \
        nginx \
    && docker-php-ext-install \
        gd \
        opcache \
        pdo \
        pdo_mysql \
        intl

# redis 安裝 & php.ini 要開啟
RUN pecl install redis
RUN cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini
RUN echo "extension=redis.so" > /usr/local/etc/php/php.ini

# PHP 的 Opcache，功用是將 PHP 的程式碼編譯成機器碼，加速 PHP 的執行速度
# opcache.memory_consumption=128 代表 Opcache 的記憶體大小為 128MB，值取決於服務器可用內存和應用的大小
# opcache.interned_strings_buffer=8 代表 Opcache 的字串快取大小為 8MB (8MB - 16MB)
# opcache.max_accelerated_files=4000 設定 Opcache 可以緩存的最大腳本數量。這個數值應該根據您的應用中腳本的 2 倍 (4000-10000)
# opcache.revalidate_freq=2 設定 Opcache 檢查腳本更新的頻率（單位是秒）
# opcache.fast_shutdown=1 啟用快速關閉，這可以提高請求終止時的性能
# opcache.enable_cli=1 允許在命令行界面（CLI）模式下啟用 Opcache。這對於 CLI 腳本也能受益於 Opcache
RUN { \
    echo 'opcache.memory_consumption=128'; \
    echo 'opcache.interned_strings_buffer=8'; \
    echo 'opcache.max_accelerated_files=4000'; \
    echo 'opcache.revalidate_freq=2'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'opcache.enable_cli=1'; \
} > /usr/local/etc/php/conf.d/php-opocache-cfg.ini

COPY ./nginx-config /etc/nginx/sites-enabled/default

COPY ./entrypoint.sh /etc/entrypoint.sh

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer install --no-dev

RUN usermod -u 1000 www-data
COPY --chown=www-data:www-data . /var/www/[PROJECT_FOLDER]

EXPOSE 80 443

ENTRYPOINT ["sh", "/etc/entrypoint.sh"]
```

### 本地測試

部署之前您可以選擇在本地先進行測試：

> ⚠️ 注意您的 Dockerfile 須複製處理 `.env`，因為上面我們將 `.env` 丟進 GKE Secret 中，因此在本地測試時須先處理好。

```sh
# 建置映像檔
$ docker build -t [IMAGE_NAME] .

# ⚠️ 容易導致 Deployment 失敗的原因。
# ⚠️ macOS ARM 架構的機器要建置推上 Artifact Registry 檔案庫的請使用：
$ docker build --platform=linux/amd64 -t [IMAGE_NAME] .

# 背景執行容器並對應本機的 8080 port 至容器 80 port
$ docker run -d -p 8080:80 [IMAGE_NAME]

# 查詢運行的容器
$ docker ps

# 查看容器的日誌
$ docker logs [CONTAINER_ID]

# 停止
$ docker stop [CONTAINER_ID]

# 刪除
$ docker rm [CONTAINER_ID]

# 刪除映像檔
$ docker rmi [IMAGE_NAME]
```


### 本地建置 GKE 映像檔並使用 Artifact Registry

- [官方教學建立 Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/store-docker-container-images)

前置作業：

1. 啟動 Artifact Registry API
2. 安裝 gcloud
3. 安裝 docker


開始執行：

```sh
# 建立 Repository
$ gcloud artifacts repositories create [REPOSITORY_NAME] \
    --repository-format=docker \
    --location=asia-east1 \
    --project=[PROJECT_ID]
    
# 查詢建立的 Repository
$ gcloud artifacts repositories list

# 幫 docker 指令驗證，請對應您使用的 Region
$ gcloud auth configure-docker asia-east1-docker.pkg.dev

# ✅ 使用本地已經建置好的映像檔
# 使用 Docker 標記您的本地映像，然後將其推送到剛建立的存儲庫
# 每次重新建置映像檔都須執行 tag 和 push
$ docker tag [IMAGE_NAME] asia-east1-docker.pkg.dev/[PROJECT_ID]/[REPOSITORY_NAME]/[IMAGE_NAME]:latest
$ docker push asia-east1-docker.pkg.dev/[PROJECT_ID]/[REPOSITORY_NAME]/[IMAGE_NAME]:latest

# ✅ 或者直接建置到對應的名稱
# ⚠️ 容易導致 Deployment 失敗的原因。macOS ARM 架構的機器要建置推上 Artifact Registry 檔案庫的請使用：--platform=linux/amd64
$ docker build --platform=linux/amd64 -t asia-east1-docker.pkg.dev/[PROJECT_ID]/[REPOSITORY_NAME]/[IMAGE_NAME]:latest .
$ docker push asia-east1-docker.pkg.dev/[PROJECT_ID]/[REPOSITORY_NAME]/[IMAGE_NAME]:latest
```

## Kubernetes 部署

### 設定 Deployment

Kubernetes 本質就是利用這些 YAML 檔案來設定配置相關服務。接下來我們設定 _Deployment_ 就是這裡會使用到我們前面準備的映像檔和環境變數，
_Deployment_ 負責定義和控制如何執行應用程式，當我們建立一個 _Deployment_ 它會建立並管理 **Pod** 並讓這些 **Pod** 運行我們的 Laravel 應用程式。

> ⚠️ Deployment 的設定會需要之前建立的 Namespace， Secret，Docker Image URL 請確認您已經建立。

建立 `deployment.yaml`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: [DEPLOYMENT_NAME]
  namespace: [NAMESPACE_NAME]
spec:
  replicas: 1 # 您希望運行的 Pod 副本數量
  selector:
    matchLabels:
      app: [APP_NAME]
  template:
    metadata:
      labels:
        app: [APP_NAME]
    spec:
      containers:
        - name: [APP_NAME]
          image: asia-east1-docker.pkg.dev/[PROJECT_ID]/[REPOSITORY_NAME]/[IMAGE_NAME]:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          envFrom:
            - secretRef:
                name: [SECRET_NAME]
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m" # 500m = 0.5 cpu, 1000m = 1 cpu
            limits:
              memory: "2Gi"
              cpu: "2000m"
```

執行

```sh
$ kubectl apply -f deployment.yaml
```

### (Debug) 部署 Deployment 失敗

```sh
# 查詢指定命名空間中的所有 Pod
kubectl get pods -n [NAMESPACE]

# 描述指定命名空間中的特定 Pod，提供詳細信息
# 替換 [POD_NAME] 為實際的 Pod 名稱
kubectl describe pod [POD_NAME] -n [NAMESPACE]

# 查看指定 Pod 的日誌，幫助理解 Pod 內部發生了什麼
# 替換 [POD_NAME] 為實際的 Pod 名稱
kubectl logs [POD_NAME] -n [NAMESPACE]

# 查詢指定命名空間中的 Deployment 狀態
kubectl get deployments -n [NAMESPACE]

# 描述指定命名空間中的特定 Deployment，提供詳細信息
# 替換 [DEPLOYMENT_NAME] 為實際的 Deployment 名稱
kubectl describe deployment [DEPLOYMENT_NAME] -n [NAMESPACE]

# 如果需要對某個節點進行調試，可以使用以下命令
# 查詢集群中所有節點的狀態
kubectl get nodes

# 描述特定節點，提供詳細信息
# 替換 [NODE_NAME] 為實際的節點名稱
kubectl describe node [NODE_NAME]
```

> ⚠️「exec format error」通常與架構不匹配有關。大概率為使用 macOS ARM 架構建置 x86 環境使用的 Docker Image。

- [部署 Deployment 失敗 Troubleshooting](https://cloud.google.com/kubernetes-engine/docs/troubleshooting?authuser=1&_ga=2.202801297.-1440217981.1681817572&_gac=1.222213354.1699942892.Cj0KCQiAr8eqBhD3ARIsAIe-buOTskmblbi8D4zCd2_uogPNIQh7g2pL7MgxwcuGXHMOcnvVit7waiYaAijEEALw_wcB#does_not_have_minimum_availability)


### 設定 Service

_Service_ 則是提供一個接口，存取運行在 **Pod** 中的應用。常見的類型包含 `ClusterIP`，`NodePort`， `LoadBalancer`

Service 設定參數的重點：

1. **Service Selector**：`Service.spec.selector` 定義了一組標籤，這些標籤用於 Service 查找並連接到一組 Pod。
2. **Pod 標籤**：Pod 的標籤通常在其所屬的 Deployment 的 Pod 模板中定義，位於`Deployment.spec.template.metadata.labels`。
3. **匹配標籤**：為了讓 Service 正確地將流量轉發到特定的 Pod，Service 的 `selector` 標籤必須與 Pod 的標籤相匹配。
4. **Deployment Selector**：`Deployment.spec.selector.matchLabels` 是用來確保 Deployment 正確管理（創建和更新）其 Pod。這個選擇器應與 Pod 模板中定義的標籤相匹配。

也就是說**設定 Service 時要注意 Deployment 對應的標籤設定。**

建立 `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: [SERVICE_NAME]
  namespace: [NAMESPACE]
spec:
  # 重要設定，決定 Service 如何將流量轉發到對應的 Pod
  selector:
    app: [APP_NAME]
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

執行

```sh
$ kubectl apply -f service.yaml
```


### SSL 憑證託管

建立 `managed-certificate.yaml`

```yaml
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: [CERT_NAME]
  namespace: [NAMESPACE_NAME]
spec:
  domains:
    - [YOUR_DOMAIN]
```

執行

```sh
$ kubectl apply -f managed-certificate.yaml

# 取得狀態
$ kubectl get managedcertificate -n [NAMESPACE]
```

除了指令外，也可以到 GCP 的 Certificate Manager 介面傳統版憑證查詢。

> ⚠️ SSL 憑證必須要有 DNS 解析，並驗證成功 Active。
> ⚠️ 因為後續我們會使用 Cloudflare，為了讓憑證生效，注意 Cloudflare 得先使用 DNS Only。

### 設定 Ingress 

#### 建立 SSL Policy

到 GCP > 網路 > SSL 政策 > 建立 Policy > Global SSL policy 。


#### 建立 Ingress

建立 `ingress.yaml`，由於 _BackendConfig_ 和 _FrontendConfig_ 關聯性比較緊密，因此彙整在同一個檔案。

```sh
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: [INGRESS_NAME]
  namespace: [NAMESPACE]
  annotations:
    # 有另外設定一個固定靜態 IP 要給 upsace-wallet，這邊要填幫那個 IP 取得名字
    # 可以在 GCP cloud shell 上用 gcloud compute addresses list 查看剛剛建立的 IP 名稱
    kubernetes.io/ingress.global-static-ip-name: [IP_NAME]
    # 使用 Google 自動憑證 要跟上面 ManagedCertificate 的 name 一樣
    networking.gke.io/managed-certificates: [CERT_NAME]
    # Ingress 類別預設使用 Google Cloud 的 Ingress 控制器，即 gce。
    # 其他還有 nginx, haproxy, traefik
    kubernetes.io/ingress.class: 'gce'
    # 跟下面 FrontendConfig 的 name 一樣
    networking.gke.io/v1beta1.FrontendConfig: [FRONTEND_CONFIG_NAME]
spec:
  rules:
    - host: [YOUR_DOMAIN]
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                # K8S service 的名字
                name: [SERVICE_NAME]
                port:
                  number: 80
---
apiVersion: cloud.google.com/v1
kind: BackendConfig
metadata:
  name: [BACKEND_CONFIG_NAME]
  namespace: [NAMESPACE]
spec:
  connectionDraining:
    drainingTimeoutSec: 60
  # 在 Cloudflare 層面獲得了 CDN 的優化，這邊就不需要了
  # cdn:
  #   enabled: true
  #   cachePolicy:
  #     includeHost: true
  #     includeProtocol: true
  #     includeQueryString: true

---
apiVersion: networking.gke.io/v1beta1
kind: FrontendConfig
metadata:
  name: [FRONTEND_CONFIG_NAME]
  namespace: [NAMESPACE]
spec:
  sslPolicy: [POLICY_NAME]
  redirectToHttps:
    enabled: true
```

執行

```sh
$ kubectl apply -f ingress.yaml

# 查詢預約的靜態IP
$ gcloud compute addresses list
```


### (Debug) Service, Ingress 無法連線

```sh
# 查詢指定命名空間中的所有 Ingress 資源
$ kubectl get ingress -n [NAMESPACE]

# 描述指定命名空間中的特定 Ingress，提供詳細信息
# 替換 [INGRESS_NAME] 為實際的 Ingress 名稱
$ kubectl describe ingress [INGRESS_NAME] -n [NAMESPACE]

# 查詢指定命名空間中的所有 Service 資源
$ kubectl get svc -n [NAMESPACE]

# 描述指定命名空間中的特定 Service，提供詳細信息
# 替換 [SERVICE_NAME] 為實際的 Service 名稱
$ kubectl describe svc [SERVICE_NAME] -n [NAMESPACE]

# 如果有 Managed SSL 證書，檢查其狀態
$ kubectl describe managedcertificate [CERTIFICATE_NAME] -n [NAMESPACE]

# 如果懷疑是 DNS 問題，檢查域名是否正確解析到 Ingress 的 IP 地址
# 這可能需要使用外部工具，如 `dig` 或 `nslookup`
```


## 連線 Pod

```sh
# 查詢 Pod 名稱
$ kubectl get pods -n [NAMESPACE]

# 連線 Pod
$ kubectl exec -it [POD_NAME] -n [NAMESPACE] -- /bin/sh
```
---
title: "2023 部署 Next.js 至 Google Cloud Run (混搭 Cloud Build, Gitlab CI, Secret Manager)"
date: 2023-01-12 18:11:53
tags:
  - docker
  - gitlab
  - gcp
categories: Cloud
---


本文的核心為將 Next.js 部署至 Google Cloud Run，CI 的部分會使用 Gitlab CI，主要會依據採用不同服務分成兩種流程：

1. 在 Gitlab CI 建置 Docker Image 暫存到 Gitlab 後續部署到 Cloud Run
2. Gitlab CI 搭配 Cloud Build 部署到 Cloud Run

> 本文希望可以儘量補充其他教學遺失的環節，幫助您可以自行選擇想用的服務調整流程。

[toc]

<!-- more -->

## 使用 Gitlab CI 建置 Docker Image 暫存到 Gitlab 後續部署至 Cloud Run

### 1. 建立 Next.js 專案

```sh
$ npx create-next-app [PROJECT_NAME]

# 測試專案
$ cd [PROJECT_NAME]
$ npm run dev
```

### 2. Docker 化 Next.js 專案

在 Next.js 專案根目錄下

```sh
$ touch Dockerfile
```

> 下面 `Dockerfile` 是參考[官方範例](https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile)修改而成的。隨著時間演進可能有所不同，建議您可以也可以一起參考官方的範例調整。
>
> 另外，為了減少正式環境建置檔案大小複製了 `/app/.next/standalone` 則 `next.config.js` 須增加設定 `output: "standalone"`
>
> ```ts
> /** @type {import('next').NextConfig} */
> const nextConfig = {
>   reactStrictMode: true,
>   output: "standalone",
> };
>
> module.exports = nextConfig;
> ```



`Dockerfile` 如下，採用複數 Stage / Layer 的理由是為了縮小最後 Image 的大小：

```dockerfile
# 安裝相依套件
FROM node:16-alpine AS deps
# 查看 https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine
# 理解為何需要 libc6-compat
RUN apk add --no-cache libc6-compat
WORKDIR /app

# 根據偏好的套件管理工具安裝套件，您也可以使用 yarn
COPY package.json package-lock.json ./
RUN npm ci --omit=dev

# 僅在需要的時候重新編譯原始碼
FROM node:16-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js 收集一般使用資料 - Telemetry
# 查看更多資料: https://nextjs.org/telemetry
# 如果您需要關閉資料收集可以解開註解
ENV NEXT_TELEMETRY_DISABLED 1

RUN npm run build

# 正式環境 Image, 複製全部檔案並執行 Next
FROM node:16-alpine AS runner
WORKDIR /app

ENV NODE_ENV production
# 如果您需要關閉資料收集可以解開註解
ENV NEXT_TELEMETRY_DISABLED 1

# 您可以變更成您希望的群組和使用者名稱
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# 自動利用輸出軌跡來減小圖像大小
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```



完成 `Dockerfile` 之後可以執行 `docker build` 和 `docker run` 進行本地測試

```sh
$ docker build -t andyyou/nextjs .
$ docker run -p 3000:3000 andyyou/nextjs
```

現在您可以瀏覽 `http://localhost:3000` 它會顯示跟之前本地開發伺服器一樣的內容。
到此我們已經成功的 Docker 化一個 Next.js 應用程式。另外建議您可以閱讀 [Nodejs 與 Docker](https://blog.logrocket.com/node-js-docker-improve-dx/)。



### 3. 設定 Google Cloud Service

這個 CI 流程大部分我們都在 Gitlab 處理，會用到 GCP 的部分只有 Cloud Run，但在發佈的過程我們會需要一組 Service Account 的金鑰。因此本節的設定：

1. 建立 GCP 專案
2. [啟用 Cloud Run API](https://cloud.google.com/run/docs/setup)
3. 建立授權一組 Service Account 並取得 JSON Key

![](https://i.imgur.com/FRBfDnT.png)

![](https://i.imgur.com/08XlCB2.png)

![](https://i.imgur.com/1cRbJC1.png)

![](https://i.imgur.com/43LYFKG.png)

![](https://i.imgur.com/bbGJeNU.png)

![](https://i.imgur.com/BiJZYq0.png)

![](https://i.imgur.com/Ytu49Bh.png)

![](https://i.imgur.com/RjZpKtj.png)

![](https://i.imgur.com/fnz4KRO.png)

![](https://i.imgur.com/HQgFq5K.png)

![](https://i.imgur.com/NT75oky.png)

> GCP 權限部分：
>
> - Editor - 允許 Service Account 啟用/停用 API 或從 Google Cloud SDK 安裝更新
> - Cloud Run Admin - 允許 Service Account 執行 Cloud Run 指令，例如 `deploy` 應用程式
> - Storage Admin - 允許 Service Account  推送 `push` Docker Image 到 Google Container Registry
> - Service Account User - 允許 Service Account 在 Cloud Run 中使用 Service Account 部署
>
> 備註：關於權限的部分可以依據您的需求調整，此處僅為展示。



### 4. 建立 `.gitlab-ci.yml`

在開始撰寫 `.gitlab-ci.yml` 之前，**請先建立好我們的 Gitlab Repository** ，我們還需要將 Cloud Run 的服務名稱，Google Project ID，Service Account 金鑰設到 Gitlab ，如此 CI 才能讀取相關資料。

**登入 Gitlab > 專案 > 左邊選單 Settings > CI/CD > Variables** 新增變數。下面變數您應該可以在 GCP Console 以及剛剛下載的 JSON 取得。`GCP_CLOUD_RUN_SERVICE_NAME` 您可以自訂就是這個專案服務的名稱。

> 如果要在其他分支使用變數記得取消 `Protect variable`。

- GCP_SERVICE_KEY

- GCP_PROJECT_ID

- GCP_CLOUD_RUN_SERVICE_NAME



接著，我們就可以使用 Gitlab CI 將 Next.js 專案部署到 Cloud Run 了

```sh
$ touch .gitlab-ci.yml
```



`.gitlab-ci.yml` 檔案如下：

```yml
image: docker

services:
  - docker:dind

variables:
  DOCKER_HOST: tcp://docker:2375
  DOCKER_DRIVER: overlay2
  CI_IMAGE: $CI_REGISTRY_IMAGE/$GCP_CLOUD_RUN_SERVICE_NAME:$CI_COMMIT_SHORT_SHA
  DEPLOY_IMAGE: gcr.io/$GCP_PROJECT_ID/$GCP_CLOUD_RUN_SERVICE_NAME:$CI_COMMIT_SHORT_SHA

stages:
  - build
  - deploy

build:
  stage: build
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_IMAGE .
    - docker push $CI_IMAGE

deploy:
  stage: deploy
  image: google/cloud-sdk:slim
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker pull $CI_IMAGE
    - docker tag $CI_IMAGE $DEPLOY_IMAGE
    - echo $GCP_SERVICE_KEY > key.json
    - gcloud auth activate-service-account --key-file key.json
    - gcloud config set project $GCP_PROJECT_ID
    - gcloud auth configure-docker
    - docker push $DEPLOY_IMAGE
    - gcloud run deploy $GCP_CLOUD_RUN_SERVICE_NAME --image $DEPLOY_IMAGE --project $GCP_PROJECT_ID --platform managed --region asia-east1 --allow-unauthenticated
  when: manual
  only:
    - main

```

然後您就可以到 Gitlab > CI/CD > Pipeline 去發佈。



### 5. 補充 - 環境變數處理

假設您沒有把 `.env` 相關的檔案 commit 到 Git 檔案庫上。上面的範例遺漏了一個沒有處理的部分，Next.js 在建置時期會無法取得環境變數，導致前端部分無法使用，即便您後來在 Cloud Run 介面補上這些變數也無法讀取。



這裡提供一個簡單的方式即先將變數以 **VAR1=a,VAR2=b,VAR3=c** 的格式儲存到 Gitlab 的 CI/CD 環境變數。然後利用 `--build-arg` 參數將變數傳入 Docker 建置流程，在稍微變更一下 `Dockerfile` 來處理。下面為兩個設定檔案



這裡示範補上一個 `NEXT_ENV` 環境變數。



`Dockerfile` 的部分注意 `RUN npm run build` 之前補上 `ARG` 等

```dockerfile
// ...

FROM node:16-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1

ARG NEXT_ENV
RUN echo $NEXT_ENV | tr "," "\n" > .env.production
RUN npm run build

// ...
```



`.gitlab-ci.yml` 則在 `build` 階段補上 `--build-arg`

```yml
build:
  stage: build
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_IMAGE . --build-arg "NEXT_ENV=$NEXT_ENV"
    - docker push $CI_IMAGE
```



## 進階 - 使用 Gitlab CI 搭配 Google Cloud Build 部署至 Cloud Run

開始前概略介紹

- Cloud Run 是一個託管計算平台，提供自動擴展無狀態，無伺服器的容器
- Cloud Build 是一個服務，可以在 Google Cloud Platform 執行編譯建置的任務
- Gitlab CI 是 Gitlab 服務的一部分，可以在開發者將程式碼推送到檔案庫時建置和測試軟體



本節和上一節的差異為這次我們不在 Gitlab 編譯 Docker Image 而是將任務直接交給 Cloud Build 後續的部署也是一樣都由 Cloud Build 處理。以及我們會使用 Secret Manager 來處理正式環境的環境變數。



### 1. 設定 Google Cloud Service

- 如果您還沒有 GCP 專案請先建立專案，或參考上面章節。
- [啟用 Cloud Build](https://cloud.google.com/build/docs/set-up)
- [啟用 Cloud Run API](https://cloud.google.com/run/docs/setup)
- [啟用 Secret Manager](https://cloud.google.com/secret-manager/docs/configuring-secret-manager)
- 登入您的 GCP 切換至 Cloud Build > Settings，在 **Service account permissions** 下面確認 **Cloud Run & Service Accounts & Secret Manager Secret Accessor** 為 **ENABLED**
- 參考上面建立 Service Accout 的步驟不過這次我們需要的權限是 `Cloud Build Service Agent`和 `Secret Manager Secret Accessor` ，一樣取得 JSON 檔案。您也可以直接至 Google IAM 去調整。
- 登入 GCP > Secret Manager > Create Secret > 使用 `VAR1=a,VAR2=b,VAR3=c` 格式建立您的環境變數。

![](https://i.imgur.com/JklRgmT.png)

![](https://i.imgur.com/OYDzSfq.png)



### 2. 建立或使用既有 Next.js 專案

您可以參考上面章節建立專案或使用既有的專案。除此之外我們還需要



### 3. 設定 Gitlab CI/CD, Cloud Build 與 Docker

從上概覽，整個流程的順序會是我們靠 `.gitlab-ci.yml` 使用 `gcloud builds submit` 將 `cloudbuild.yaml` 腳本和專案本身一併交給 Cloud Build。然後 Cloud Build 依據 `cloudbuild.yaml`

- 讀取 Secret Manager 的資料
- 建置 Docker Image
- 推送 Docker Image 到 Registry
- 使用 `gcloud` 指令部署到 Cloud Run



第一步我們先搞定 Gitlab CI，一樣到 Gitlab 專案下左邊選單 **Settings > CI/CD > Variables** 設定環境變數

- GCP_SERVICE_KEY
- GCP_PROJECT_ID

```sh
$ touch .gitlab-ci.yml
```



`.gitlab-ci.yml` 檔案如下

```yml
image: google/cloud-sdk:slim

stages:
  - deploy

deploy:
  stage: deploy
  script:
    - echo $GCP_SERVICE_KEY > key.json
    - gcloud auth activate-service-account --key-file key.json
    - gcloud config set project $GCP_PROJECT_ID
    - gcloud builds submit . --config=cloudbuild.yaml
  when: manual
  only:
    - main
```

注意到這裡我們使用 Google 的 Image 支援 `gcloud` 相關指令。如果您希望在 Gitlab 這邊也增加 `test` 和 `build` 階段可以自行增加。我們這裡單純演示使用 Cloud Build 的部分。



### 4. 建立 `cloudbuild.yaml`

接著我們新增 `cloudbuild.yaml` 檔案如下。注意 Secret Manager 的使用。您可以閱讀一下[官方文件](https://cloud.google.com/build/docs/securing-builds/use-secrets)，有個重點 `-c` 後面的指令是不可以拆開的。

```yaml
steps:
  - name: "gcr.io/cloud-builders/docker"
    entrypoint: "bash"
    args:
      [
        "-c",
        "docker build -t gcr.io/$PROJECT_ID/nextjs . --build-arg NEXT_ENV=$$NEXT_ENV",
      ]
    secretEnv: ["NEXT_ENV"]
  - name: "gcr.io/cloud-builders/docker"
    args: ["push", "gcr.io/$PROJECT_ID/nextjs"]
  - name: "gcr.io/cloud-builders/gcloud"
    args:
      [
        "run",
        "deploy",
        "nextjs",
        "--image",
        "gcr.io/$PROJECT_ID/nextjs",
        "--platform",
        "managed",
        "--region",
        "asia-east1",
        "--allow-unauthenticated",
      ]
availableSecrets:
  secretManager:
    - versionName: projects/$PROJECT_ID/secrets/NEXT_ENV/versions/1
      env: "NEXT_ENV"


```



### 5. 調整 Dockerfile

```dockerfile
# 安裝相依套件
FROM node:16-alpine AS deps
# 查看 https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine
# 理解為何需要 libc6-compat
RUN apk add --no-cache libc6-compat
WORKDIR /app

# 根據偏好的套件管理工具安裝套件，您也可以使用 yarn
COPY package.json package-lock.json ./
RUN npm ci --omit=dev

# 僅在需要的時候重新編譯原始碼
FROM node:16-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js 收集一般使用資料 - Telemetry
# 查看更多資料: https://nextjs.org/telemetry
# 如果您需要關閉資料收集可以解開註解
ENV NEXT_TELEMETRY_DISABLED 1

ARG NEXT_ENV
RUN echo $NEXT_ENV | tr "," "\n" > .env.production
RUN npm run build

# 正式環境 Image, 複製全部檔案並執行 Next
FROM node:16-alpine AS runner
WORKDIR /app

ENV NODE_ENV production
# 如果您需要關閉資料收集可以解開註解
ENV NEXT_TELEMETRY_DISABLED 1

# 您可以變更成您希望的群組和使用者名稱
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# 自動利用輸出軌跡來減小圖像大小
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

到此推送到 Gitlab 之後就可以部署了。本文的使用範例都是經過測試，希望可以提供一些幫助。


另外，兩個方式都是使用 `--build-arg` 的方式傳遞，如果您專精 Docker 有更好的作法歡迎告訴我 XD。

## 參考資源


- [Deploy to Cloud Run using GitLab CI](https://medium.com/google-cloud/deploy-to-cloud-run-using-gitlab-ci-e056685b8eeb)
- [Build repositories from GitLab](https://cloud.google.com/build/docs/automating-builds/gitlab/build-repos-from-gitlab)
- [This Is How I Deploy Next.js into Google Cloud Run with Github Actions](https://medium.com/weekly-webtips/this-is-how-i-deploy-next-js-into-google-cloud-run-with-github-actions-1d7d2de9d203)
- [Deploying to Cloud Run using Cloud Build](https://cloud.google.com/build/docs/deploying-builds/deploy-cloud-run)
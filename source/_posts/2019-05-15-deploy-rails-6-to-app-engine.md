---
title: Deploy Rails 6 to Google App Engine
date: 2019-05-15 15:06:10
tags:
  - gcp
  - ruby
  - rails
categories: Cloud
---
* [官方 Rails 5 參考](https://cloud.google.com/ruby/rails/using-cloudsql-postgres)

<!--more-->

### 事前準備

* 於 Google Cloud Platform 主控台建立專案。

* 為專案啟用計費功能，如果您尚未啟用，請立即[啟用計費功能](https://console.cloud.google.com/projectselector2/settings)

* [安裝並初始化 Google Cloud SDK](https://cloud.google.com/sdk/docs/)

* 設定本機 Rails 環境，本文假設您對於 Rails 的部分已有足夠的理解，故不在此增加冗余的介紹



### 啟用 Cloud SQL Admin API

[立即啟用](https://console.cloud.google.com/flows/enableapi?apiid=sqladmin.googleapis.com)或使用指令介面：

```bash
# 切換專案
$ gcloud projects list
$ gcloud config set project <project_id>
# 查詢服務
$ gcloud services list --available
# 啟用
$ gcloud services enable sqladmin.googleapis.com
```



### 建立 Cloud SQL Instance 和 Database

* 建立 [PostgreSQL 執行個體](https://cloud.google.com/sql/docs/postgres/create-instance?authuser=0)
* [在執行個體中建立資料庫](https://cloud.google.com/sql/docs/postgres/create-manage-databases?authuser=0)



### 安裝 gem

```bash
$ bundle add pg # 如果尚未安裝
$ bundle add appengine
```



### 設定資料庫連線

```bash
# 取得 Cloud SQL 資料庫資訊
$ gcloud sql instances describe <instance_name>
```

```yml
# config/database.yml
production:
  adapter: postgresql
  encoding: unicode
  pool: 5
  timeout: 5000
  username: "[YOUR_POSTGRES_USERNAME]"
  password: "[YOUR_POSTGRES_PASSWORD]"
  database: [your_database_name_production]
  host:   "/cloudsql/[YOUR_INSTANCE_CONNECTION_NAME]"
```

> 您也可以一併在 `app.yaml` 中加入 `env_variables`。



### Active Storage 設定

```yml
# config/storage.yml
google:
  service: GCS
  project: [YOUR_PROJECT_ID]
  credentials: <%= ENV['GOOGLE_APPLICATION_CREDENTIALS'] %>
  bucket: [YOUR_BUCKET_NAME]
```

如果您的圖片權限異常 `SignedUrlUnavailable` 可以取得 `credential.json` 再試試。

* 控制台 > *APIs & Services* > *Credentials* > *Create credentials* >  *Service account key* > *App engine default service account* > *JSON* > *Create*。

```yml
# config/storage.yml
google:
  service: GCS
  project: website
  credentials: <%= Rails.root.join("your/path.json") %>
  bucket: [YOUR_BUCKET_NAME]
```




### 設定 app.yaml 部署至 App Engine

App Engine 彈性環境使用 `app.yaml` 描述部署環境設定，在 Rails 專案根目錄加入 `app.yaml`

```yaml
entrypoint: bundle exec rackup --port $PORT
env: flex
runtime: ruby

# Rails 5.2+ 之後使用 config/master.key
env_variables:
  RAILS_MASTER_KEY: [MASTER_KEY]

beta_settings:
  cloud_sql_instances: [YOUR_INSTANCE_CONNECTION_NAME]
```

設定完成後執行下列指令：

```bash
$ gcloud app create
$ RAILS_ENV=production bundle exec rails assets:precompile
$ gcloud app deploy

# 取得專案資訊
$ gcloud info

# 瀏覽
$ gcloud app browse
# OR
$ gcloud app browse -s <service_name>

# 讀取日誌
$ gcloud app logs read
$ gcloud app logs tail
$ gcloud app logs tail -s <service_name>
```



### 授予 *appengine* RubyGem 權限，執行 Migration

```bash
$ gcloud projects list
$ gcloud projects add-iam-policy-binding [YOUR-PROJECT-ID] \
  --member=serviceAccount:[PROJECT_NUMBER]@cloudbuild.gserviceaccount.com \
  --role=roles/editor]

$ bundle exec rake appengine:exec -- bundle exec rake db:migrate
```

### 其他資源與參考

```bash
# 取得用於驗證 GCP 服務的本機憑證。
$ gcloud auth application-default login

# Stackdriver
# Add this to config/environments/*.rb
Rails.application.configure do |config|
  # Stackdriver Logging specific parameters
  config.google_cloud.logging.project_id = "YOUR-PROJECT-ID"
  config.google_cloud.logging.keyfile    = "/path/to/service-account.json"
end
```

- [Ruby on Rails GCP](https://guides.rubyonrails.org/active_storage_overview.html#google-cloud-storage-service)
- [Fix Google::Cloud::Storage::SignedUrlUnavailable](https://stackoverflow.com/questions/50549176/google-cloud-storage-500-internal-server-error-googlecloudstoragesignedur/50572991)
- [外部連線資料庫](https://cloud.google.com/sql/docs/postgres/external-connection-methods)
- [使用 IP 位址連結 psql 用戶端](https://cloud.google.com/sql/docs/postgres/connect-admin-ip#configure-instance-mysql)
- [Setting up Rails 5.2 Active Storage, using Google Cloud Storage and Heroku](https://medium.com/@pjbelo/setting-up-rails-5-2-active-storage-using-google-cloud-storage-and-heroku-23df91e830f8)



---
title: 手把手實戰部署 ReactJS 至 Amazon S3
date: 2019-08-22 15:36:18
categories: Cloud
tags:
  - react
  - aws
---


> 本文嘗試盡可能實作最佳實踐，但部分設定請依據自身需求調整。

## 大綱

* 建立一個簡易的 React 應用程式
* 設定 S3 提供靜態網站託管
* 部署
* 進階實戰 - SSL 搭配自訂網域
* 進階實戰 - S3
* 進階實戰 - CloudFront
* 進階實戰 - 自動化腳本

<!-- more -->

基本上前半段和後半段有蠻多重複的，如果您已經有些 AWS 的使用經驗，可直接跳至進階實戰。

## 準備

* AWS 帳號
* AWS IAM User
* 安裝 AWS CLI
* 設定 AWS 憑證


#### 設定 AWS 帳號

[註冊](https://portal.aws.amazon.com/billing/signup)或登入[AWS Console](https://signin.aws.amazon.com/signin)，點擊【服務/Services】搜尋【IAM】。

{% asset_img 1.png %}

在【IAM】介面點擊左側的【使用者/Users】，我們需要為 *Serverless Framework* 建立一組使用帳號。這組帳號會授權我們的框架建立、更新、刪除 AWS 上的資源。

{% asset_img 2.png %}

點擊【新增使用者/Add User】，輸入使用者名稱和 *程式設計方式存取/Programmatic access* 。下一步。

{% asset_img 3.png %}

至【設定許可】點擊【直接連結現有政策/Attach existing policies directly】選取【AmazonS3FullAccess】。後續直接沿用預設值，建立使用者。授予的許可應該盡遵循最小權限原則，如果您只是針對測試需求可選取【AdministratorAccess】

{% asset_img 4.png %}



取得【存取金鑰 ID/Access Key Id】和【私密存取金鑰/Secret Access Key】，後續我們會需要這兩個資料，即我們上面提到的 AWS 憑證。

{% asset_img 5.png %}



開啟您的終端機程式：

```bash
# 安裝 AWS CLI
$ brew install awscli

# 設定預設的 AWS Credential
$ aws configure
AWS Access Key ID [None]: YOUR KEY
AWS Secret Access Key [None]: YOUR KEY
Default region name [None]: us-west-2
Default output format [None]: ENTER

# 新增其他 Profile 名稱憑證
# 後續部分指令，如果您使用多組 Profile 請記得補上 --profile
$ aws configure --profile serverless

```

憑證預設會存放在 `~/.aws` 目錄下。



#### 安裝 NodeJS

```bash
# 您可以使用一般 node 或 nvm 這裡僅概列指令
$ brew install node

# 或
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
$ nvm install node
```



## 建立簡易的專案

```bash
$ npx create-react-app [YOUR_PROJECT_NAME] # Replace [YOUR_PROJECT_NAME] with `react-s3-demo`
$ cd react-s3-demo
$ npm start
```

## 設定 S3 Bucket

切換至 S3 介面
{% asset_img 6.png %}
點擊【建立儲存貯體】
{% asset_img 7.png %}


注意：儲存佇體名稱（Bucket name）必須要全域唯一。
名稱與區域
{% asset_img 8.png %}
設定選項，直接使用預設值
{% asset_img 9.png %}
設定許可，取消所有封鎖
{% asset_img 10.png %}
檢閱，點擊建立儲存佇體

回到列表頁面，選取剛剛建立的儲存佇體。點擊【屬性/Properties】頁籤。您應該可以看到【靜態網站託管/Static Website Hosting】點擊之後，選擇【使用此儲存貯體來託管網站/Use this bucket to host a website】。輸入 `index.html`。以 React 應用程式來說您的錯誤頁面也會是 `index.html`。
{% asset_img 11.png %}
設定靜態網站託管
{% asset_img 12.png %}
現在，儲存佇體已經可以託管靜態網站，我們還需要設定許可權限。點擊【許可/Permissions】接著 【儲存貯體政策/Bucket Policy】貼上下面的設定（取代您的 Bucket Name）：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicReadAccess",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME/*"
      ]
    }
  ]
}
```

{% asset_img 13.png %}

到此步驟關於 AWS S3 的設定已經完成了。

## 建置與部署

開啟我們一開始建立的 React 專案

```bash
$ npm run build
$ aws s3 sync build/ s3://react-s3-demo --acl public-read
```

您也可以將指令加入 `package.json` 。

## 進階實戰 - SSL 搭配自訂網域

注意：藉由 Amazon Certificate Manager 建立的憑證若要給 CloudFront 使用。您可以使用儲存位於美國東部
(維吉尼亞北部) 區域的 AWS Certificate Manager (ACM) 中的憑證，或者您可以使用儲存在 IAM 中的憑證。
步驟 1
{% asset_img ssl-1.png %}
步驟 2
{% asset_img ssl-2.png %}
步驟 3
{% asset_img ssl-3.png %}
步驟 4
{% asset_img ssl-4.png %}
步驟 5
{% asset_img ssl-5.png %}
步驟 6 請至您的網域 DNS 管理介面加入 CNAME 紀錄，通過驗證。
{% asset_img ssl-6.png %}

## 進階實戰 - S3
步驟 1 靜態網站搭配 Amazon CloudFront 時，我們可以限制儲存體的存取僅讓 CloudFront 存取即可。
{% asset_img s1.png %}
步驟 2
{% asset_img s2.png %}

步驟 3 保留預設值
由於使用了 Cloud Front 這個步驟可以封鎖項目。
{% asset_img s3-1.png %}

```bash
# 上傳檔案到 S3（移除 --acl）
$ aws s3 sync build/ s3://react-s3-demo
```

> 注意：封鎖公開存取之後，無法依據上面設定【設定 S3 Bucket】段落的公開的政策，託管的網址會無法取得檔案，另外 `aws s3 sync` 的 `--acl` 也會被拒絕。詳細的權限設定請參考下面參考資源。如果想要支援 `--acl public-read`，可以開啟兩個選項即可。

{% asset_img s3-2.png %}


步驟 4
{% asset_img s4.png %}


## 進階實戰 - CloudFront

建立分佈設定：

```bash
源網域名稱/Origin Domain Name:
  您建立的 S3 Bucket
源 ID/Origin ID:
  保留預設值或可自行設定名稱
限制儲存貯體存取/Restrict Bucket Access:
  是，如此網站必須通過 CDN 存取
源存取身份/Origin Access Identity:
  自行設定依據狀況新增或選擇既有
授與對儲存貯體的讀取許可/Grant Read Permissions on Bucket:
  是，更新儲存貯體政策
檢視器通訊協定政策/Viewer Protocol Policy:
  重新導向 HTTP 到 HTTPS
允許的 HTTP 方法/Allowed HTTP Methods:
  GET、HEAD、OPTIONS、PUT、POST、PATCH、DELETE
自動壓縮物件/Compress Objects Automatically:
  是
備用網域名稱/Alternate Domain Names(CNAMEs):
  您的網域
SSL 憑證/SSL Certificate:
  自訂 SSL 憑證
```

步驟 1
{% asset_img cdn-1.png %}
步驟 2
{% asset_img cdn-2.png %}
步驟 3
{% asset_img cdn-3.png %}
步驟 4
{% asset_img cdn-5-2.png %}
步驟 5
{% asset_img cdn-6.png %}
步驟 6
{% asset_img cdn-7.png %}
步驟 7
{% asset_img cdn-8.png %}
步驟 8
{% asset_img cdn-9.png %}

步驟 9 設定好之後檢查一下 S3 是否有自動更新政策。

> 如果您需要使用 AWS 提供的靜態網站託管網址或有其他原因，那麼記得設定【靜態網站託管】。否則不須設定。

{% asset_img cdn-10.png %}
步驟 10 到 CloudFront 查看網域名稱
{% asset_img cdn-11.png %}
步驟 11 最後到您的網域 DNS 介面補上 CNAME 設定
{% asset_img cdn-12.png %}
等待約 10 - 20 分鐘即生效。
{% asset_img cdn-13.png %}
針對 React 應用程式的錯誤，CloudFront 記得要導向 index.html
{% asset_img cdn-14.png %}
瀏覽
{% asset_img cdn-15.png %}

## 進階 - 自動化腳本

下面為 NodeJS 的自動化腳本，您需要安裝 AWS CLI。
> ⚠️警告：本腳步**請勿直接複製使用**，您應該理解並一步步調整成您的狀況。

```js
const util = require('util');
const exec = util.promisify(require('child_process').exec);
const fs = require('fs');

const S3_BUCKET_NAME = '';
// CloudFront ID 用來取得 CloudFront 當前設定。請從 Cloud Front 介面查詢
const CLOUD_FRONT_ID = '';
// JSON 或 YAML 暫存目錄
const TEMP_FOLDER = '.aws';
// CloudFront 當前設定檔案
const CONFIG_PATH = `${TEMP_FOLDER}/cloudfront.json`;
// 更新後設定檔路徑
const PATCHED_CONFIG_PATH = `${TEMP_FOLDER}/patched-dist-config.json`;
// 部署檔案目錄，請替換成您編譯結果的目錄
const DEPLOY_FOLDER = 'build/';
// 更新 CloudFront 和 S3 預設物件路徑和錯誤頁面路徑
const INDEX_FILENAME_PATTERN = /index.*html/;
const ERROR_FILENAME_PATTERN = /500.*html/;

async function main() {
  // 請逐個 function 實驗檢查
  createTempFolder();
  // 由於使用 CloudFront 快取，為了確保每次更新時檔案為最新的
  // 您有兩種選擇：1. 使用 Invalidate 不過依次計費。 2. 檔案名稱每次編譯後加入 Hash 或時間戳記
  // 此步驟為更新相關 index.{timestamp}.html and error-{hash}.js 設定
  // (可選) 為了更新 CloudFront 設定
  await getCloudFrontDistributionConfig();
  const { indexFilename, eTag } = modifyCloudFrontDistributionConfig();

  // 上傳檔案到 S3
  await deployFilesToS3();

  // (可選) 為了更新 S3 設定
  await updateS3StaticWebsiteHostingConfig(indexFilename);
  // (可選) 為了更新 CloudFront 設定
  await updateCloudFrontDistributionConfig(eTag);

  // 移除暫存檔
  await removeTempFolder();
}
main();

// actions
function createTempFolder() {
  console.log('Creating temp folder...');
  if (!fs.existsSync(TEMP_FOLDER)) {
    fs.mkdirSync(TEMP_FOLDER);
  }
}

async function getCloudFrontDistributionConfig() {
  // 如果您使用的是預設的 AWS 憑證請移除 --profile
  console.log('Downloading cloudfront distribution config...');
  await exec(`aws cloudfront get-distribution-config --id ${CLOUD_FRONT_ID} > ${CONFIG_PATH} --profile [IF_YOU_USE_DEFAUTL_PLEASE_REMOVE]`);
}

function updateDistConfig(obj, newIndexFilename, newErrorFilename) {
  Object.keys(obj).forEach((key) => {
    if (obj[key] !== null && typeof obj[key] === 'object') {
      updateDistConfig(obj[key], newIndexFilename, newErrorFilename);
      return;
    }
    if (typeof obj[key] === 'string') {
      switch (key) {
        case 'DefaultRootObject':
          obj[key] = newIndexFilename;
          break;
        case 'ResponsePagePath':
          if (obj[key].match(INDEX_FILENAME_PATTERN)) {
            obj[key] = `/${newIndexFilename}`;
          }
          if (obj[key].match(ERROR_FILENAME_PATTERN)) {
            obj[key] = `/${newErrorFilename}`;
          }
          break;
      }
    }
  });
};

function modifyCloudFrontDistributionConfig() {
  console.log('Modify cloudfront distribution config...');
  let indexFilename = '';
  let errorFilename = '';
  const files = fs.readdirSync(DEPLOY_FOLDER);
  files.forEach(file => {
    if (file.match(INDEX_FILENAME_PATTERN)) {
      indexFilename = file;
    }

    if (file.match(ERROR_FILENAME_PATTERN)) {
      errorFilename = file;
    }
  });
  // 讀取並編輯 Distribution Config
  const { DistributionConfig, ETag } = JSON.parse(fs.readFileSync(CONFIG_PATH, 'utf8'));
  updateDistConfig(DistributionConfig, indexFilename, errorFilename);
  fs.writeFileSync(PATCHED_CONFIG_PATH, JSON.stringify(DistributionConfig, null, 2), 'utf8');
  return {
    indexFilename: indexFilename,
    eTag: ETag,
  };
}

async function deployFilesToS3(bucketName) {
  console.log('Deploy files to AWS S3...');
  await exec(`aws s3 rm s3://${S3_BUCKET_NAME} --recursive --profile seefu`);
  await exec(`aws s3 sync public/ s3://${S3_BUCKET_NAME} --acl public-read --profile [IF_YOU_USE_DEFAUTL_PLEASE_REMOVE]`);
}

async function updateS3StaticWebsiteHostingConfig(indexFilename) {
  console.log('Updating AWS S3 static website hosting config...');
  await exec(`aws s3 website s3://app.uxtesting.io --index-document ${indexFilename} --error-document ${indexFilename} --profile [IF_YOU_USE_DEFAUTL_PLEASE_REMOVE]`);
};

async function updateCloudFrontDistributionConfig() {
  console.log('Updating cloudfront distribution config...');
  await exec(`aws cloudfront update-distribution --id ${CLOUD_FRONT_ID} --distribution-config file://${PATCHED_CONFIG_PATH} --if-match ${eTag} --profile [IF_YOU_USE_DEFAUTL_PLEASE_REMOVE]`);
}

async function removeTempFolder()  {
  if (fs.existsSync(TEMP_FOLDER)) {
    console.log('Removing temp folder...');
    if (fs.existsSync(CONFIG_PATH)) {
      fs.unlinkSync(CONFIG_PATH);
    }

    if (fs.existsSync(PATCHED_CONFIG_PATH)) {
      fs.unlinkSync(PATCHED_CONFIG_PATH);
    }

    fs.rmdirSync(TEMP_FOLDER);
  }
};
```



## 參考資源

* [Deploy ReactJS App with S3 Static Hosting](https://medium.com/serverlessguru/deploy-reactjs-app-with-s3-static-hosting-f640cb49d7e6)
* [IAM Policies and Bucket Policies and ACLs! Oh, My! (Controlling Access to S3 Resources)](https://aws.amazon.com/tw/blogs/security/iam-policies-and-bucket-policies-and-acls-oh-my-controlling-access-to-s3-resources/)
* [Backup to S3 by CLI](https://aws.amazon.com/tw/getting-started/tutorials/backup-to-s3-cli/)

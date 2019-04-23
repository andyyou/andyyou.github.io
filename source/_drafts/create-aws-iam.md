---
title: AWS IAM 筆記
date: 2016-10-14 11:42:15
categories: Cloud
tags: [aws]
---


### IAM 是什麼？

AWS Identity and Access Management (IAM) 屬於一個網路服務協助您的使用者安全的控制存取 AWS 資源權限。
您可以透過 IAM 控制誰能夠使用您的資源，透過什麼方式使用。

IAM 資源/組成

* User
* Role
* Group
* Identity Provider
* Policy

IAM 賦予我們下列的功能：

##### 委派帳號存取權限給其他使用者

您可以授權給其他人管理您 AWS 帳號的資源而不需要把帳密或存取金鑰交給對方。

##### 細分權限 - 存取部分功能

您可以授予不同的權限與資源給不同的使用者。舉例來說您也許會允許某些使用者可以完整存取 Amazon Elatic Compute Cloud（Amazon EC2），Amazon Simple Storage Service（Amazon S3），Amazon DynamoDB，Amazon Redshift，和其他 AWS 服務。另外其他人您可以讓他們只有讀取 S3 buckets 的權限、只有管理 EC2 執行個體的權限，又或者只能讀取帳單資訊。

##### 賦予 Amazon EC2 中的應用程式權限

您可以使用 IAM 的功能安全的給予在執行個體上運行的應用程式權限，讓它們可以存取其他資源像是 S3，RDS 或 DynamoDB。

##### 成員認證

您可以授權在其他地方已經有密碼的使用者 - 舉例來說您的企業網路或內部網域整合識別服務取得暫時的權限以存取您的 AWS 帳號。

##### 確認身份資訊

如果您使用 [AWS CloudTrail](https://aws.amazon.com/tw/cloudtrail/)，您會收到日誌訊息，其資訊包含誰請求您帳號的服務資源，這些資訊都會根據 IAM 身份取得。

##### 支援 PCI DSS（支付卡產業資料安全標準）

IAM 支援商家，服務提供者處理，儲存，傳送信用卡資訊並且整個流程通過 PCI DSS 的驗證。更多關於 PCI DSS 的資訊請參閱[PCI DSS Level 1](http://aws.amazon.com/compliance/pci-dss-level-1-faqs/)

##### 和其他 AWS 服務整合

關於 AWS 服務列表與 IAM 整合的相關資訊，請查閱 [AWS Services That Work with IAM](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html)

---
title: 使用 Zoom API (1)
date: 2021-09-29 17:52:40
tags:
  - zoom
categories: Program
---

Zoom API 允許開發者從 Zoom 請求平台的資訊，包含但不限於使用者詳細資料，會議報告，儀表板資料，以及執行平台的一些功能。例如建立新的使用者或刪除會議紀錄。

<!-- more -->

## 驗證

每一個到 Zoom API 的 HTTP 請求必須經過 Zoom 的驗證。Zoom 支援下列驗證方式：

* OAuth 2.0
* JWT

## 使用 OAuth 2.0

OAuth 2.0 讓應用程式可以利用 Zoom API 獲取 Zoom 的資源例如使用者個人資料。下面提供關於 OAuth 協定的概覽。要開始使用 OAuth 首先必須在 Zoom App Marketplace 為您的應用程式[建立 OAuth App](https://marketplace.zoom.us/docs/guides/build/oauth-app)。

### OAuth 角色

OAuth 協定定義了 4 個特定角色。這些角色參與了 Zoom API 驗證的流程：

* 資源擁有者 - Zoom 的使用者，他們可以授權或拒絕**客戶端**存取使用者帳戶的資源
* 資源伺服器 - 提供資源的伺服器。如果您的應用程式整合了 Zoom API 以獲取使用者相關資料，Zoom API 伺服器即資源伺服器。
* 客戶端 - 發出使用者資料請求的應用程式。如果您的應用程式向 Zoom API 發出請求，那麼您的應用程式即客戶端。
* Zoom 驗證伺服器 - 驗證伺服器發出存取金鑰讓驗證成功的客戶端授權取得資源擁有者的資源

一般來說，客戶端，Zoom 使用者，Zoom 驗證伺服器，Zoom API 之前的互動關係如下圖：

![](https://s3.amazonaws.com/user-content.stoplight.io/16138/1570826762485)

1. 客戶端請求 Zoom 使用者授權存取使用者資訊。
2. 使用者授權該應用程式，該應用程式收到授予授權。
3. 應用程式向 Zoom 驗證伺服器提供授予授權資訊驗證是否收到使用者的授權進而允許存取使用者資料。
4. Zoom 驗證伺服器驗證使用者已經授權該應用程式，並回傳存取金鑰和更新金鑰給應用程式。在調用 Zoom API 時必須使用存取金鑰（Access Token）
5. 應用程式對 ZoomAPI 發出包含存取金鑰的請求。金鑰過期會無效，在這種情況下必須使用更新金鑰請求另一個有效的金鑰。
6. Zoom API 檢查金鑰無誤之後，使用 JSON 格式回傳請求的資源。如果驗證失敗，而回傳錯誤訊息。

## OAuth 授權類型

OAuth 授權是由資源擁有者發給客戶端的。授權類型指的是客戶端使用的授權驗證方式。

OAuth 2.0 支援了多種授權的方式。使用 Zoom API 您可以使用 Authorization Code 或 Client Credentials

### Authorization Code

Authorization Code 是 Zoom API 最常使用的授權方式。使用方式在[OAuth with Zoom](https://marketplace.zoom.us/docs/guides/authorization/oauth/oauth-with-zoom)有詳細描述

下面對於 Authorization Code 授權流程提供一個步驟的說明

1. 客戶端將使用者導到 Zoom 驗證伺服器。使用者會看到授權請求和 scope 的相關說明。這個流程由 Zoom 所提供。當使用者安裝或重新安裝您的應用程式時，使用者會被導向 Zoom API 的驗證連結。如果要在本地使用您的應用程式對其測試，可以使用測試連結。導向的頁面大概如下圖所示：
   ![](https://s3.amazonaws.com/user-content.stoplight.io/16138/1570827314139)


2. 使用者點擊授權


3. 使用者會到導向應用程式的 `redirect_url`同時在 Query String 包含 Authorization Code，導向的連結組成大概如下
`https://yourappsredirecturl/?code={theauthorizationcode}`


4. 客戶端使用 Authorization Code 發送請求到 Zoom 要求 Access Token

```js
var request = request('request');

var options = {
  method: 'POST',
  url: 'https://zoom.us/oauth/token',
  qs: {
  grant_type: 'authorization_code',
  // 下面是 Authorization Code 的範例，您應使用您取得的 Code
  code: 'B1234558uQ',
  // 下面是 redirect_uri 範例
  redirect_uri: 'https://abcd.example.com',
  },
  headers: {
  // 下面的憑證是 base64 encoded 的。使用下面的程式碼範例產生您的
  // "Authorization: 'Basic ' + Buffer.from(your_app_client_id + ':' + your_app_client_secret).toString('base64')"
  Authorization: 'Basic abcdesdkjfesjfg',
  },
};

request(options, function(error, response, body) {
  if (error) {
  throw new Error(error);
  }
  console.log(body);
})
```

### Client Credentials

Client Credentials 授權方式取得的 Token 只能用在服務層級。意思是無法取得使用者的權限，只針對應用程式。

例如可以使用 Client Credentials 取得聊天機器人服務的 Token 進而使用 Chatbot Messages API。

要使用 Client Credentials，步驟如下：

1. 到 [Zoom App Marketplace](https://marketplace.zoom.us/) 的管理介面
2. 在 **App Credentials** 複製 **Client ID** 和 **Client Secret**
3. 發送 `POST` 請求到下面的驗證網址 `https://zoom.us/oauth/token?grant_type=client_credentials`
   

```js
var request = require('request');

var options = {
  method: 'POST',
  url: 'https://zoom.us/oauth/token?grant_type=client_credentials',
  headers: {
    // "Authorization: 'Basic ' + Buffer.from(your_app_client_id + ':' + your_app_client_secret).toString('base64')"
    authorization: 'Basic abcdsdkjfesjfg'
  }
};

request(options, function(error, response, body) {
 if (error) throw new Error(error);

 console.log(body);
});
```

在您收到 Access Token 之後就可以使用 Chatbot Messages API 了。

## 使用 JWT

JSON Web Token 讓我們可以使用一個 JSON 物件來建立一個安全傳輸的 Token。JWT 包含簽章的資料協助建立伺服器對伺服器的驗證。

如果只有您會您的 Zoom 帳戶會在應用程式中使用，那麼建議您可以使用 JWT 驗證。要完成這個功能，需要先在 Zoom App Marketplace 註冊 [JWT App](https://marketplace.zoom.us/docs/guides/getting-started/app-types/create-jwt-app)。使用該 JWT App 產生的 Token 來發送 API 請求即可。

更多詳細介紹可以參考 [JWT with Zoom](https://marketplace.zoom.us/docs/guides/authorization/jwt)



## API 請求

所有 API 請求必須使用 HTTPS 。`https://api.zoom.us/v2/` 是 API 的網址，完整的端點路徑會基於不同的資源而不同。

例如，要取得使用者詳細資料則是 `GET https://api.zoom.us/v2/users/{userId}` 。如果您的應用程式是已經在 Zoom App Marketplace 註冊帳戶層級的 OAuth 應用程式，您的應用程式必須取得 `user:read:admin` scope 才能使用這個 User API



## JWT 應用程式請求

JWT 應用程式不需要 scope。您的 JWT 應用程式一般只能存取您自己的帳戶資訊。要檢視特定使用者的資料，您必須要提供 `userId` 或電子郵件作為 `{userId}` 。也可以使用 `me` 這個關鍵字取代 `userId` 值。

```js
var request = require("request");

var options = {
	method: 'GET',
	url: 'https://api.zoom.us/v2/users/sjkf1234',
	headers: {
		authorization: 'Bearer {yourtokenhere}' // Do not publish or share your token publicly.
	}
};

request(options, function (error, response, body) {
	if (error) throw new Error(error);

	console.log(body);
});
```

## OAuth 應用程式請求

使用者層級的 OAuth 應用程式要取得使用者資料， 應用程式必須要有 `user:read` scope。雖然端點路徑一樣，但 `userId` 值的行為和帳戶層級應用程式不同。不是使用 `userId` 或電子郵件，而是必須得使用 `me` 這個關鍵字作為 `userId` 的值，否則會收到無效 Token的錯誤。

```js
var request = require('request');

var options = {
  method: 'POST',
  url: 'https://api.zoom.us/v2/users/me',
  headers: {
    authorization: 'Bearer {yourtokenhere}'
  }
};

request(options, function (error, response, body) {
	if (error) throw new Error(error);

	console.log(body);
});
```



### `me` 關鍵字

您可以任何有 `userId` 的 API 使用 `me` 關鍵字。當您使用 `me` 關鍵字時， API 會使用通過驗證使用者的 Access Token。

例如要使用 API 更新通過驗證使用者的設定，您應該使用 `/users/me/settings` 而不是 `/users/{userId}/settings`。

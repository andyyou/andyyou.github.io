---
title: 'TinyMCE 6 處理圖片上傳, 整合 Laravel, React, TypeScript'
date: 2022-06-03 17:11:51
tags:
  - laravel
categories: Program
---

> 若您只想參考如何快速整合 Laravel + TypeScript + TinyMCE 範例可直接至最下方該節。

TinyMCE 使用 **Image Uploader** 上傳編輯圖片。這讓 TinyMCE 支援了圖片編輯的功能。通過其他方式加入的本地圖片也使用這個功能上傳。例如使用 [paste_data_images](https://www.tiny.cloud/docs/plugins/opensource/paste/#paste_data_images)設定搭配拖拉圖片，或使用 [PowerPaste 套件](https://www.tiny.cloud/docs/plugins/premium/powerpaste/)，TinyMCE 會自動更新 `<img>` 的 `src` 屬性。

<!-- more -->

使用 `editor.uploadImages()` 可以上傳本地圖片。這個功能支援讓使用者在所有圖片上傳完成之前可以儲存內容。一旦這種情況發生且遠端圖片路徑還沒好，則圖片會存成 Base64 格式。

> 注意: 在送出編輯器內容之前，執行 `editor.uploadImages()` 以避免圖片儲存成 Base64 格式。可利用 Success Callback 在全部圖片上傳完成再執行。這個 Success Callback 可以通過 POST 儲存編輯器的內容。

## 使用 `uploadImages` 然後提交表單

```js
tinymce.activeEditor.uploadImages().then(() => {
  document.forms[0].submit();
});
```

## 使用 `uploadImages` 搭配 jQuery

```js
tinymce.activeEditor.uploadImages().then(() => {
  $.post('ajax/post.php', tinymce.activeEditor.getContent()).done(() => {
    console.log('Uploaded images and posted content as an ajax request.');
  });
});
```

> 注意: TinyMCE 不支援 SVG 是為了保護使用者，因為 SVG 可能包含一些可執行的攻擊。

## Image Uploader 需求

一個伺服器端上傳處理(Upload Handler)負責上傳圖片到遠端伺服器。程式必須:

- 接受圖片
- 合適的儲存圖片
- 回傳 JSON 物件包含圖片上傳的路徑

這裡提供一個[ PHP 上傳處理的實作範例](https://www.tiny.cloud/docs/advanced/php-upload-handler/)。

圖片透過 HTTP POST 傳送到 Image Uploader 每張圖片發送一個 POST。而利用 `images_upload_url` 設定網址對應的 Image Handler 圖片處理必將圖片存在應用程式。例如:

- 儲存在 Web 伺服器的目錄
- 儲存在 CDN 伺服器
- 儲存在資料庫
- 儲存在資源檔案管理系統

圖片上傳時建議在 POST 使用標準化的名稱 `blobid0`, `blobid1`, `imagetools0`, `imagetools1`

> 注意: 確保上傳處理程式為每一張圖產生唯一的名稱。一個常見的方式就是把當前的時間加到檔名尾巴，時間粒度到毫秒。例如 `blobid0-1458428901092.png` 或 `blobid0-1460405299-0114.png`。

> 警告: 如果檔名重複，檔案會被複寫。

伺服器端的上傳處理必須要回傳一個 JSON 物件包含 `location` 屬性。這個屬性表示上傳圖片的遠端路徑和檔名。

## 圖片上傳參數

`images_upload_url` 或 `images_upload_handler` 參數可以設定上傳圖片的功能。其他參數則是可選的。

必須:

- [images_upload_url](https://www.tiny.cloud/docs/general-configuration-guide/upload-images/#images_upload_url)
- 或 [images_upload_handler](https://www.tiny.cloud/docs/general-configuration-guide/upload-images/#images_upload_handler)

其他可選參數

- [images_upload_base_path](https://www.tiny.cloud/docs/general-configuration-guide/upload-images/#images_upload_base_path)
- [images_upload_credentials](https://www.tiny.cloud/docs/general-configuration-guide/upload-images/#images_upload_credentials)
- [images_reuse_filename](https://www.tiny.cloud/docs/general-configuration-guide/upload-images/#images_reuse_filename)

## images_upload_url

該參數讓您設定一個 URL 指定伺服器端的上傳處理函式。每當您調用 `editor.uploadImages()` 就會觸發上傳，或自動上傳如果 `automatic_uploads` 選項有提供也會觸發。上傳處理函式應回傳一個新的路徑如下格式

```json
{
  "location": "folder/sub-folder/new-location.png"
}
```

請務必查閱伺服器端實作[範例](https://www.tiny.cloud/docs/advanced/php-upload-handler/)

```js
tinymce.init({
  selector: 'textarea',
  images_upload_url: 'postAcceptor.php',
});
```

## images_upload_handler

`images_upload_handler` 參數可以設定一個 function 用以取代 TinyMCE 預設的 JavaScript 函式，您可以自訂邏輯。

該函式包含 2 個參數

- `blobInfo`
- `progress` 回呼函式接收 `value` 1 到 100

並且需回傳一個 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 其需要 `resolve` 上傳完成的圖片 URL 或 `reject` 錯誤訊息。錯誤訊息可以是一個字串或物件包含:

- `message` 顯示的錯誤訊息
- `remove` 是否從 document 移除圖片，預設為 `false`

當參數沒設定時，TinyMCE 會使用 XMLHttpRequest 一次上傳一個圖片並解析 `resolve` Promise 傳入 JSON 的 `location`。

### 範例

```js
const example_image_upload_handler = (blobInfo, progress) =>
  new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.withCredentials = false;
    xhr.open('POST', 'postAcceptor.php');

    xhr.upload.onprogress = (e) => {
      progress((e.loaded / e.total) * 100);
    };

    xhr.onload = () => {
      if (xhr.status === 403) {
        reject({ message: 'HTTP Error: ' + xhr.status, remove: true });
        return;
      }

      if (xhr.status < 200 || xhr.status >= 300) {
        reject('HTTP Error: ' + xhr.status);
        return;
      }

      const json = JSON.parse(xhr.responseText);

      if (!json || typeof json.location != 'string') {
        reject('Invalid JSON: ' + xhr.responseText);
        return;
      }

      resolve(json.location);
    };

    xhr.onerror = () => {
      reject(
        'Image upload failed due to a XHR Transport error. Code: ' + xhr.status
      );
    };

    const formData = new FormData();
    formData.append('file', blobInfo.blob(), blobInfo.filename());

    xhr.send(formData);
  });

tinymce.init({
  selector: 'textarea', // change this value according to your HTML
  images_upload_handler: example_image_upload_handler,
});
```

### 整合 Laravel with axios 範例

```js
const image_handler = () => {
  const axios = (window as any).axios;
    const formData = new FormData();
    formData.append('file', blobInfo.blob(), blobInfo.filename());
    return axios
      .post(route('dashboard.assets.create'), formData, {
        onUploadProgress: (e) => {
          progress((e.loaded / e.total) * 100);
        },
      })
      .then((response: any) => {
        return response.data.location;
      })
      .catch((error: any) => {
        console.log(error);
      });
}
```

```js
// async 範例
const handler: UploadHandler = async (blobInfo, progress) => {
    const axios = (window as any).axios;
    const formData = new FormData();
    formData.append("file", blobInfo.blob(), blobInfo.filename());
    try {
      const response = await axios.post("/upload", formData, {
        onUploadProgress: (e: ProgressEvent) => {
          progress((e.loaded / e.total) * 100);
        },
      });
      return response.data.location;
    } catch (error) {
      console.log(error);
    }
  };
```

## images_upload_base_path

該參數可以指定 `basepath` 會在設定的 `images_upload_url` 加上該路經

```js
tinymce.init({
  selector: 'textarea',
  images_upload_url: 'postAcceptor.php',
  images_upload_base_path: '/some/basepath',
});
```

## images_upload_credentials

`images_upload_credentials` 參數設定當呼叫 `images_upload_url` 網址時應該一起傳送憑證例如 Cookie 或者 Header 中驗證的資料。當其設定為 `true` 時憑證會一併傳給 Handler 類似於 `withCredentials` 屬性

```js
tinymce.init({
  selector: 'textarea',
  images_upload_url: 'postAcceptor.php',
  images_upload_credentials: true,
});
```

## images_reuse_filename

預設 TinyMCE 會為每一個上傳檔案產生唯一的檔名。有時這可能會造成非預期的副作用。例如當 `automatic_uploads` 開啟時，使用 [Enhanced Image Editing](https://www.tiny.cloud/docs/tinymce/6/editimage/) 套件編輯圖片，即使圖片一樣，但其結果會變成不同檔名上傳。

設定 `images_reuse_filename` 為 `true` 則 TinyMCE 會使用真實圖片檔名，而不是產生新的名稱。需要注意對應 `<img>` 的 `src` 會被換成上傳到伺服器的檔名，下一次同樣的檔名依舊會被上傳。

```js
tinymce.init({
  selector: 'textarea',
  automatic_uploads: true,
  images_upload_url: 'postAcceptor.php',
  images_reuse_filename: true,
});
```

## CORS 因素

設定跨來源資源共用來上傳圖片到不同的網域。

CORS 包含了一些限制，即使是上傳圖片到同一個伺服器，但有些情況還是需要一些 CORS Headers 例如

- 同樣網域，不同 Port
- 使用 IP 而不是網域
- 頁面和上傳程式的 HTTP 和 HTTPS 不一樣

呼叫上傳程式的 URL 來源和當前頁面的 URL 來源必須要一致。否則需要 CORS headers。通過相對路徑來設定上傳程式可以避免這點。

如果發生 CORS 錯誤您可以在瀏覽器開發者工具 Console 找到。

[PHP 範例](https://www.tiny.cloud/docs/tinymce/6/php-upload-handler/) 有提供關於 CORS 的設定，參考 `$accepted_origins` 。

## 快速實作 Laravel 整合 TinyMCE with TypeScript

1. 安裝套件

   ```sh
   $ npm install --save tinymce @tinymce/tinymce-react
   ```

2. 使用 npm build 方式可參考[官網](https://www.tiny.cloud/docs/tinymce/6/react-pm-host/)，這裡使用對應版本 CDN 的方式。在 `app.blade.php` 加入對應版本的 TinyMCE Script

3. 簡易 Controller 範例

   ```php
   Route::get('/', function () {
       return Inertia::render('Index');
   });

   Route::post('/upload', function (Request $request) {
       if ($request->hasFile('file')) {
           $path = $request->file('file')->storePublicly('tinymce');
           return response()->json(['location' => Storage::url($path)]);
       }
   });
   ```

4. JS 範例

   ```tsx
   import React from 'react';
   import { useForm } from '@inertiajs/inertia-react';
   import { Editor } from '@tinymce/tinymce-react';

   enum ToolbarMode {
     default = 'wrap',
     floating = 'floating',
     sliding = 'sliding',
     scrolling = 'scrolling',
   }
   interface BlobInfo {
     id: () => string;
     name: () => string;
     filename: () => string;
     blob: () => Blob;
     base64: () => string;
     blobUri: () => string;
     uri: () => string | undefined;
   }

   type ProgressFn = (percent: number) => void;

   type UploadHandler = (
     blobInfo: BlobInfo,
     progress: ProgressFn
   ) => Promise<string>;

   export default function Index() {
     const form = useForm<{
       content: string;
     }>({
       content: '',
     });

     const handler: UploadHandler = async (blobInfo, progress) => {
       const axios = (window as any).axios;
       const formData = new FormData();
       formData.append('file', blobInfo.blob(), blobInfo.filename());
       try {
         const response = await axios.post('/upload', formData, {
           onUploadProgress: (e: ProgressEvent) => {
             progress((e.loaded / e.total) * 100);
           },
         });
         return response.data.location;
       } catch (error) {
         console.log(error);
       }
     };

     return (
       <Editor
         id='content'
         init={{
           height: 600,
           menubar: false,
           paste_data_images: true,
           plugins:
             'advlist autolink lists image link quickbars charmap preview anchor searchreplace visualblocks code fullscreen insertdatetime media table code help wordcount',
           toolbar:
             'code undo redo | styles | ' +
             'fontsize forecolor backcolor | ' +
             'bold italic underline strikethrough subscript superscript removeformat | ' +
             'alignleft aligncenter alignright | ' +
             'bullist numlist outdent indent |' +
             'quickimage media link table',
           toolbar_mode: ToolbarMode.default,
           content_style: 'img { max-width: 100%; }',
           images_upload_handler: handler,
         }}
         initialValue={form.data.content}
         onEditorChange={(value) => form.setData('content', value)}
       />
     );
   }
   ```

## 參考資源

- [Tiny Docs - Handling image uploads](https://www.tiny.cloud/docs/tinymce/6/upload-images/)
- [Laravel 9 + Inertia.js + React + TypeScript](https://andyyou.medium.com/laravel-9-inertia-js-react-typescript-32cc8096d123)
- [Typescript with Laravel](https://laravel-news.com/typescript-laravel)

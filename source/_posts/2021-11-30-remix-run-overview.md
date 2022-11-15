---
title: "Remix 潮什麼？"
date: 2021-11-30 10:24:24
tags:
  - react
  - javascript
categories: Program
post_asset_folder: true
---

## 前言

本文只是快速看一下 Remix 的賣點，至於推不推暫時不好說，待筆者更深入研究分享，但..就目前官網提供的下面賣點，我是已經被點火了。

Remix 專注在網頁的基礎組成和 UX，更簡單的建置更棒的網站。

> 筆者：乍看之下，用不精準的感覺形容就是 Meteor 流星 React 版。

<!-- more -->

Remix 是一個全端網頁框架，讓您回到網頁基礎組成並專注在使用者介面進而提供一個快速，流暢，有彈性的使用者體驗。

Remix 無縫整合伺服器和瀏覽器，通過分散式系統和原生瀏覽器的功能支援高效的頁面載入和即時轉換效果，而不是使用靜態建置。基於 Web Fetch API 可以在任何地方執行。原生支援 Cloudflare Workers 當然 serverless 架構和傳統的 Node 環境都可，您可以選擇偏好的使用方式。

速度只是目標的一部分。Remix 追求更好的使用者體驗。從最基本的請求到華麗的效果 Remix 都可以支援您達成。

## Remix 的秘技：Nested Routes

大部分網站通常具有不同的導覽階層，進而控制不同階層的呈現，通常載入資料和拆分的程式碼片段或元件組成也和網址組成有語意上的關聯。利用巢狀路由 Remix 幾乎可以移除處理載入資料到 `state`的部分。

{% asset_img 1.png %}

{% asset_img 2.png %}

{% asset_img 3.png %}

大部分的網站都在元件內部 `fetch` 資料，逐層建立 `request`，導致降低載入速度和卡頓。Remix 則是在伺服器端平行的載入資料並傳送完整的 HTML，更快，不會卡頓。如此您就可以不用 Spinner, Loader 效果等元件。SSR 也可省去。巢狀路由讓您的應用程式可以更快更即時。

{% asset_img 4.png %}

另外，Remix 可在使用者點擊一個連結之前預先平行載入任何東西；資料，模組，CSS 等。0 載入狀態，0 載入前的架構 UI 畫面，0 卡頓。

## 資料更新

在您應用程式中的程式碼大多數都是為了變更資料。想像如果 React 只有 `props` 沒有 `state` 會如何？如果有個網頁框架協助您載入資料卻沒有幫助您更新狀態。

而 Remix 則不會在`form onSubmit` 進行到一半就不管你。表單範例如下：

```jsx
export default function NewInvoice() {
  return (
    <Form method="post">
      <input type="text" name="company" />
      <input type="text" name="amount" />
      <button type="submit">Create</button>
    </Form>
  );
}
```

然後對應在伺服器端處理的行為

```js
export async function action({ request }) {
  let body = await request.formData();
  // ...
}
```

Remix 會在伺服器端執行 `action` 重新驗證資料甚至也會協助處理反覆送出表單 Race conditions 的問題。

還可以使用最新的 Transition Hooks 實作過度狀態介面，Remix 會處理狀態，您只需讀取即可。或者使用 Optimistic UI 模式您可以直接模擬更新後的結果，Remix 會提供送到伺服器端的資料，您就可以略過使用 Spinner 效果：

```jsx
export default function NewInvoice() {
  let { submission } = useTransition();
  return submission ? (
    <Invoice
      invoice={Object.fromEntries(
        submission.formData
      )}
    />
  ) : (
  	<Form method="post">
      <input type="text" name="company" />
      <input type="text" name="amount" />
      <button type="submit">Create</button>
    </Form>
  );
}
```

## 錯誤處理

當您的應用程式遇到例外的狀態，使用 Remix 不需要瀏覽器重新刷新頁面。錯誤處理機制也是內建。Remix 會協助您處理伺服器渲染，客戶端渲染的錯誤，甚至是伺服器端處理資料時發生的錯誤。每一個路由元件可以匯出一個錯誤邊界元件。如果錯誤發生會看到邊界元件的內容。

## 資料參考

* [Remix 官網](https://remix.run/)
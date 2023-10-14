---
title: "[譯] Next.js 13 vs Remix 深入解析"
date: 2023-10-14 11:00:00
tags:
  - javascript
  - nextjs
categories: Program
---

> [原文連結](https://prateeksurana.me/blog/nextjs-13-vs-remix-an-in-depth-case-study/?ck_subscriber_id=887777478)
> 備註：本文有略微簡化原文內容並提供一些補充，目標提供一個較簡化流暢容易理解的版本。另外本文改寫了 tsx 的範例希望能更容易了解觀念。


提到建置網頁應用程式，React 已經成為主流好一陣子了（目前 2023），並且採用率持續在上升。常見搭配 React 開發網站的方式中，Next.js 是比較主流的選項之一。

自從去年他們發佈了 [App Router](https://nextjs.org/blog/next-13) 等重大更新以來也一直備受關注。新的路由架構採用了嵌套佈局（Nested Layouts）並緊密整合了 `React Server Component` 和 `Suspense` 。

<!-- more -->

但 Next.js 並不是第一個實作**基於佈局路由的框架**。大約在 1 年前，Next.js 公佈新的路由架構之前，Remix 就已經發佈了第一版。Remix 的開發者就是 [React Router](https://reactrouter.com/en/main) 的開發者。

Remix 背後的概念非常簡單就是一個 Edge-first full-stack framework，支援雲端邊緣進行高效運算，面向用戶而設計的全端框架。具體來說就是鼓勵使用[標準網頁 API](https://developer.mozilla.org/en-US/docs/Web/API/Response) 如 `Request`, `Response`，`FormData` 等 API 並且支援嵌套佈局，並行下載資料，處理 Race Conditions，在 JavaScript 開始載入之前網站也可以運行。Remix 團隊的核心思想就是 - 當你熟悉和精通 Remix 框架時，你也會同時提升對網路基礎知識的掌握。

Remix 背後的理念很棒，但同時 Next.js 擁抱 RSC 的方向也令人感到興奮。

本文會通過復刻一個 [X 網站的範例](https://github.com/prateek3255/twitter-clone)，使用兩個框架的核心功能來比較。藉由過程中學習到的經驗以及是否該借鏡另一個框架的分析。



## 佈局 Layout

提到佈局，兩個框架採用類似的路由設計，都支援在瀏覽頁面之間保留延用共用的嵌套佈局。大多數網站在不同頁面都有共用的佈局。無論是側邊欄或是儀表板中的 Tab，共用佈局幾乎到處都是。X 網站也一樣。事實上，有些頁面甚至有一個佈局嵌套在另一個佈局的情況。例如幾乎全部頁面有側邊欄，也有 Tab 

![](https://prateeksurana.me/img/twitter-clone-user-profile-page-1200.webp)

假如您已經使用 Next.js 12 以前的版本，你就知道在 `_app.tsx` 建立 Layout 函式的複雜度，如果您的 Layout 需要從伺服器讀取資料則事情會更複雜。通常必須得在 `getServerSideProps` 重複 Layout 所需資料的邏輯。

但現在，不管 Remix 或 Next.js 13 您可以根據框架提供的基於檔案系統的路由架構來處理 Layout

### Remix

使用 Remix，在最新的版本 2 中，您可以使用 `.` 分隔符號來產生對應網址上的 `/`，舉例來說 `app/routes/invoices.new.jsx` 將會對應 `/invoices/new` 路由，檔案 `app/routes/invoices/$id.jsx` 會對應 `/invoices/{id}` 這裡的 `id` 代表發票 ID。

在你的發票相關 URL 共用相同的 Layout 也就是擁有 `/invoices` 這路由段的網址，您可以建立一個 `invoices.tsx` 檔案裡面包含 Layout 所需的元素。在這個檔案，您可以加入 `<Outlet />` 元件，這裡使用 `<Outlet />` 元件放置子路由的渲染內容。舉例來說就是 `/invoices/new` 和 `/invoices/{id}` 頁面會共用 `/invoices` Layout。

當然，您可能需要在不同 URL 下使用 Layout。Remix 提供了一個解決方案，您可以建立路由搭配 `_` 前綴，如此這個檔案名稱就不會對應到網址。

支援的功能讓我們可以組合出強大的嵌套佈局。

除了側邊欄幾乎每個頁面都有。用戶資料頁面頁需要一個獨立的 Layout 因爲它還包含了推文，留言和讚，各自需要獨立的頁面和網址。檔案結構大概如下：

```
app/
  routes/
    _base.tsx
    _base._index.tsx -->   /
    _base.$username.tsx
    _base.$username._index.tsx -->   /{username}
    _base.$username.replies.tsx -->   /{username}/replies
    _base.$username.likes.tsx -->   /{username}/likes
    _base.status.$id.tsx -->   /status/{id}
    _auth.tsx
    _auth.signin.tsx -->   /signin
    _auth.signup.tsx -->   /signup
```

這裡 `_base.tsx` 是最外層主要的 Layout 包含了側邊欄和大部分頁面需要的元素。然後 `_base.$username.tsx` Layout 嵌套在 `_base`  包含了表頭，Tab 

![](https://prateeksurana.me/img/remix-layout-for-user-profile-page-1200.webp)



### Next.js

Next.js 13 的 `app` 目錄也提供非常類似的佈局系統。主要的差異是使用目錄和目錄下的檔案來組織，檔案如 `layout.tsx` 就是佈局用，然後 `page.tsx` 讓該路由可以被存取並且作為 `children` 填入 Layout 中。

實際上，在 Next.js 13 更進一步支援每個路由段的 `loading.tsx` 和錯誤處理 `error.tsx` 。

建立不共用 URL 的 Layout 和 Remix 很類似，差別是不用 `_` 底線前綴，我們建立目錄搭配括號例如 `(folderName)`。這個目錄在 Next.js 稱作路由群組。動態路由區段是使用 `[]`。同樣功能在 Next.js 下，結構如下
``` 
app/
  (base)/
    [username]/
      likes/
        page.tsx -->   /{username}/likes
      replies/
        page.tsx -->   /{username}/replies
      layout.tsx
      page.tsx -->   /{username}
    status/[id]/
      page.tsx -->   /status/{id}
    layout.tsx
    page.tsx  -->   /
  (auth)
    signin/
      page.tsx -->   /signin
    signup/
      page.tsx -->   /signup
```

![](https://prateeksurana.me/img/nextjs-layout-for-user-profile-page-1200.webp)

### 小結

現在如果我們比較兩個路由架構，我會比較偏好 Remix 因為比較直觀，透過檔名就可以知道佈局了。而 Next.js 最終大概率會產生一大堆 `page.tsx` 和 `layout.tsx`。

但必須要說，Next.js 這麼做是因為這不只是跟頁面和佈局有關，同時也是為了支援 `notFound.tsx`，`loading.tsx`，`error.tsx` 等情況。另一個好處就是您可以在目錄下直接看到路由相關的處理。

無論那種方式，兩個框架的設計方向挺接近的都是採用檔案系統路由。



## 資料讀取

在現代網頁應用程式中，資料讀取至關重要。起初，大部分 React App 都是在客戶端渲染，伺服器傳來一個空的 `index.html` 並且其中的 `<script />` 包含相關的 JavaScript 。結果就是當瀏覽器還在下載的時候，初始頁面是空白的，然後執行 JavaScript，React 初始化開始讀取資料，渲染元件。這在效能不高或網路不好的裝置上嚴重影響效能。

Next.js 和 Gatsby 在這方面進行了主要的改變，它們簡化了 “在伺服器” 或 “在編譯時期” 讀取資料的過程，支援預先渲染初始的 HTML。因此，用戶在你的網站首次載入時就已經有了初始的畫面。雖然還是需要等待 JavaScript 被下載和 React Hydrate 後，才能開始操作。

而現在 Next.js 13 和 Remix 都更進一步解決了這個問題；Next.js 採用 React Server Component，Remix 則是提出 Loader 和並行資料讀取的方式。



### Remix

提到 Remix，讀取資料的方式是使用 Loader ，每一個路由可以定義 `loader` 函式，在渲染時提供路由相關資料。第一次渲染時 `loader` 只會在伺服器端執行。

下面是一個 Remix 使用 `loader` 的範例，我們在 `_base.jsx` 這個 Layout 元件使用

```js
export const loader = async ({ request }) => {
  const currentLoggedInUser = await getCurrentLoggedInUser(request);
  return json({
    user: currentLoggedInUser,
  }, {
    status: 200,
  });
}

export default function RootLayout() {
  const { user } = useLoaderData();
}
```



Loader 會取得 Fetch `Request` 物件參數，然後您可以通過這個物件讀取 `headers`，`cookies` 等。Loader 回傳的型別是 `Response` 物件。Remix 提供了一些輔助函式如 `json`，`redirect` 等等，讓您可以回傳特定 Response 搭配 HTTP 狀態碼，接著您就可以在元件中使用 `useLoaderData` Hook。

使用 Remix 由於您可以在每個路由段定義 `loader` 當然包含 Layout，每一個路由段都可以並行讀取資料，如此就不用在瀏覽器瀑布式的讀取資料（發送多個請求）。

另外，Loader 不只被用來協助在伺服器端渲染頁面。因為它回傳的是 Fetch `Response` ，所以 Remix 在換頁或重現驗證的時候也會透過 `fetch` 來呼叫 Loader。

### Next.js

隨著 Next.js 13 引進了 `app` 目錄架構，Next.js 已經從只能在 `getServerSideProps` 或 `getStaticProps` 幫頁面元件從伺服器讀取資料轉向到支援 React Server Component

RSC 是一個非常大的主題，除了解決和伺服器讀取資料外還解決了很多問題包含 CSR Client-Side Rendering 性能問題，SSR Server-Side Rendering 的 Time to First Byte(TTFB) 很慢。具體詳細介紹可以[參考](https://prateeksurana.me/blog/future-of-rendering-in-react/)。

簡單的說， RSC 是 React 一個新典範。它們是只在伺服器端渲染的元件，不像傳統的 SSR，也不需要在客戶端 Hydrate，優點包括：

- 資料擷取的安全性：因為只能在伺服器執行，可以直接使用一些機敏資料，直接呼叫 API 不用擔心暴露給外部
- 可預期的 Bundle 大小（Deterministic Bundle Size）：相依套件會影響客戶端下載的 JavaScript bundle 大小，假如只在伺服器元件（Server Component）中使用，現在完全不用下載了。例如 Markdown 解析功能的相依套件，如果在伺服器元件中處理，那麼就不用下載到客戶端再執行。

互動操作的部分，您需要使用客戶端元件（Client Component）。不像它們的名稱，客戶端元件在伺服器端也可以渲染，但是會遵循一般 SSR 的流程，還是必須要等待 JavaScript 下載和 Hydrate。

RSC 並不是萬能的銀子彈，不能解決所有問題，它還是有一些限制：

- 因為只能在伺服器執行，且不會 Hydrate，它們不能包含互動操作的程式碼，像是 `useState`，`useEffect`，事件處理函式，瀏覽器的 API 。在您需要互動操作的部分還是要使用客戶端元件。
- 您不能在一個“客戶端元件”中匯入“伺服器元件”。由於伺服器元件只能在伺服器渲染，我們需要清楚掌握伺服器元件的結構組成，雖然還是有些方法可以組合它們。

**在 `app` 目錄，所有元件預設都是“伺服器元件“**，並且如果您希望加入互動的部分，您需要加入客戶端元件到結構樹。客戶端元件通過在檔頭使用 `use client` 宣告。同時，伺服器元件和客戶端元件不可以是同一個檔案。

下面是跟上面 Remix `base` Layout 一樣功能的範例：

```js
export default async function BaseLayout({
  children
}) {
  const user = await getCurrentLoggedInUser();
  const isLoggedIn = !!user;
  
  return (
  	...
  );
}
```

由於它們都在伺服器端渲染，伺服器元件可以回傳 Promise。因此您只要在元件中 `await` 讀取資料，然後直接使用即可。

> 反過來說原本我們會避免在客戶端元件直接使用 await，因為會阻塞渲染，我們通常會用 useEffect，而使用 RSC 可以就直接使用 await 即可。



Next.js 同時也提供一些輔助函式如：`headers`，`cookies`，`redirect`，`revalidatePath` 等等，讓您可以在伺服器端存取請求的資料以及一些只能在伺服器上執行的操作。`getCurrentLoggedInUser` 方法實際上使用 `cookies` 的資訊，再從資料庫讀取當前登入用戶的資料。

這確實是一個改變遊戲規則的功能，潛力無限，因為您不只可以直接用可讀性更佳的方式從資料庫讀取資料，更進一步在每個元件中而不再被侷限在路由段（Route Segment），只要在伺服器元件即可達成。

Next.js 13 推薦的組織方式是將客戶端元件放在樹狀結構的末端，需要和狀態互動或使用瀏覽器 API 的地方。下面就是用戶頁面的結構示意圖：

![](https://prateeksurana.me/img/nextjs-server-and-client-component-1200.webp)



Next.js 渲染文件也提供了一張表格協助您如何決定該使用那種元件：

![](https://prateeksurana.me/img/nextjs-server-and-client-components-comparision-table-1200.webp)

### 小結

雖然 Remix 內建 Loader 的方式很不錯，讓子路由也可以並行讀取資料，並且容易重新驗證。但 React Server Component 可能是相對正確的選擇。

Remix 也認同 RSC 帶來的好處，他們也計畫在未來整合 RSC 的功能。

值得一提一個關於 Loader 的問題是；我們可以在和元件相同的檔案下定義 Loader。雖然編譯工具在區分客戶端和伺服器端封裝方面作的很好，但還是可能導致伺服器的機密資訊意外洩漏，或者將限制在伺服器的封裝檔 Bundle 發送到客戶端。Remix 文件也有說明在導入路由段中的模塊時需要注意的陷阱。沒錯！使用 Next.js 的 `getServerSideProps` 也存在相同的警告。



## 串流

React 18 支援了串流和 Suspense，這兩個功能讓我們可以利用串流傳輸，逐步渲染 UI 單元。

串流的功能讓阻塞的 Layout 局部或某個路由段可以顯示載入效果，而不是延遲整個頁面直到全部資料準備完成，伺服器首先回傳各個對應區塊的載入狀態，然後當資料讀取完成則會取代該區塊。這在 [Next.js 串流文件](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming) 有說明而且解釋的很好。

![](https://prateeksurana.me/img/streaming-with-suspense-1200.webp)



Remix 和 Next.js 13 都支援串流和 Suspense。

> 這裡的串流跟所謂的影音串流很類似，只是對象變成把 HTML 切割傳輸。另外提供一張比較表格
> ![](https://imgur.com/l5X0vwC.png)



### Remix

使用 Remix，您可以單純的使用 `defer` 輔助函式把想要串流處理的地方直接回傳 Promise 而不是完成 `resolve`  。然後在元件中，您可以使用 `Await` 元件來處理延遲的 Loader 並搭配 Suspense 顯示載入效果。

下面是一個簡化的範例，注意 `tweets` 和 `currentLoggedInUser` 的差異：

```jsx
export const loader = async ({ request, params }) => {
  const username = params.username;
  
  return defer({
    tweets: getTweetsByUsername(request, username),
    currentLoggedInUser: await getCurrentLoggedInUser(request),
  });
};

export default function UserTweets = (props) => {
  const data = useLoaderData();
  consolelog(props.currentLoggedInUser.name);
  
  return (
  	<Suspense fallback={<Spinner />}>
    	<Await
				resolve={props.tweets}
				errorElement={<p>Wrong!</p>}
			>
        {(initialTweets) => (
        	{/* 渲染元件 */}
        )}
      </Await>         
    </Suspense>
  );
}
```



上面的範例，注意到 `getCurrentLoggedInUser` 會等到資料讀取完成才往下執行，因此這不是串流，您直接就可以在元件使用該資料。

> 請參考[原文提供的影片](https://prateeksurana.me/blog/nextjs-13-vs-remix-an-in-depth-case-study/?ck_subscriber_id=887777478)



### Next.js

在 Next.js 13 串流更加直覺。當我們稍早討論 Layout 部分時，您可以直接建立一個 `loading.jsx`在對應該路由段的目錄下就可以取得該段的載入狀態。

在底層，Next.js 在路由段處理時就將頁面包在 Suspense 中了同時也搭配了您指定的 `loading.jsx`

![](https://prateeksurana.me/img/nextjs-streaming-with-suspense-1200.webp)



除此之外，如果您希望 Suspense 某些內容不在路由段結構上，您依然可以在一個非同步元件中使用 `Suspense` 達成效果。

舉例來說我希望 tweets 支援載入效果，但不希望建立推文被阻塞。

```jsx
export default async function Home() {
  const user = await getCurrentLoggedInUser();
  
  return (
  	<>
    	{/** Header */}
			{user && (
      	<div className="hidden sm:flex p-4 ...">
          <Image
          	src={user.profileImage ?? DEFAULT_PROFILE_IMAGE}
            className="rounded-full object-contain max-h-[48px]"
            width={48}
            height={48}
            alt={`${user.username}'s avatar`}
          />
          <div className="flex-1 ml-3 mt-2">
            <CreateTweetHomePage />
          </div>
        </div>
      )}
			<Suspense fallback={<Spinner />}>
        <HomeTweets />
      </Suspense>
    </>
  );
}

async function HomeTweets() {
  const initialTweets = await getHomeTweets();

  return (
    /** Render initial infinite Tweets */
  );
}
```



### 小結

串流搭配 Suspense 對於提供好的用戶體驗是非常棒的功能，可以顯著的降低 TTFB。

React Server Components 的簡單性，您可以在 Suspense 中包裝因資料讀取阻塞的元件。



## 資料變更 Data Mutation

提到變更，我們大概率都曾經自己處理，通過發送 API 請求然後更新本地的狀態，進而影響 UI 更新，或者使用 React Query 協助處理大部分的事情。這兩個框架都希望將操作作為其核心功能的一部分來改變這一現狀。

在 Remix 使用 Action，與此同時 Next.js 13.4 也加入了 Server Action。



### Remix

在 Remix，變更需要通過 Action 處理，它是 Remix 的核心功能。Action 通過在路由檔案定義 `export const action` ，類似 Loader，一個 Action 也是只能在伺服器端執行，然後回傳 Fetch Response，但跟 Loader 不一樣，它可以處理非 `GET` 請求，如 `POST` `PUT` 等。

調用 Action 的主要方式是在 Remix 中通過 HTML 表單實現。還記得稍早我們提到，當您越加熟悉 Remix 您也就越加熟悉網頁技術的一些基本知識。這個部分就很明顯。Remix 鼓勵我們將應用程式中用戶操作的每個部分都使用 HTML 表單。沒錯，即使是「讚」按鈕也是這種形式。

每當用戶觸發提交一個表單，它就會根據當下的狀況呼叫最接近路由的 Action，當然你也可以在表單的 `action` 屬性指定 POST 的 URL 。一旦 Action 被執行，Remix 會通過瀏覽器的 `fetch` 針對該路由重新讀取所有的 Loader 刷新，確保 UI 和資料一致。Remix 稱為全端資料流。

來看看範例：

```jsx
export const action = async ({ request }) => {
  const form = await request.formData();
  const usernameOrEmail = form.get('usernameOrEmail')?.toString() ?? '';
  const password = form.get('password')?.toString() ?? '';
  
  const isUsername = !isEmail(usernameOrEmail);
  
  const user = await prisma.user.findFirst({
    where: {
      [isUsername ? 'username' : 'email']: usernameOrEmail,
    },
  });
  
  const fields = {
    usernameOrEmail,
    password,
  };
  
  if (!user) {
    return json({
      fields,
      fieldErrors: {
        usernameOrEmail: `No account found with the given ${
        	isUsername ? 'username' : 'email'
      }`,
        password: null,
      }
    }, {
      status: 400
    });
  }
  
  const isPasswordCorrect = await comparePassword(password, user.password);
  
  if (!isPasswordCorrect) {
    return json({
      fields,
      fieldErrors: {
        usernameOrEmail: null,
        password: 'Incorrect password.',
      }
    }, {
      status: 400,
    });
  }
  
  return createUserSession(user.id, '/');
}

export default function Signin() {
  const actionData = useActionData();
  const navigation = useNavigation();
  
  return (
  	<>
      <h1 className="font-bold text-3xl text-white mb-7">Sign in to Twitter</h1>
      <Form method="post">
        <div className="flex flex-col gap-4 mb-8">
          <FloatingInput
            autoFocus
            label="Username or Email"
            id="usernameOrEmail"
            name="usernameOrEmail"
            placeholder="john@doe.com"
            defaultValue={actionData?.fields?.usernameOrEmail ?? ""}
            error={actionData?.fieldErrors?.usernameOrEmail ?? undefined}
            aria-invalid={Boolean(actionData?.fieldErrors?.usernameOrEmail)}
            aria-errormessage={actionData?.fieldErrors?.usernameOrEmail ?? undefined}
          />
          <FloatingInput
            required
            label="Password"
            id="password"
            name="password"
            placeholder="********"
            type="password"
            defaultValue={actionData?.fields?.password ?? ""}
            error={actionData?.fieldErrors?.password ?? undefined}
            aria-invalid={Boolean(actionData?.fieldErrors?.password)}
            aria-errormessage={actionData?.fieldErrors?.password ?? undefined}
          />
        </div>
        <ButtonOrLink type="submit" size="large" stretch disabled={navigation.state === "submitting"}>
          Sign In
        </ButtonOrLink>
      </Form>
    </>
  );
}
```



對於表單提交會導致 URL 變更，Remix 提供了一個 `Form` 元件，這是一個漸進式增強的 Wrapper 主要針對原生 HTML 表單行為進行調整。然後，你可以使用 `useNavigation` ，它提供有關進行頁面導航的資訊，你可以使用這些資訊來處理載入狀態。例如登入頁面，在表單提交時禁用按鈕可以使用 `navigation.state`。

類似 `useLoaderData` ，Remix 也提供了 `useActionData` 作為伺服器和客戶端的溝通橋樑。

同時，注意到我們沒有使用任何狀態來管理 `input`，取而代之的是我們依靠瀏覽器的預設行為來序列化所有表單的欄位 POST 到伺服器。一旦表單提交，`action` 就可以通過 fetch Request 的  [formData](https://developer.mozilla.org/en-US/docs/Web/API/Request/formData) 方法讀取資料。

可是，我們不希望每一次當表單提交都跳轉頁面。因此 Remix 提供了其他工具來處理表單行為，使其不要跳轉 - 叫做 [fetcher](https://remix.run/docs/en/main/hooks/use-fetcher) 。在這個範例中幾乎剩下的表單都是 `fetcher` 表單

```jsx
export default function TweetStatus() {
  const { tweet, user, replies } = useLoaderData();
  const fetcher = useFetcher();
  const isLoading = fetcher.state !== 'idle';
  
  return (
  	<fetcher.Form method='post'>
      <input type="hidden" name="tweetId" value={originalTweetId} />
      <input type="hidden" name="hasLiked" value={(!tweet.hasLiked).toString()} />
      <TweetAction 
      	size="normal"
        type="like"
        active={tweet.hasLiked}
        disabled={isLoading}
        submit
        name="_action"
        value="toggle_tweet_like"
      />
    </fetcher.Form>
  );
}

export const action = async ({ request }) => {
  const formData = await request.formData();
  const action = formData.get(_action);
  const userId = await getUserSession(request);
  
  if (!userId) {
    return redirect('/signin', 302);
  }
  
  switch (action) {
    case 'toggle_tweet_like':
      {
        const tweetId = formData.get('tweetId');
        const hasLiked = formData.get('hasLiked') === 'true';
        await toggleTweetLike({
          request,
          tweetId,
          hasLiked,
        });
      }
      break;
    case 'toggle_tweet_retweet':
      {
        // ...
      }
      break;
    case 'reply_to_tweet':
      {
        // ...
      }
      break;
  }
  
  return json({ success: true });
}
```

注意我們是如何在表單使用 `<input type="hidden" />` 來傳遞相關資料如 `tweetId` 和 `hasLiked`。我們同時也設定了按鈕的名稱 `_action` 和值 `toggle_tweet_like` 讓伺服器知道我們希望觸發的行為，這在頁面有多個表單時很實用。

現在我們已經看過 Remix 的全端資料流了，Remix 會通過瀏覽器的 `fetch` 自動執行 Loader，更新介面上那些從 Loader 讀取相關資料的地方。因此，像是推文， 讚的數量和按鈕的狀態會自動更新。（可觀看原文效果影片）。

**因為 Remix 強制使用 HTML 表單，因此預設瀏覽器就是序列化表單欄位資料，然後送到伺服器，用戶可以在 JavaScript 載入之前就操作介面。您甚至可以停止 JavaScript 網頁的行為大部分都還是正常運作。**



### Next.js

在 Next.js 13.4 之前，您只能依靠建立 API Route 來執行伺服器端的行為。任何在 `pages/api` 目錄下的檔案會被視為 API 而不是頁面元件。

雖然這對於一次性的 API 是不錯的解法（例如，處理表單提交或執行一些後端邏輯），但它沒有提供一個完整的解決方案來處理客戶端的 API 呼叫或資料重新驗證。（例如用戶資料的 API 你可能要在各個有用到的元件自行處理刷新以避免資料是過期的）。

這也是為什麼 [trpc](https://trpc.io/docs/quickstart) 越來越受歡迎的原因之一，因為可以利用 Next.js 的 API 路由搭配 React Query 來處理 API 請求和資料更新。

現在，Next.js 13.4 引進了 Server Action，目前處於實驗階段。

有了 Server Action 你就不需要建立 API 了。您可以直接建立非同步的伺服器函式直接在元件中呼叫，並且可以存取 Next.js 伺服器端的功能如 `cookies`，`revalidate`，`redirect`等。這個功能可以有效大量減少實現同樣功能的程式碼。

在元件中，您可以通過 `'use server'` 來定義 Server Action，然後將其直接設定到表單 `form` 的 `action` 屬性或傳入客戶端元件使用。直接看下面範例：

```jsx
export default async function Page() {
  async function createTodo(formData) {
    'use server'
    // ...
  }
  
  return <form action={createTodo}>...</form>
  // 或
  return <ClientComponent createTodo={createTodo} />
}
```



當然您可以獨立建立一個檔案搭配 `'use server'` ，在下面範例中全部 export 的函式都可以作為 Server Action

```js
'use server'

export async function create()
{
  // ...
}
```

```jsx
'use client'

import { create } from './actions';

export default function Button() {
  return (
  	<form action={create}>
    	<button type="submit">Create</button>
    </form>
  );
}
```

當在客戶端元件使用 `action` 屬性，Action 會被放到佇列中直到表單元件完成 Hydrate。因為支援 [Selective Hydration](https://www.patterns.dev/posts/react-selective-hydration) 這個表單會被優先處理，以便能夠盡快變得可以操作。

現在讓我們看一些例子。登入頁面的代碼是這樣的：

```jsx
export default function Signin({ searchParams }) {
  const signin = async (formData) => {
    'use server';
    
    const auth = {
      usernameOrEmail: formData.get('usernameOrEmail')?.toString() ?? '';
      password: formData.get('password')?.toString() ?? '';
    };
  
    const isUsername = !isEmail(auth.usernameOrEmail);
    const user = await prisma.user.findFirst({
      where: {
        [isUsername ? 'username' : 'email']: auth.usernameOrEmail,
      },
    });
  	if (!user) {
      const error = encodeValueAndErrors({
        fieldErrors: {
          usernameOrEmail: `No account found with the given ${
        		isUsername ? 'username' : 'email'
         	}`,
        },
        fieldValues: auth,
      });
      return redirect(`/signin?${error}`);
    }
  
  	const isPasswordCorrect = await comparePassword(
    	auth.password,
      user.passwordHash
    );
  
  	if (!isPasswordCorrect) {
      const error = encodeValueAndErrors({
        fieldErrors: {
          password: 'Incorrect password',
        },
        fieldValues: auth,
      });
      return redirect(`/signin?${error}`);
    }
  	setAuthCookie({
      userId: user.id,
    });
  	return redirect("/");
  };
	
	const { fieldErrors, fieldValues } = decodeValueAndErrors({
    fieldErrors: searchParams.fieldErrors,
    fieldValue: searchParams.fieldValues,
  });

	return (
  	<>
    	<h1>Sign In</h1>
    	<form action={signin}>
    		<div>
      		<FloatingInput
          	autoFocus
            label="Username or Email"
            id="usernameOrEmail"
            name="usernameOrEmail"
            placeholder="john@doe.com"
            defaultValue={fieldValues?.usernameOrEmail}
            error={fieldErrors?.usernameOrEmail}
            aria-invalid={Boolean(fieldErrors?.usernameOrEmail)}
            aria-errormessage={fieldErrors?.usernameOrEmail ?? undefined}
          />
          <FloatingInput
            required
            label="Password"
            id="password"
            name="password"
            placeholder="********"
            type="password"
            defaultValue={fieldValues?.password}
            error={fieldErrors?.password}
            aria-invalid={Boolean(fieldErrors?.password)}
            aria-errormessage={fieldErrors?.password ?? undefined}    
          />
      	</div>
      	<SubmitButton>Sign In</SubmitButton>
    	</form>
    </>
  );
}
```

雖然到目前為止，Next.js 沒有像 Remix 的 `useActioData` 這樣的 Hook 來取得 Server Action 的回應。但我們可以用一種不需要 JavaScript 的方式來顯示錯誤，因此我們採用了 Query String（Search Parameters）的方式。

而 `SubmitButton` 是一個客戶端元件，使用了一個實驗的 Hook 叫 `useFormStatus` 來實現當送出表單時禁用按鈕的功能。

```jsx
'use client';
import { experimental_useFromStatus as useFormStatus } from 'react-dom';
import { ButtonOrLink } from 'components/ButtonOrLink';

export const SubmitButton = ({ children }) => {
  const { pending } = useFormStatus();
  
  return (
  	<ButtonOrLink
      type="submit"
      size="large"
      disabled={pending}
    >
      {children ?? "Submit"}
    </ButtonOrLink>
  );
}
```

在客戶端，您也可以使用 `startTransition` API 執行 Server Action 進行伺服器端資料變更（執行 `revalidatePath` ， `redirect` 或 `revalidateTag`）

```jsx
const [isPending, startTransition] = useTransition();

<ButtonOrLink
	disabled={isPending}
  onClick={() => {
    startTransition(async () => {
      await toggleFollowUser({ userId: profileUserId, isFollowing: true });
    })
  }}
>
  Follow
</ButtonOrLink>
```



類似 Remix，您可以直接從 Server Action 重新驗證，進而讓過期的伺服器元件失效，然後介面會跟著回應自動更新。雖然不像 Remix，您必須手動呼叫 `revalidatePath` 來刷新特定路徑的資料。

```jsx
export const toggleFollowUser = async ({ userId, isFollowing }) {
  // ... 更新資料庫
  revalidatePath('/[username]');
}
```



### 小結

Remix 的 Action 設計和完整的全端資料流，自動化重新讀取 Loader 和更新介面，同時讓 App 在 JavaScript 載入之前就能運作非常不錯，不只改善的用戶體驗，開發者體驗也很明顯的得到改善。

但 Action 也有需要注意的地方，和我們之前探討 Loader 一樣，只能在路由段定義。也就是很可能類似的邏輯到處都是，或者重複使用一個 Action，就必須在 `form` 的 `action` 指定 URL，然後隨著應用的成長確實可能造成混亂。

Next.js 的 Server Action 通過讓你建立到處都可以執行的函式解決這個問題。然而，在這個時間點沒有支援自動重新驗證的機制，而且感覺非常不穩定，文件也缺少。

> Next.js 13.5 發佈之後有些 API 不像上面提到的，例如使用 startTransition 伺服器端變更資料  似乎沒有在文件上了。



## 無限載入 Infinite Loading

無限載入曾經是個有趣的問題，因爲不管那個框架都沒有很好的支援。

當你需要無限載入這個功能，您必須自己在客戶端處理，大概需要在客戶端使用狀態來處理。



### Remix

關於無限載入您可以參考 [Full Stack Component](https://www.epicweb.dev/full-stack-components) ，其中最重要的就是關於 [Resource Routes](https://remix.run/docs/en/main/guides/resource-routes) 的利用。

概念就是您建立一個類似一般路由的模組，但不要 `export default` ，你還是可以使用 Loader 和 Action ，有點像 Next.js 的 API routes。

所以在 Remix，可以建立一個新的路由 `routes/resource-infinite-tweets.jsx` 然後具名匯出 `InfiniteTweets` 。由於不是預設匯出，Remix 不會為這個路由渲染 UI。簡而言之，這個方法使用了 `IntersectionObserver` API 來檢測頁面的底部，並觸發一個請求來獲取下一頁的推文，然後將其加到狀態來處理。

下面的範例首先第一次是串流載入的，但後續則是使用 `resource-infinite-tweets.jsx` 來載入

```jsx
export const loader = async ({ request }) => {
  const cursor = getSearchParam(request.url, 'cursor') ?? undefined;
  const type = getSearchParam(request,url, 'type');
  const username = getSearchParam(request.url, 'username');
  const tweetId = getSearchParam(request.url, 'tweetId');
  
  let tweets = [];
  
  switch(type) {
    case 'user_tweets':
      tweets = await getTweetsByUsername(request, username, cursor);
      break;
    case 'home_timeline':
      tweets = await getHomeTweets(request, cursor);
      break;
    case 'tweet_replies':
      tweets = await getUserReplies(request, username, cursor);
      break;
  }
  
  return json({
    tweets,
  }, 200)
}
```

現在為了觸發 Loader，我們可以使用 `fetcher` 如我們在資料變更章節看到的一樣。該物件還有一個 `submit` 方法可以直接在程式中觸發 Loader 來取得下一批推文。

```js
React.useEffect(() => {
  if (isLoading || isLastPage || !isVisible || !shouldFetch) {
    return;
  }
  
  fetcher.submit({
    type,
    cursor: lastTweetId,
    ...rest,
  }, {
    method: 'GET',
    action: '/resource/infinite-tweets'
  });
  setShouldFetch(false);
}, [
  isVisible,
  lastTweetId,
  isLoading,
  isLastPage,
  type,
  shouldFetch,
  rest,
  fetcher
]);
```

Effect 會在特定條件觸發，進而讀取頁面需要的資料。這些資料會在 `fetcher.data` 然後通過另一個 Effect 加到狀態管理。

```jsx
React.useEffect(() => {
  if (fetcher.data && Array.isArray(fetcher.data.tweets)) {
    dispatch({
      type: 'add_tweets',
      newTweets: mapToTweet(fetcher.data.tweets, isLoggedIn),
    });
    setShouldFetch(true)l
  }
}, [fetcher.data, isLoggedIn]);
```

這個路由模組也有一個 Action， 使用 `fetcher.Form` 處理其他像是讚和回覆的內容 。



### Next.js

在 Next.js 實作無限載入也非常類似 Remix，主要的差異是 `InfiniteTweets`會是客戶端元件，然後我們使用 Server Action 來載入後續的資料。

和 Remix 一樣，第一次推文的資料是通過串流的方式取得。在串流一節看到的 `loading.jsx` 非常方便。我們只需要把元件加到需要的 Tab，然後 Next.js 會處理 `Suspense`。

下面是我們處理推文的範例：

```jsx
export default async function Profile({ params: { username }}) {
  const [tweets, currentLoggedInUser] = await Promise.all([
    getTweetsByUsername(username),
    getCurrentLoggedInUser(),
  ]);
  
  const fetchNextUserTweetsPage = async (cursor) => {
    'use server';
    const tweets = await getTweetsByUsername(username, cursor);
    return tweets
  };
  
  return (
  	<>
    	{/* Tweets */}
			<InfiniteTweets
      	currentLoggedInUser={
          currentLoggedInUser
            ? {
                id: currentLoggedInUser.id,
                username: currentLoggedInUser.username,
                name: currentLoggedInUser.name ?? undefined,
                profileImage: currentLoggedInUser.profileImage,
              }
            : undefined
        }
        fetchNextPage={fetchNextUserTweetsPage}
        isUserProfile  
      />
    </>
  );
}
```

注意到我們建立了一個 Server Action `fetchNextUserTweetsPage` 然後傳入 `InfiniteTweets` 元件，然後這個元件利用我們傳入的 Server Action 載入後續的資料。

```jsx
useEffect(() => {
  const updateTweets = async () => {
    if (isLoading || isLastpage) {
      return;
    }
    
    setIsLoading(true);
    
    const nextTweets = await fetchNextPage(lastTweetId);
    
    setIsLoading(false);
    
    dispatch({
      type: 'add_tweets',
      newTweetsRemixToTweet(newTweets, isLoggedIn)
    });
  };
  if (isVisible) {
  	updateTweets();
  }
}, [isVisible, lastTweetId, isLoading, isLastPage, fetchNextPage, isLoggedIn]);
```

### 小結

無限載入是唯一兩個框架都必須要利用狀態管理自行處理的功能。

這時候，Next.js 13 Server Action 的可組合性就比較突出。只要在伺服器元件中處理第一頁的資料，然後在元件本身中建立一個用於獲取下一頁的 Server Action，這個 Server Action 直接傳遞給客戶端組件就可以使用。

在 Remix 中，儘管 `fetcher` 和 resource route 確實讓資料讀取變得更容易，但我們還需要為每個路由建立一個單獨的 `loader` 處理第一頁的串流。

無論如何，兩者的解決方案都不是完美的，一般來說，像 React Query 提供的 `useInfiniteQuery` hook 方案可能可以更方便的協助你在客戶端管理這種資料失效（invalidations）以及樂觀更新（optimistic updates）。

> 樂觀更新：立即在用戶界面上反映變更，而不等待服務器的回應。



## 路由

Remix 和 Next.js 都有非常強大的客戶端路由方案，取代全頁重新載入，處理和伺服器之間的溝通，然後更新介面重新渲染對應需要更新的部分（路由段）。

當你在 Next.js 應用中看到一些 `<Link/>` 或者說顯示在瀏覽器 viewport 的 `<Link />`，Next.js 會自動預先加載（prefetch）這些連結對應的頁面。當用戶點擊這些連結時，相關的頁面已經加載好了，所以能夠瞬間顯示，不需要等待。

對於動態路由（網址中有變數的路由），Next.js 會預先加載共用佈局和第一個 `loading.tsx` 文件，並將其快取30秒。當用戶點擊該路由時能立即顯示加載狀態。

Remix 則更進一步提供了 `prefetch` 屬性，讓你可以根據不同的使用情境來設定不同的值。例如 `intent` 這個值不僅預先加載所有的 JavaScript ，甚至當你將滑鼠 hover 到連結上時，還會通過 `<link rel="prefetch">` 標籤來載入下一個路由所需的所有資料。這讓你幾乎能瞬間渲染下一個頁面。



## 錯誤處理

兩個框架在全域和每個路由段都支援優秀的錯誤處理機制包含預期和非預期的錯誤。

在 Remix 裡，和`loader` 和 `action` 類似，你可以匯出命名為 `ErrorBoundary` 渲染該路由段的錯誤狀態。從伺服器到瀏覽器可能發生的非預期錯誤，或者像 404 這樣的預期錯誤都可以處理。要擷取預期的錯誤，你可以從你的 Loader 中拋出一個 Response。例如

```jsx
export const ErrorBoundary = () => {
  const error = useRouteError();
  
  if (isRouteErrorResponse(error) && error.status === 404) {
    return (...);
  }
}
```



在 Next.js 中，你同樣在每個路由段有各自獨立的文件來渲染該路由的錯誤。`error.jsx` 專門用來處理在該路由段中發生的任何錯誤包含瀏覽器或伺服器。透過加入 ErrorBoundary，這和我們看到的與 `loading.jsx` 的 Suspense 很類似。

![](https://prateeksurana.me/img/nextjs-error-boundary-1200.webp)

針對 404 ，Next.js 還有 `not-found.jsx` 一樣是每個路由段都可以設定。可以通過在伺服器元件回傳 `notFound` 函式觸發。



## 快取

Next.js 在快取的部分有比較明顯的改進，支援不同層級的快取。讓我們可以不只快取路由，也可以快取 `fetch` 的資料，更多細節可以參考官方文件。

Remix 的部分沒有提出太多對於快取的想法，因為它單純就是 HTTP，你可以通過 `Cache-Control` headers 來快取回應或使用伺服器端的解決方案 - Redis。



## 結論

如果您讀到這邊，希望您可以學習到一些關於兩個框架的知識。

總結來說多虧了這兩個框架，用 React 建構複雜的全端網頁應用變得快速和容易。

- Remix 利用基礎的 Web API，為你提供一種簡單但功能強大的方式來建立現代網頁應用。

- Next.js `app` 目錄整合 React Server Components 和 Server Actions，大幅縮小 Bundle 大小等等。

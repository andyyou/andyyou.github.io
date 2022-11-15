---
title: "[譯] Remix 快速入門"
date: 2021-12-31 16:56:10
tags:
  - javascript
  - react
categories: Program
---

我們廢話不多說直接進入程式碼的部分。如果您在找尋 15 分鐘快速實作，那本篇就是。

本篇會使用 TypeScript，但我們會在寫完程式之後才添加型別。這不是正常的流程，但因為部分讀者不使用 TypeScript 因此我們不想讓您混亂。一般來說我們應該在寫程式的時候就建立型別，才可以在一開始就享受 TypeScript 的好處。

<!-- more -->

## 建立專案

```sh
$ npx create-remix@latest
Welcome to Remix! Let's get you set up with a new project.

? Where would you like to create your app? [demo-remix]
? Where do you want to deploy? Choose Remix if you're unsure, it's easy to change deployment targets. (Use arrow keys)
❯ Remix App Server
  Express Server
  Architect (AWS Lambda)
  Fly.io
  Netlify
  Vercel
  Cloudflare Pages
? TypeScript or JavaScript? (Use arrow keys)
❯ TypeScript
  JavaScript
? Do you want me to run `npm install`? (Y/n) Y
$ cd demo-remix
$ npm run dev
```

> 須選擇 Remix App Server

我們將對檔案系統做些設定，但不是全部的設定都和這篇教學相容。

瀏覽器開啟 http://localhost:3000 可以看到我們的應用程式正在執行。如果您想，可以花幾分鐘看一下預設的檔案。

如果您的程式沒有正常運作請查閱專案裡的 README.md 看看您是否需要額外的設定。

>請確認 `postinstall` 在啟動應用程式之前正確執行，如果沒有可以手動執行 `npm run postinstall`

當您的 `npm` 設定了 `ignore-scripts = true` 時可能會發生。或者您使用 `pnpm` 或其他函式庫管理工具不會自動執行 `postinstall`

## 第一個路由

我們將建置第一個路有 `/posts` ，在開始之前我們先在 `app/root.tsx` 加入連結

```tsx
{/* 記得載入 Link */}
<Link to="/posts">Posts</Link>
```

您可以加到任何您喜歡的位置。回到瀏覽器點擊連結應該會看到 404，這是因為我們還沒建立路由。

接著，建立 `app/routes/posts/index.tsx`

```sh
$ mkdir app/routes/posts
$ touch app/routes/posts/index.tsx
```

> 任何時候您看到終端機指令建立檔案或目錄，您當然可以使用您習慣的方式，`mkdir` 和 `touch` 只是我們比較習慣的方式，並且在這裡呈現比較清楚容易。

我們可以將檔案命名為 `posts.tsx`，但很快我們就會有另一個路由，將它們放在一起會比較適合。而且當路徑指到目錄時 `index` 路由會被渲染，就跟伺服器 `index.html` 一樣。

當回傳為 `null` 時您可能會看到空白的頁面。您已經建立了路由但檔案裏面什麼都沒有，讓我們來加入元件 `app/routes/posts/index.tsx`

```tsx
export default function Posts() {
  return (
    <div>
      <h1>Posts</h1>
    </div>
  );
}
```

您可能需要重新載入。

## 載入資料

Remix 內建資料載入。如果您的網頁開發經驗是近幾年才開始的您大概會分成；一個後端 API 路由提供資料然後一個前端專案。在 Remix 一個前端元件同時也是自己的 API 路由，而且它已經知道如何從瀏覽器和伺服器端的自己溝通，因此您不需要自己 `fetch`。

如果您的開發經驗稍微久一點可能熟悉 MVC 框架像是 Rails，那麼您可以將 Remix 路由看作是一個後端的 View 只是樣版採用 React，並且它知道如何無縫在瀏覽器進行 `hydrate` ，而不是寫入一些分離的 jQuery 程式碼來實作互動操作。它充分的展現了漸進式增強，此外您的路由也是它自己的 Controller。

```tsx
import { useLoaderData } from 'remix';

export const loader = () => {
  return [
    {
      slug: 'my-first-post',
      title: 'My First Post'
    },
    {
      slug: '90s-mixape',
      title: 'A Mixtape I Made Just For You'
    }
  ];
};

export default function Posts() {
  const posts = useLoaderData();
  console.log(posts);
  return (
    <div>
      <h1>Posts</h1>
    </div>
  );
}
```

Loaders 就是給元件自身使用的後端 API，而且他們已經通過 `useLoaderData` 連接起來了。在 Remix 路由中前端和伺服器之間有典模糊，這確實有點瘋狂。如果您的伺服器和瀏覽器的 Console 都開啟，您會看到兩者都輸出了文章資料。這是因為 Remix 像傳統的網頁框架一樣是在伺服器端渲染然後傳送完整的 HTML，但它也在客戶端進行 `hydrate` (跟 `render` 一樣但是重複使用了已產生的 HTML Node)。讓我們來加入文章連結

```tsx
import { Link, useLoaderData } from 'remix';

export const loader = () => {
  return [
    {
      slug: 'my-first-post',
      title: 'My First Post'
    },
    {
      slug: '90s-mixape',
      title: 'A Mixtape I Made Just For You'
    }
  ];
};

export default function Posts() {
  const posts = useLoaderData();
  console.log(posts);
  return (
    <div>
      <h1>Posts</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.slug}>
            <Link to={post.slug}>
              {post.title}
            </Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

發現 TypeScript 在警告了，我們來處理一下。加入 `Post` 型別

```tsx
import { Link, useLoaderData } from 'remix';

export type Post = {
  slug: string;
  title: string;
}
export const loader = () => {
  const posts: Post[] = [
    {
      slug: 'my-first-post',
      title: 'My First Post'
    },
    {
      slug: '90s-mixape',
      title: 'A Mixtape I Made Just For You'
    }
  ];
  return posts;
};

export default function Posts() {
  const posts = useLoaderData<Post[]>();
  console.log(posts);
  return (
    <div>
      <h1>Posts</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.slug}>
            <Link to={post.slug}>
              {post.title}
            </Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

好的我們完成了型別的宣告，即使通過網路請求取得也沒問題，因為它們定義在同一份檔案。除非網路在 Remix 取得資料時中斷。現在您同時在元件和 API 得到型別的保護(記得元件已經是自己的 API 路由)

## 重構

建立模組一個穩固的實作就是只處理特定問題。在我們的例子就是讀取和撰寫文章。現在讓我們加入 `getPosts` 並匯出到我們的模組。建立 `app/post.ts`

```sh
$ touch app/post.ts
```

接著我們從路由複製剛剛的程式碼

```ts
export type Post = {
  slug: string;
  title: string;
};

export function getPosts() {
  const posts: Post[] = [
    {
      slug: "my-first-post",
      title: "My First Post"
    },
    {
      slug: "90s-mixtape",
      title: "A Mixtape I Made Just For You"
    }
  ];
  return posts;
}
```

更新 `app/routes/posts/index.tsx` 路由使用我們剛建立的模組

```tsx
import { Link, useLoaderData } from "remix";
import { getPosts } from "~/post";
import type { Post } from "~/post";

export const loader = () => {
  return getPosts();
};

// ...
```

## 讀取資料來源

假如是在真實的專案，我們通常會把資料放在資料庫例如 Postgres，FaunaDB，Supabase 之類的。本篇只是快速入門，因此我們只會是用本機的檔案系統。我們將會讀取本機檔案取代在程式碼寫死資料。

在專案根目錄建立 `posts/`目錄，注意不是 `app` 目錄下。

```sh
$ mkdir posts
```

然後加入一些文章

```sh
$ touch posts/my-first-post.md
$ touch posts/90s-mixtape.md
```

在裡面寫入一些內容，但記得要有 **Front Matter** (Markdown 開頭的設定區塊)

`posts/my-first-post.md`

```md
---
title: My First Post
---

# This is my first post

Isn't it great?
```

`posts/90s-mixtape.md`

```md
---
title: 90s Mixtape
---

# 90s Mixtape

- I wish (Skee-Lo)
- This Is How We Do It (Montell Jordan)
- Everlong (Foo Fighters)
- Ms. Jackson (Outkast)
- Interstate Love Song (Stone Temple Pilots)
- Killing Me Softly With His Song (Fugees, Ms. Lauryn Hill)
- Just a Friend (Biz Markie)
- The Man Who Sold The World (Nirvana)
- Semi-Charmed Life (Third Eye Blind)
- ...Baby One More Time (Britney Spears)
- Better Man (Pearl Jam)
- It's All Coming Back to Me Now (Céline Dion)
- This Kiss (Faith Hill)
- Fly Away (Lenny Kravits)
- Scar Tissue (Red Hot Chili Peppers)
- Santa Monica (Everclear)
- C'mon N' Ride it (Quad City DJ's)
```

更新 `getPosts`，改成讀取檔案，這裡我們需要安裝 `front-matter`

```sh
$ npm add front-matter
```

更新 `app/post.ts`

```ts
import path from 'path';
import fs from 'fs/promises';
import parseFrontMatter from 'front-matter';

export type Post = {
  slug: string;
  title: string;
};

const postsPath = path.join(__dirname, '..', 'posts');

export async function getPosts() {
  const dir = await fs.readdir(postsPath);
  return Promise.all(
    dir.map(async (filename) => {
      const file = await fs.readFile(
        path.join(postsPath, filename)
      );
      const { attributes } = parseFrontMatter(
        file.toString()
      );
      return {
        slug: filename.replace(/\.md$/, ''),
        title: attributes.title
      };
    })
  );
}
```

這不是 Node 檔案系統教學，因此您需要信任這邊的程式碼。如之前提到的，您可以從資料庫或任何地方讀取 Markdown。

> 如果您不是使用 Remix App Server，您的路徑可能需要加入額外的 '..' 。同時注意您不能在不包含儲存空間(系統硬碟)的主機(環境)部署這個範例。

您應該注意到 TypeScript 又警告了，讓我們接著處理。

因為我們讀取了檔案，型別不知道裡面有什麼，因此我們需要在執行時期檢查，為了這個我們需要 `invariant` 方法使其在執行環境檢查。

確保文章有正確的資料

```sh
$ npm add tiny-invariant
```

更新 `app/post.ts`

```tsx
import path from 'path';
import fs from 'fs/promises';
import parseFrontMatter from 'front-matter';
import invariant from 'tiny-invariant';

export type Post = {
  slug: string;
  title: string;
};

export type PostMarkdownAttributes = {
  title: string;
};

const postsPath = path.join(__dirname, '..', 'posts');

function isValidPostAttributes(attributes: any): attributes is PostMarkdownAttributes {
  return attributes?.title;
}

export async function getPosts() {
  const dir = await fs.readdir(postsPath);
  return Promise.all(
    dir.map(async (filename) => {
      const file = await fs.readFile(
        path.join(postsPath, filename)
      );
      const { attributes } = parseFrontMatter(
        file.toString()
      );
      invariant(isValidPostAttributes(attributes), `${filename} has bad meta data!`);
      
      return {
        slug: filename.replace(/\.md$/, ''),
        title: attributes.title
      };
    })
  );
}
```

就算您不是使用 TypeScript 您也可以使用 `invariant` 檢查。

好! 回到介面上我們應該會看到文章列表。您可以多加入一些文章，重新載入看看變化。

## 動態路由參數

現在是時候讓我們的路由可以檢視文章了，我們希望網址如下

```
/posts/my-first-post
/posts/90s-mixtape
```

我們可以使用 "Dynamic segment"，而不是替每一篇文章都建立路由。Remix 會解析並傳給我們，如此我們就可以動態查詢對應的文章。建立動態路由

```sh
touch app/routes/posts/$slug.tsx
```

```tsx
export default function PostSlug() {
  return (
    <div>
      <h1>Some Post</h1>
    </div>
  );
}
```

現在您可以點擊文章連結應該會看到新頁面。

接著加入 Loader 來存取參數

```tsx
import { useLoaderData } from 'remix';

export const loader = async ({ params }) => {
  return params.slug;
};

export default function PostSlug() {
  const slug = useLoaderData();

  return (
    <div>
      <h1>Some Post: {slug}</h1>
    </div>
  );
}
```

我們在檔案名稱前面加上 `$` 會在 `params` 物件加入一個具名的屬性，然後會傳給 Loader，這就是我們查詢文章的方式。

又到了處理 TypeScript 的段落，在 `app/routes/posts/$slug.tsx` 補上

```tsx
import { useLoaderData } from 'remix';
import type { LoaderFunction } from 'remix';

export const loader: LoaderFunction = async ({ params }) => {
  return params.slug;
};
```

現在讓我們來讀取文章內容，加入 `getPost` 函式到文章模組 `app/post.ts`

```tsx
export async function getPost(slug: string) {
  const filepath = path.join(postsPath, slug + '.md');
  const file = await fs.readFile(filepath);
  const { attributes } = parseFrontMatter(file.toString());
  invariant(isValidPostAttributes(attributes), `Post ${filepath} is missing attributes`);
  return {
    slug,
    title: attributes.title
  };
}
```

在 `app/routes/posts/$slug.tsx` 使用 `getPost`

```tsx
import { useLoaderData } from 'remix';
import type { LoaderFunction } from 'remix';
import { getPost } from '~/post';
import invariant from 'tiny-invariant';

export const loader: LoaderFunction = async ({ params }) => {
  invariant(params.slug, 'expected params.slug');
  return getPost(params.slug);
};

export default function PostSlug() {
  const post = useLoaderData();

  return (
    <div>
      <h1>{post.title}</h1>
    </div>
  );
}
```

我們現在已經可以從資料來源讀取文章了。

注意到 `invariant`，因為 `params` 的資料來自網址，我們不能保證 `params.slug` 一定是被我們定義過的，可能您將檔案換成是 `$postId.tsx`。實務上最好使用 `invariant` 檢查，同時它讓 TypeScript 不會警告我們。

網路上有很多 Markdown 的解析器，這篇教學我們將使用 `marked` 因為它比較簡單。

```sh
npm add marked
# 如果使用 TypeScript
npm add @types/marked
```

然後回到 `app/post.ts` 我們要補上文章的內容資料

```ts
import { marked } from 'marked';

export async function getPost(slug: string) {
  const filepath = path.join(postsPath, slug + '.md');
  const file = await fs.readFile(filepath);
  const { attributes, body } = parseFrontMatter(file.toString());
  invariant(isValidPostAttributes(attributes), `Post ${filepath} is missing attributes`);
  const html = marked(body);
  return {
    slug,
    html,
    title: attributes.title
  };
}
```

更新 `app/routes/posts/$slug.tsx`

```tsx
import { useLoaderData } from 'remix';
import type { LoaderFunction } from 'remix';
import { getPost } from '~/post';
import invariant from 'tiny-invariant';

export const loader: LoaderFunction = async ({ params }) => {
  invariant(params.slug, 'expected params.slug');
  return getPost(params.slug);
};

export default function PostSlug() {
  const post = useLoaderData();

  return (
    <div dangerouslySetInnerHTML={{ __html: post.html }} />
  );
}
```

## 新增部落格文章

現在我們部落格的文章需要靠部屬才能新增。雖然這樣的流程還算是一種簡單的方式，不過如果只是打幾個錯字最好還是避免重新部署比較好。這裡的想法就是把文章放到資料庫，這樣我們就需要一個方式來建立文章我們將使用 Actions 來實現。建立管理路由

> 文章後面會說明 Actions

```sh
$ touch app/routes/admin.tsx
```

```tsx
import { Link, useLoaderData } from 'remix';
import { getPosts } from '~/post';
import type { Post } from '~/post';

export const loader = () => {
  return getPosts();
};

export default function Admin() {
  const posts = useLoaderData<Post[]>();
  return (
    <div className="admin">
      <nav>
        <h1>Administrator</h1>
        <ul>
          {posts.map((post) => (
            <li key={post.slug}>
              <Link to={`/posts/${post.slug}`}>
                {post.title}
              </Link>
            </li>
          ))}
        </ul>
      </nav>
      <main>
        
      </main>
    </div>
  );
}
```

您應該認出很多程式碼是來自 `posts` 路由。我們還額外設定了一些 HTML 結構這是因為我們會快速的套用一些樣式

```sh
$ mkdir app/styles
$ touch app/styles/admin.css
```

```css
.admin {
  display: flex;
}

.admin > nav {
  padding-right: 2rem;
}

.admin > main {
  flex: 1;
  border-left: solid 1px #ccc;
  padding-left: 2rem;
}

em {
  color: red;
}
```

把樣式和路由關聯，到 `app/routes/admin.tsx` 加入

```tsx
import adminStyles from '~/styles/admin.css';
// ...
export const links = () => {
  return [{ rel: 'stylesheet', href: adminStyles}];
}
```

每一個路由可以匯出一個 `links` 函式，回傳 `<link>` 的陣列，除了格式是物件而不是 HTML。因此使用 `{ ref: 'stylesheet', href: adminStyles}` 的格式，而不是 `<link rel="stylesheet" href="..." />`。這麼做是為了讓 Remix 合併所有渲染路由的 `links` ，彙整在文件上方的 `<Links />` 元素裡。如果您好奇可以看看 `root.tsx`。

OK，您應該要看到簡單的排版，左邊是文章列表，右邊留白我們還沒實作。現在您需要手動輸入 http://localhost:3000/admin 才能造訪管理頁面，因為我們還沒設定連結

##  Index 路由

讓我們是使用 Index 路由來實作右邊的區塊。現在我們要介紹的是"巢狀路由"即根據路由轉換成巢狀元件結構。

利用建立一個目錄設定 `admin.tsx` 的子路由

```sh
$ mkdir app/routes/admin
$ touch app/routes/admin/index.tsx
```

如果您重新載入，應該看不到任何東西。在 `app/routes/admin/` 目錄下的每一個路由，一旦網址符合都可以渲染到 `app/routes/admin.tsx` 之中。您可以控制要渲染在哪裡。在 `admin.tsx` 加入 `Outlet`

```tsx
import { Outlet, Link, useLoaderData } from 'remix';
import { getPosts } from '~/post';
import type { Post } from '~/post';
import adminStyles from '~/styles/admin.css';

export const loader = () => {
  return getPosts();
};

export const links = () => {
  return [{ rel: 'stylesheet', href: adminStyles}];
}

export default function Admin() {
  const posts = useLoaderData<Post[]>();
  return (
    <div className="admin">
      <nav>
        <h1>Administrator</h1>
        <ul>
          {posts.map((post) => (
            <li key={post.slug}>
              <Link to={`/posts/${post.slug}`}>
                {post.title}
              </Link>
            </li>
          ))}
        </ul>
      </nav>
      <main>
        <Outlet />
      </main>
    </div>
  );
}
```

給我們幾分鐘，剛開始 Index 路由可能會讓您困惑。只要知道當網址上層路由路徑，子路由的 `Index` 就會對應到 `Outlet` 。

讓我們先來實作 `/admin/new` ，也許對您的理解有幫助。建立 `app/routes/admin/new.tsx`

```sh
$ touch app/routes/admin/new.tsx
```

```tsx
export default function NewPost() {
  return (
    <h2>New Post</h2>
  );
}
```

現在點擊 Index 上面的 "Creae a New Post" ，`<Outlet />` 會自動切換到 "new" 路由。

## Actions

然後我們要在這個 "new" 路由建置一個表單來建立文章

```tsx
import { Form } from 'remix';

export default function NewPost() {
  return (
    <>
      <h2>New Post</h2>
      <Form method="post">
        <p>
          <label>
            Post Title:
            <input type="text" name="title" />
          </label>
        </p>
        <p>
          <label>
            Post Slug:
            <input type="text" name="slug" />
          </label>
        </p>
        <p>
          <label htmlFor="markdown">Markdown:</label>
          <br />
          <textarea id="markdown" rows={20} name="markdown" />
        </p>
        <p>
          <button type="submit">Create Post</button>
        </p>
      </Form>
    </>
  );
}
```

要完成新增功能我們需要一個表單從使用者那取得資料，然後一個後端處理收到的資料。在 Remix 您一樣需要完成這些。讓我們先在 `post.ts` 建立儲存資料的功能。在 `app/post.ts` 新增 `createPost` 函式

```ts
export async function createPost(post) {
  const md = `---\ntitle: ${post.title}\n---\n\n${post.markdown}`;
  await fs.writeFile(
    path.join(postsPath, post.slug + '.md'),
    md
  );
  return getPost(post.slug);
}
```

在新增頁面使用 `createPost` ，編輯 `app/routes/admin/new.tsx` 加入 `action`

```tsx
import { Form, redirect } from 'remix';
import { createPost } from '~/post';

export const action = async ({ request }) => {
  const formData = await request.formData();

  const title = formData.get('title');
  const slug = formData.get('slug');
  const markdown = formData.get('markdown');

  await createPost({
    title,
    slug,
    markdown
  });

  return redirect('/admin');
}

export default function NewPost() {
  return (
    <>
      <h2>New Post</h2>
      <Form method="post">
        <p>
          <label>
            Post Title:
            <input type="text" name="title" />
          </label>
        </p>
        <p>
          <label>
            Post Slug:
            <input type="text" name="slug" />
          </label>
        </p>
        <p>
          <label htmlFor="markdown">Markdown:</label>
          <br />
          <textarea id="markdown" rows={20} name="markdown" />
        </p>
        <p>
          <button type="submit">Create Post</button>
        </p>
      </Form>
    </>
  );
}
```

就這樣。Remix 會幫您處理剩下的事情。當點擊 submit 按鈕，會看到側邊欄自動多了一篇文章。

 `formData` 可以使用和`input` 的 `name` 相同值取得 `input` 資料。好的過程中應該又看到一些 TypeScript 的警告。

在 `app/post.ts` 加入型別

```tsx
export type NewPost = {
  title: string;
  slug: string;
  markdown: string;
}

export async function createPost(post: NewPost) {
  const md = `---\ntitle: ${post.title}\n---\n\n${post.markdown}`;
  await fs.writeFile(
    path.join(postsPath, post.slug + '.md'),
    md
  );
  return getPost(post.slug);
}
```

`app/routes/admin/new.tsx` 也需要處理型別的問題

```tsx
import type { ActionFunction } from "remix";

export const action: ActionFunction = async ({ request }) => {
  const formData = await request.formData();

  const title = formData.get('title');
  const slug = formData.get('slug');
  const markdown = formData.get('markdown');

  await createPost({
    title,
    slug,
    markdown
  });

  return redirect('/admin');
}
```

無論是否使用 TypeScript 我們現在還是有個問題，就是當使用者不提供欄位值的時候(TypeScript 依然在警告 `createPost`)。讓我們在建立文章之前加入驗證

```tsx
// app/routes/admin/new.tsx
export const action: ActionFunction = async ({ request }) => {
  const formData = await request.formData();

  const title = formData.get('title');
  const slug = formData.get('slug');
  const markdown = formData.get('markdown');

  const errors = {};
  if (!title) errors.title = true;
  if (!slug) errors.slug = true;
  if (!markdown) errors.markdown = true;

  if (Object.keys(errors).length) {
    return errors;
  }


  await createPost({
    title,
    slug,
    markdown
  });

  return redirect('/admin');
}
```

注意到我們這次沒有使用 `redirect`，我們回傳 `errors`。這些錯誤可以通過 `useActionData` 取得，就類似 `useLoaderData`，但資料來自一個 Form Post 之後的 Action。接著在 `app/routes/admin/new.tsx` 加入驗證資訊的部分

```tsx
import { Form, redirect, useActionData } from 'remix';
import type { ActionFunction } from 'remix';
import { createPost } from '~/post';

//...

export default function NewPost() {
  const errors = useActionData();

  return (
    <>
      <h2>New Post</h2>
      <Form method="post">
        <p>
          <label>
            Post Title:
            {" "}
            {errors?.title ? (
              <em>Title is required</em>
            ) : null}
            <input type="text" name="title" />
          </label>
        </p>
        <p>
          <label>
            Post Slug:
            {" "}
            {errors?.slug ? (
              <em>Slug is required</em>
            ) : null}
            <input type="text" name="slug" />
          </label>
        </p>
        <p>
          <label htmlFor="markdown">Markdown:</label>
          {" "}
          {errors?.markdown ? (
            <em>Markdown is required</em>
          ) : null}
          <br />
          <textarea id="markdown" rows={20} name="markdown" />
        </p>
        <p>
          <button type="submit">Create Post</button>
        </p>
      </Form>
    </>
  );
}
```

TypeScript 依然有警告，讓我們加入 `invariant` 和錯誤資訊的物件型別

```tsx
import { Form, redirect, useActionData } from 'remix';
import type { ActionFunction } from 'remix';
import invariant from 'tiny-invariant';
import { createPost } from '~/post';

type PostError = {
  title?: boolean;
  slug?: boolean;
  markdown?: boolean;
}

export const action: ActionFunction = async ({ request }) => {
  const formData = await request.formData();

  const title = formData.get('title');
  const slug = formData.get('slug');
  const markdown = formData.get('markdown');

  const errors: PostError = {};
  if (!title) errors.title = true;
  if (!slug) errors.slug = true;
  if (!markdown) errors.markdown = true;

  if (Object.keys(errors).length) {
    return errors;
  }

  invariant(typeof title === 'string');
  invariant(typeof slug === 'string');
  invariant(typeof markdown === 'string');
  await createPost({
    title,
    slug,
    markdown
  });

  return redirect('/admin');
}

export default function NewPost() {
  const errors = useActionData();

  return (
    <>
      <h2>New Post</h2>
      <Form method="post">
        <p>
          <label>
            Post Title:
            {" "}
            {errors?.title ? (
              <em>Title is required</em>
            ) : null}
            <input type="text" name="title" />
          </label>
        </p>
        <p>
          <label>
            Post Slug:
            {" "}
            {errors?.slug ? (
              <em>Slug is required</em>
            ) : null}
            <input type="text" name="slug" />
          </label>
        </p>
        <p>
          <label htmlFor="markdown">Markdown:</label>
          {" "}
          {errors?.markdown ? (
            <em>Markdown is required</em>
          ) : null}
          <br />
          <textarea id="markdown" rows={20} name="markdown" />
        </p>
        <p>
          <button type="submit">Create Post</button>
        </p>
      </Form>
    </>
  );
}
```

為了讓事情更有趣，在瀏覽器開發工具停用 JavaScript ，在試試看功能。因為 Remix 是根據 HTML 和 HTTP 基礎功能構成的，就算少了瀏覽器支援 JavaScript 還是可以正常運作，但這不是重點。讓我們放慢腳步為表單加入一些待處理狀態的介面。在 `app/routes/admin/new.tsx` 仿造一些延遲

```tsx
export const action: ActionFunction = async ({ request }) => {
  await new Promise((reslove) => setTimeout(reslove, 1000));

  const formData = await request.formData();

  const title = formData.get('title');
  const slug = formData.get('slug');
  const markdown = formData.get('markdown');

  // ...
}
```

使用 `useTransition` 加入待處理的介面

```tsx
import { Form, redirect, useActionData, useTransition } from 'remix';
import type { ActionFunction } from 'remix';
import invariant from 'tiny-invariant';
import { createPost } from '~/post';

// ...

export default function NewPost() {
  const errors = useActionData();
  const transition = useTransition();

  return (
    <>
      <h2>New Post</h2>
      <Form method="post">
        {/ ... /}
        <p>
          <button type="submit">
            {transition.submission 
              ? 'Creating ...'
              : 'Create Post'
            }
          </button>
        </p>
      </Form>
    </>
  );
}
```

現在如果啟用 JavaScript 的話，使用者體驗好了一些。一些您可以做的優化像是新增時自動將 `title` 轉換填入 `slug`，讓使用者可以自行在調整。

教學大致上到這裡，您可以思考一下如何建置 `/admin/edit` 編輯文章。側邊欄的連結可以使用。建立一個新的路由讀取文章，將資料放入欄位，全部需要的程式碼在 `app/routes/posts/$slug.tsx` 和 `app/routes/admin/new.tsx` 都有。

希望您會喜歡 Remix。

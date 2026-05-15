---
tags:
  - Next.js
  - React
  - TypeScript
created: 2026-05-15
updated: 2026-05-16
---

# Next.js リファレンス

## 目次

1. [インストール・セットアップ](#インストール・セットアップ)
2. [ディレクトリ構成](#ディレクトリ構成)
3. [ページとルーティング](#ページとルーティング)
4. [レイアウト](#レイアウト)
5. [Link コンポーネント](#link-コンポーネント)
6. [Server Components と Client Components](#server-components-と-client-components)
7. [データ取得](#データ取得)
8. [ローディング・エラー UI](#ローディング・エラー-ui)
9. [Route Handlers（API）](#route-handlersapi)
10. [メタデータ](#メタデータ)
11. [next/image](#nextimage)
12. [環境変数](#環境変数)
13. [プロジェクト構成の実践](#プロジェクト構成の実践)
14. [CSS・スタイリング](#cssスタイリング)
15. [フォームと Server Actions](#フォームと-server-actions)
16. [キャッシュと再検証](#キャッシュと再検証)
17. [デプロイ](#デプロイ)
18. [ルーティング応用](#ルーティング応用)
19. [Suspense と部分ローディング](#suspense-と部分ローディング)
20. [Client Component の境界設計](#client-component-の境界設計)
21. [SEO・メタデータ応用](#seoメタデータ応用)
22. [設定ファイル](#設定ファイル)
23. [用語集](#用語集)
24. [リンク集](#リンク集)

---

## インストール・セットアップ

```bash
npx create-next-app@latest my-app
cd my-app
npm run dev
```

対話形式で設定を聞かれる。初心者向けのおすすめ設定：

```
Would you like to use TypeScript? › Yes
Would you like to use ESLint? › Yes
Would you like to use Tailwind CSS? › No（まずは使わない）
Would you like your code inside a `src/` directory? › No
Would you like to use App Router? › Yes
Would you like to use Turbopack for `next dev`? › Yes
Would you like to customize the import alias (@/*)? › No
```

開発サーバーが起動したら `http://localhost:3000` で確認できる。

---

## ディレクトリ構成

```
my-app/
├── app/
│   ├── layout.tsx      ← 全ページ共通のレイアウト
│   ├── page.tsx        ← / （トップページ）
│   └── globals.css
├── public/             ← 画像などの静的ファイル
├── next.config.ts
└── package.json
```

`app/` ディレクトリ内のフォルダ構造がそのままURLになる（App Router）。

---

## ページとルーティング

フォルダを作って `page.tsx` を置くだけでページができる。

```
app/
├── page.tsx          → /
├── about/
│   └── page.tsx      → /about
└── blog/
    ├── page.tsx      → /blog
    └── [slug]/
        └── page.tsx  → /blog/abc, /blog/xyz など
```

**page.tsx の基本形**

```tsx
// app/about/page.tsx
export default function AboutPage() {
  return (
    <main>
      <h1>About</h1>
      <p>このサイトについて</p>
    </main>
  );
}
```

**動的ルート**（URL の一部を変数として受け取る）

フォルダ名を `[slug]` のように `[]` で囲むと動的セグメントになる。`params` として受け取る型は `Promise<{ slug: string }>` になる（Next.js 15 以降）。

```tsx
// app/blog/[slug]/page.tsx
type Props = {
  params: Promise<{ slug: string }>;
};

export default async function BlogPost({ params }: Props) {
  const { slug } = await params;
  return <h1>記事: {slug}</h1>;
}
```

---

## レイアウト

`layout.tsx` はそのフォルダ以下の全ページに適用される共通レイアウト。ナビゲーションやフッターを置く場所。

```tsx
// app/layout.tsx
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "My App",
  description: "Next.js アプリ",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ja">
      <body>
        <header>
          <nav>ナビゲーション</nav>
        </header>
        <main>{children}</main>
        <footer>フッター</footer>
      </body>
    </html>
  );
}
```

`children` が各ページの内容に置き換わる。ネストしたフォルダに `layout.tsx` を置くと、そのフォルダ以下だけに適用されるレイアウトを追加できる。

---

## Link コンポーネント

ページ間の移動には `<a>` タグではなく Next.js の `Link` を使う。ページ全体をリロードせずに遷移する。

```tsx
import Link from "next/link";

export default function Nav() {
  return (
    <nav>
      <Link href="/">ホーム</Link>
      <Link href="/about">About</Link>
      <Link href="/blog">ブログ</Link>
    </nav>
  );
}
```

**コードから遷移する場合**（ボタンクリック後などに遷移する場合）

`useRouter` は Client Component でしか使えないため、`"use client"` が必要。（説明は後述）

```tsx
"use client";

import { useRouter } from "next/navigation";

export default function BackButton() {
  const router = useRouter();

  return (
    <button onClick={() => router.push("/")}>
      ホームへ戻る
    </button>
  );
}
```

---

## Server Components と Client Components

Next.js の App Router では、コンポーネントはデフォルトで **Server Component**（サーバー側で実行）になる。

| | Server Component | Client Component |
|--|--|--|
| デフォルト | Yes（何も書かない） | `"use client"` を先頭に書く |
| 実行場所 | サーバー | ブラウザ |
| `useState` / `useEffect` | 使えない | 使える |
| API キーなどの秘密情報 | 扱える | 扱えない（ブラウザに露出） |
| データ取得（`fetch`、DB） | 直接書ける | `useEffect` などで別途取得 |

**Client Component**

```tsx
"use client";  // ファイルの先頭に書く

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount((c) => c + 1)}>
      カウント: {count}
    </button>
  );
}
```

**判断の目安**

- `useState` / `useEffect` を使う → Client Component
- クリックなどのイベントを扱う → Client Component
- データベースや API から取得して表示するだけ → Server Component でよい

Server Component の中に Client Component を子として含めることができる。Client Component から Server Component を直接 `import` することはできないが、Server Component 側で `children` として渡す形なら組み合わせられる。

---

## データ取得

Server Component では `async/await` を直接使ってデータを取得できる。

```tsx
// app/users/page.tsx（Server Component）
type User = {
  id: number;
  name: string;
  email: string;
};

export default async function UsersPage() {
  const res = await fetch("https://jsonplaceholder.typicode.com/users");
  const users: User[] = await res.json();

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

`fetch` は Next.js によって拡張されており、`cache` オプションでキャッシュ動作を制御できる。

```tsx
// キャッシュしない（毎回取得）
const res = await fetch(url, { cache: "no-store" });

// 指定秒数ごとに再取得（60秒）
const res = await fetch(url, { next: { revalidate: 60 } });
```

---

## ローディング・エラー UI

### loading.tsx

データ取得中に表示するUI。同じフォルダに置くだけで自動で使われる。

```tsx
// app/users/loading.tsx
export default function Loading() {
  return <p>読み込み中...</p>;
}
```

### error.tsx

エラーが起きたときに表示するUI。Client Component にする必要がある。

```tsx
// app/users/error.tsx
"use client";

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <p>エラーが発生しました: {error.message}</p>
      <button onClick={reset}>再試行</button>
    </div>
  );
}
```

### not-found.tsx

存在しないページにアクセスしたときに表示するUI。

```tsx
// app/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h2>404 - ページが見つかりません</h2>
    </div>
  );
}
```

`notFound()` 関数を呼ぶと、そのページの `not-found.tsx` が表示される。

```tsx
import { notFound } from "next/navigation";

export default async function PostPage({ params }: Props) {
  const { slug } = await params;
  const post = await getPost(slug);

  if (!post) notFound();

  return <h1>{post.title}</h1>;
}
```

---

## Route Handlers（API）

`app/api/` 以下に `route.ts` を置くと API エンドポイントを作れる。

```tsx
// app/api/hello/route.ts
import { NextResponse } from "next/server";

export function GET() {
  return NextResponse.json({ message: "Hello!" });
}
```

`GET /api/hello` にアクセスすると `{ "message": "Hello!" }` が返る。

**POST リクエストを受け取る**

```tsx
// app/api/users/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function POST(req: NextRequest) {
  const body: { name: string } = await req.json();
  // 実際はここで DB 保存などを行う
  return NextResponse.json({ created: body.name }, { status: 201 });
}
```

---

## メタデータ

ページの `<title>` や `<meta>` タグを設定する。`layout.tsx` や `page.tsx` から `metadata` をエクスポートする。

**静的なメタデータ**

```tsx
// app/about/page.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "About | My App",
  description: "このサイトについて",
};

export default function AboutPage() {
  return <h1>About</h1>;
}
```

**動的なメタデータ**（URL のパラメータに応じて変える）

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from "next";

type Props = {
  params: Promise<{ slug: string }>;
};

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  return {
    title: `${slug} | ブログ`,
  };
}

export default async function BlogPost({ params }: Props) {
  const { slug } = await params;
  return <h1>{slug}</h1>;
}
```

---

## next/image

`<img>` の代わりに Next.js の `Image` コンポーネントを使うと、自動でサイズ最適化・遅延読み込みが行われる。

```tsx
import Image from "next/image";

export default function Avatar() {
  return (
    <Image
      src="/avatar.png"    // public/ 以下のパス
      alt="アバター"
      width={100}
      height={100}
    />
  );
}
```

`width` と `height` は基本的に指定する（レイアウトのズレ防止のため）。静的インポートした画像では自動で補われ、親要素いっぱいに表示する場合は `fill` も使える。外部URLの画像を使う場合は `next.config.ts` の `images.remotePatterns` に許可するURLパターンを追加する必要がある。

---

## 環境変数

`.env.local` ファイルに書く（`.gitignore` に含まれているのでコミットされない）。

```
# .env.local
DATABASE_URL=postgres://...
NEXT_PUBLIC_API_URL=https://api.example.com
```

**サーバー専用変数**（`NEXT_PUBLIC_` なし）

```tsx
// Server Component や Route Handler から参照できる
const dbUrl = process.env.DATABASE_URL;
```

**ブラウザからも参照できる変数**（`NEXT_PUBLIC_` プレフィックスをつける）

```tsx
// Client Component からも参照できる（ブラウザに公開される）
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

> `NEXT_PUBLIC_` をつけた変数はビルド時にバンドルされてブラウザに公開される。API キーなどの秘密情報には絶対につけない。

---

## プロジェクト構成の実践

Next.js は `app/` のフォルダ構造でルーティングするが、すべてのファイルがページになるわけではない。公開されるのは基本的に `page.tsx` や `route.ts` を置いた場所だけ。

```
my-app/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── _components/       ← app 内で使う非公開コンポーネント
│   ├── (marketing)/       ← URL に出ないグループ
│   │   └── about/
│   │       └── page.tsx   ← /about
│   └── dashboard/
│       ├── page.tsx
│       └── _lib/          ← dashboard 専用の処理
├── components/            ← 複数機能で使う共通 UI
├── lib/                   ← DB, API, utility など
└── public/                ← 画像などの静的ファイル
```

**よく使う置き場所**

| 場所 | 用途 |
|------|------|
| `components/` | アプリ全体で使う共通コンポーネント |
| `lib/` | DB 接続、API クライアント、共通関数 |
| `app/_components/` | `app/` 内だけで使う部品 |
| `app/(group)/` | URL を変えずにルートを整理する |
| `app/feature/_lib/` | 特定ルート専用の処理 |

`_` で始まるフォルダはルーティング対象外にできる。`(marketing)` のような丸括弧のフォルダは URL には出ず、レイアウトや機能のまとまりを作るために使う。

---

## CSS・スタイリング

Next.js では Global CSS、CSS Modules、Tailwind CSS などを使える。まずは全体のスタイルを `globals.css`、部品ごとのスタイルを CSS Modules に分けるとわかりやすい。

**Global CSS**

```tsx
// app/layout.tsx
import "./globals.css";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ja">
      <body>{children}</body>
    </html>
  );
}
```

```css
/* app/globals.css */
body {
  margin: 0;
  font-family: system-ui, sans-serif;
}
```

**CSS Modules**

```css
/* app/components/Button.module.css */
.button {
  padding: 8px 12px;
  border-radius: 6px;
}
```

```tsx
import styles from "./Button.module.css";

export function Button() {
  return <button className={styles.button}>保存</button>;
}
```

Tailwind CSS を使う場合は `className` にユーティリティクラスを書く。小さい学習用アプリなら、CSS Modules から始めると CSS の流れを理解しやすい。

---

## フォームと Server Actions

App Router では、フォーム送信時の処理を Server Action として書ける。簡単な保存処理なら API Route を作らずに実装できる。

```tsx
// app/todos/page.tsx
import { redirect } from "next/navigation";

async function createTodo(formData: FormData) {
  "use server";

  const title = String(formData.get("title") ?? "");

  if (!title.trim()) {
    return;
  }

  // ここで DB 保存などを行う
  redirect("/todos");
}

export default function TodosPage() {
  return (
    <form action={createTodo}>
      <input name="title" placeholder="やること" />
      <button type="submit">追加</button>
    </form>
  );
}
```

`"use server"` をつけた関数はサーバー側で実行される。DB 保存、認証チェック、秘密情報を使う処理は Server Action 側に置く。

入力値は必ず検証する。実用では Zod などのバリデーションライブラリを使うことが多い。

---

## キャッシュと再検証

Next.js では `fetch` のオプションでデータのキャッシュ方法を指定できる。

```tsx
// 常に最新を取得する
const res = await fetch(url, { cache: "no-store" });

// キャッシュを使う
const res = await fetch(url, { cache: "force-cache" });

// 60秒ごとに再検証する
const res = await fetch(url, { next: { revalidate: 60 } });
```

**使い分け**

| 指定 | 用途 |
|------|------|
| `cache: "no-store"` | 毎回変わるデータ、ログインユーザーごとのデータ |
| `cache: "force-cache"` | あまり変わらない公開データ |
| `next: { revalidate: 60 }` | 一定時間ごとに更新したいデータ |

Server Action や Route Handler からキャッシュを更新したい場合は `revalidatePath` や `revalidateTag` を使う。

```tsx
"use server";

import { revalidatePath } from "next/cache";

export async function updatePost() {
  // DB 更新など
  revalidatePath("/posts");
}
```

---

## デプロイ

本番用にビルドして動かすには `build` と `start` を使う。

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

```bash
npm run build
npm run start
```

Vercel にデプロイする場合は、GitHub などのリポジトリを接続すると自動でビルドされる。環境変数は `.env.local` ではなく、Vercel の Project Settings など本番環境側に設定する。

静的ファイルだけで配信したい場合は Static Export もあるが、Server Actions、Route Handlers、サーバー実行が必要な機能には制限がある。

---

## ルーティング応用

動的ルートには複数の書き方がある。

| フォルダ | 例 | params |
|------|------|------|
| `[slug]` | `/blog/hello` | `{ slug: "hello" }` |
| `[...slug]` | `/docs/a/b` | `{ slug: ["a", "b"] }` |
| `[[...slug]]` | `/docs`, `/docs/a/b` | `{ slug?: string[] }` |

```tsx
// app/docs/[...slug]/page.tsx
type Props = {
  params: Promise<{ slug: string[] }>;
};

export default async function DocsPage({ params }: Props) {
  const { slug } = await params;
  return <h1>{slug.join(" / ")}</h1>;
}
```

フォルダ名を括弧で囲むことで Route Group を作成できる。

(public) や (admin) は 整理用のフォルダ名なので、URL には含まれない。

```
app/
├── (public)/
│   └── about/page.tsx      → /about
└── (admin)/
    └── dashboard/page.tsx  → /dashboard
```

---

## Suspense と部分ローディング

`loading.tsx` はそのルートのページ読み込み中に表示される。より細かく一部分だけローディング表示にしたい場合は React の `Suspense` を使う。

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";

async function RecentPosts() {
  const res = await fetch("https://example.com/api/posts");
  const posts: { id: number; title: string }[] = await res.json();

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

export default function DashboardPage() {
  return (
    <main>
      <h1>Dashboard</h1>
      <Suspense fallback={<p>記事を読み込み中...</p>}>
        <RecentPosts />
      </Suspense>
    </main>
  );
}
```

ページ全体の待機表示には `loading.tsx`、画面の一部だけを待たせたいときは `<Suspense>` を使う。

---

## Client Component の境界設計

`"use client"` はファイルの先頭につける。そのファイルから import される子コンポーネントも Client 側の境界に入るため、必要な場所だけ Client Component にする。

**Client Component にするもの**

- `useState` / `useEffect` を使う
- `onClick` などのイベントを使う
- `window`, `localStorage` などブラウザ API を使う
- Context を使う

**Server Component に残すもの**

- DB や外部 API からデータを取得する
- 秘密情報を使う
- ただ表示するだけの UI

Context を提供するコンポーネントはできるだけ深い場所に置くと、Server Component として残せる範囲が広くなる。

React 19 以降では `<ThemeContext.Provider>` ではなく、`<ThemeContext value={...}>` の形で Context の値を提供できる。

```tsx
// app/providers.tsx
"use client";

import { createContext } from "react";

export const ThemeContext = createContext("");

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <ThemeContext value="light">
      {children}
    </ThemeContext>
  );
}
```

```tsx
// app/layout.tsx
import { Providers } from "./providers";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ja">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

---

## SEO・メタデータ応用

`metadata` にはタイトルや説明以外にも、OGP や canonical URL などを設定できる。

```tsx
// app/layout.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  metadataBase: new URL("https://example.com"),
  title: {
    default: "My App",
    template: "%s | My App",
  },
  description: "Next.js の学習用アプリ",
  openGraph: {
    title: "My App",
    description: "Next.js の学習用アプリ",
    url: "https://example.com",
    siteName: "My App",
    locale: "ja_JP",
    type: "website",
  },
};
```

`app/icon.png`、`app/apple-icon.png`、`app/opengraph-image.png` のようなファイルを置くと、アイコンや OGP 画像として扱われる。

---

## 設定ファイル

Next.js でよく触る設定ファイル。

| ファイル | 用途 |
|------|------|
| `next.config.ts` | Next.js の設定 |
| `tsconfig.json` | TypeScript の設定 |
| `eslint.config.mjs` | ESLint の設定 |
| `.env.local` | ローカル環境変数 |
| `next-env.d.ts` | Next.js 用の型定義。基本的に編集しない |

**next.config.ts**

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "images.example.com",
        pathname: "/**",
      },
    ],
  },
};

export default nextConfig;
```

**import alias**

`@/*` を設定している場合、深い相対パスを短く書ける。

```tsx
import { Button } from "@/components/Button";
```

---

## 用語集

| 用語 | 説明 |
|------|------|
| App Router | `app/` ディレクトリを使った Next.js 13 以降のルーティング方式 |
| Server Component | サーバー側で実行されるコンポーネント（デフォルト） |
| Client Component | ブラウザ側で実行されるコンポーネント（`"use client"` が必要） |
| Server Action | フォーム送信やデータ更新などをサーバー側で実行する関数 |
| page.tsx | そのルートのページを定義するファイル |
| layout.tsx | 複数ページで共有するレイアウトを定義するファイル |
| loading.tsx | データ取得中に表示されるローディング UI |
| error.tsx | エラー発生時に表示される UI |
| not-found.tsx | 404 ページの UI |
| 動的ルート | `[slug]` のように URL の一部を変数にするルート |
| Catch-all Route | `[...slug]` のように複数階層の URL をまとめて受け取るルート |
| Route Group | `(group)` のように URL に出さずにルートを整理するフォルダ |
| Route Handler | `route.ts` で作る API エンドポイント |
| Suspense | 非同期 UI の待機中に fallback を表示する React の仕組み |
| SSR | サーバーサイドレンダリング。リクエストのたびにサーバーで HTML を生成する |
| SSG | 静的サイト生成。ビルド時に HTML を生成する |
| ISR | 増分静的再生成。静的生成しつつ、一定時間ごとに再生成する |
| revalidate | キャッシュを無効にして再取得するタイミングを指定すること |
| `revalidatePath` | 指定したパスのキャッシュを再検証する関数 |
| `remotePatterns` | `next/image` で外部画像URLを許可する設定 |
| `next/image` | 画像最適化・遅延読み込みを自動で行う Image コンポーネント |
| `next/link` | ページ遷移に使う Link コンポーネント |
| `next/navigation` | `useRouter`, `usePathname` などのナビゲーション系 Hook |
| `NEXT_PUBLIC_` | ブラウザにも公開する環境変数につけるプレフィックス |

---

## リンク集

### Next.js 公式

| リンク | 用途 |
|------|------|
| [Next.js 公式ドキュメント](https://nextjs.org/docs) | ドキュメントのトップ |
| [App Router](https://nextjs.org/docs/app) | App Router の全機能 |
| [Project Structure](https://nextjs.org/docs/app/getting-started/project-structure) | プロジェクト構成とフォルダ規約 |
| [Routing](https://nextjs.org/docs/app/building-your-application/routing) | ルーティングの基本 |
| [Linking and Navigating](https://nextjs.org/docs/app/getting-started/linking-and-navigating) | Link、プリフェッチ、遷移の仕組み |
| [CSS](https://nextjs.org/docs/app/getting-started/css) | CSS Modules、Tailwind CSS、Global CSS |
| [Data Fetching](https://nextjs.org/docs/app/building-your-application/data-fetching) | データ取得パターン |
| [Caching and Revalidating](https://nextjs.org/docs/app/getting-started/caching-and-revalidating) | キャッシュと再検証 |
| [Rendering](https://nextjs.org/docs/app/building-your-application/rendering) | Server / Client Component |
| [Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components) | Server / Client Component の使い分け |
| [Forms](https://nextjs.org/docs/app/guides/forms) | フォームと Server Actions |
| [Metadata API](https://nextjs.org/docs/app/api-reference/functions/generate-metadata) | title, description の設定 |
| [next/image](https://nextjs.org/docs/app/api-reference/components/image) | Image コンポーネントの props |
| [next/link](https://nextjs.org/docs/app/api-reference/components/link) | Link コンポーネントの props |
| [Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) | API エンドポイントの作り方 |
| [Environment Variables](https://nextjs.org/docs/app/building-your-application/configuring/environment-variables) | 環境変数の設定 |
| [Deploying](https://nextjs.org/docs/app/getting-started/deploying) | 本番ビルドとデプロイ |

### 関連

| リンク | 用途 |
|------|------|
| [React 公式ドキュメント](https://react.dev/) | Next.js の土台となる React |
| [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) | TypeScript の公式ドキュメント |
| [Vercel（Next.js のホスティング）](https://vercel.com/) | Next.js アプリを無料でデプロイできる |

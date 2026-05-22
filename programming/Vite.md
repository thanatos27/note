---
tags:
  - Vite
  - フロントエンド
  - TypeScript
created: 2026-05-21
updated: 2026-05-23
---

# Vite リファレンス

## Vite とは

フロントエンド開発向けの高速なビルドツール。開発時はネイティブ ES モジュールを活用し、アプリ全体の事前バンドルを避けることで起動・HMR が非常に速い。本番ビルドでは、依存関係をまとめて最適化された成果物を生成する。

- 開発サーバーの起動が速い（従来の全体バンドル型より規模の影響を受けにくい）
- HMR（Hot Module Replacement）が高速
- TypeScript・JSX・CSS Modules をほぼ設定なしで扱える
- Rollup/Rolldown 系のプラグインと互換性のあるプラグインシステム

## 目次

1. [インストール・セットアップ](#インストールセットアップ)
2. [開発サーバー](#開発サーバー)
3. [ビルド](#ビルド)
4. [設定ファイル（vite.config.ts）](#設定ファイルviteconfigts)
5. [環境変数](#環境変数)
6. [静的アセット](#静的アセット)
7. [CSS](#css)
8. [パスエイリアス](#パスエイリアス)
9. [デプロイ](#デプロイ)
10. [プラグイン](#プラグイン)
11. [Vitest との連携](#vitest-との連携)
12. [用語集](#用語集)
13. [リンク集](#リンク集)

---

## インストール・セットアップ

### Node.js のバージョン

Vite 8 では Node.js `20.19+` または `22.12+` が必要。古い Node.js では `npm create vite@latest` や `npm run dev` が失敗することがある。

```bash
node -v
```

Node.js のバージョン管理には `fnm`、`nvm`、Volta などを使うと切り替えやすい。

### プロジェクト作成

```bash
npm create vite@latest my-app
```

対話形式でフレームワークとバリアント（TypeScript かどうかなど）を選択できる。テンプレートを直接指定する場合：

```bash
# React + TypeScript
npm create vite@latest my-app -- --template react-ts

# Vue + TypeScript
npm create vite@latest my-app -- --template vue-ts

# Vanilla TypeScript（フレームワークなし）
npm create vite@latest my-app -- --template vanilla-ts
```

主なテンプレート一覧：

| テンプレート | 内容 |
|---|---|
| `vanilla` | JavaScript のみ |
| `vanilla-ts` | TypeScript |
| `react` | React + JavaScript |
| `react-ts` | React + TypeScript |
| `vue` | Vue + JavaScript |
| `vue-ts` | Vue + TypeScript |

### セットアップ手順

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npm run dev
```

### ディレクトリ構成（react-ts の場合）

```
my-app/
├── public/           ← そのまま配信される静的ファイル
├── src/
│   ├── assets/       ← ビルドに取り込まれる画像など
│   ├── App.tsx
│   └── main.tsx      ← index.html から参照される JS 側の起点
├── index.html        ← Vite のエントリーポイント（ここからモジュールグラフを構築）
├── vite.config.ts
├── tsconfig.json
└── package.json
```

Vite は `index.html` をエントリーポイントとして扱う点が webpack などと異なる。

---

## 開発サーバー

```bash
npm run dev
```

デフォルトで `http://localhost:5173` で起動する。ソースを変更すると HMR によりページリロードなしで更新される。

**よく使うオプション（CLI）：**

```bash
npx vite --port 3000   # ポートを変更
npx vite --open        # 起動時にブラウザを開く
npx vite --host        # LAN に公開（モバイル確認など）
```

### 依存関係の事前処理

初回起動時や依存パッケージを追加した直後は、Vite が依存関係を開発用に事前処理するため少し時間がかかることがある。2回目以降はキャッシュされるため速くなる。

---

## ビルド

```bash
npm run build     # dist/ に成果物を生成
npm run preview   # build した成果物をローカルで確認
```

`npm run preview` は本番ビルドの成果物を簡易サーバーで確認するもの。本番環境として使うためのサーバーではない。

**package.json の scripts（デフォルト）：**

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview"
  }
}
```

`tsc -b` は TypeScript の型チェック。Vite 自身は型チェックをせず、トランスパイルのみ行う。

---

## 設定ファイル（vite.config.ts）

プロジェクトルートに置く設定ファイル。`defineConfig` でラップすると型補完が効く。

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
});
```

### よく使う設定項目

```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "node:path";

export default defineConfig({
  plugins: [react()],

  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },

  server: {
    port: 3000,
    open: true,
    proxy: {
      "/api": "http://localhost:8080",  // API プロキシ
    },
  },

  build: {
    outDir: "dist",      // 出力先（デフォルト: "dist"）
    sourcemap: true,     // ソースマップを生成
  },

  base: "/",             // 配信される URL のベースパス

  define: {
    __APP_VERSION__: JSON.stringify("1.0.0"),  // グローバル定数を置換
  },
});
```

### server.proxy

開発時に API サーバーへのリクエストをプロキシする。CORS を回避するために使うことが多い。

```ts
server: {
  proxy: {
    "/api": {
      target: "http://localhost:8080",
      changeOrigin: true,
      rewrite: (path) => path.replace(/^\/api/, ""),
    },
  },
},
```

---

## 環境変数

### .env ファイル

プロジェクトルートに配置する。

| ファイル | 用途 |
|---|---|
| `.env` | 全環境で読み込まれる |
| `.env.local` | ローカルのみ（git 管理外推奨） |
| `.env.development` | `vite` 起動時のみ |
| `.env.development.local` | 開発環境のローカルのみ（git 管理外推奨） |
| `.env.production` | `vite build` 時のみ |
| `.env.production.local` | 本番ビルド用のローカルのみ（git 管理外推奨） |

`.env` ファイルの例：

```
VITE_API_URL=https://api.example.com
VITE_APP_TITLE=My App
```

### クライアントに公開する変数

クライアントコードから `import.meta.env` で読むには `VITE_` プレフィックスが必要。プレフィックスのない変数は、Vite が処理するクライアントコードには公開されない。

`VITE_` 付きの値はブラウザに配信されるため、API キーやパスワードなどの秘密情報を入れてはいけない。また、`.env` から読まれる値は基本的に文字列として扱われる。

```ts
// クライアントコードで読む
const apiUrl = import.meta.env.VITE_API_URL;

// Vite が提供する組み込み変数
import.meta.env.MODE       // "development" | "production" | "test"
import.meta.env.DEV        // 開発時は true
import.meta.env.PROD       // 本番時は true
import.meta.env.BASE_URL   // vite.config の base（デフォルト: "/"）
```

### mode と読み込み優先順位

`npm run dev` は通常 `development`、`npm run build` は通常 `production` mode で動く。mode は CLI から変更できる。

```bash
vite --mode staging
vite build --mode staging
```

mode が `staging` の場合は `.env.staging` や `.env.staging.local` も読み込まれる。後から読み込まれるファイルほど優先されるため、同じ変数名がある場合は mode 専用や `.local` の値が優先される。

### TypeScript での型定義

`src/vite-env.d.ts`（create-vite で自動生成）に追記する。

```ts
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

---

## 静的アセット

### public ディレクトリ

`public/` 以下のファイルはビルド時にそのまま `dist/` にコピーされる。URL は `/` からの絶対パスで参照する。

```
public/
├── favicon.ico
└── robots.txt
```

```html
<!-- HTML 内から参照 -->
<link rel="icon" href="/favicon.ico" />
```

ファイル名にハッシュを付けたくないもの（favicon、robots.txt など）は `public/` に置く。

### src 内のアセット

`src/assets/` などに置いたファイルは `import` で読み込む。ビルド時にハッシュ付きファイル名になり、キャッシュバスティングされる。

```ts
// 画像を URL として取得
import logoUrl from "./assets/logo.png";

// コンポーネントで使う
<img src={logoUrl} alt="ロゴ" />
```

```ts
// ?raw でテキストファイルを文字列として読み込む
import svgContent from "./icon.svg?raw";

// ?url で URL として読み込む（デフォルトと同じ）
import iconUrl from "./icon.svg?url";
```

---

## CSS

### グローバル CSS のインポート

```ts
// main.tsx
import "./index.css";
```

### CSS Modules

ファイル名を `*.module.css` にすると CSS Modules として扱われる。クラス名がスコープされ、コンポーネント間のクラス名衝突を防げる。

```css
/* Button.module.css */
.button {
  background: blue;
  color: white;
}
```

```tsx
import styles from "./Button.module.css";

function Button() {
  return <button className={styles.button}>クリック</button>;
}
```

### CSS プリプロセッサ

パッケージをインストールするだけで使える（設定不要）。

```bash
npm install -D sass        # Sass / SCSS
npm install -D less        # Less
npm install -D stylus      # Stylus
```

```ts
// .scss ファイルをそのまま import できる
import "./style.scss";
```

### PostCSS

`postcss.config.js` をプロジェクトルートに置くと自動で読み込まれる。

```js
// postcss.config.js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

---

## パスエイリアス

TypeScript + Vite でパスエイリアスを使うには、両方に設定が必要。

### vite.config.ts

```ts
import { defineConfig } from "vite";
import path from "node:path";

export default defineConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

### tsconfig.json

```jsonc
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

両方に書くことで、TypeScript の型チェックと Vite のモジュール解決が一致する。

```ts
// エイリアスを使った import
import { Button } from "@/components/Button";
import { useAuth } from "@/hooks/useAuth";
```

---

## デプロイ

### 基本

Vite アプリは `npm run build` で生成された `dist/` を静的ホスティングに配置する。

```bash
npm run build
npm run preview
```

`npm run preview` はローカル確認用。実際の公開には Netlify、Vercel、Cloudflare Pages、GitHub Pages、S3 などの静的ホスティングを使う。

### base

サイトをドメイン直下ではなくサブディレクトリに置く場合は `base` を設定する。たとえば GitHub Pages で `https://user.github.io/my-app/` に配信する場合：

```ts
// vite.config.ts
import { defineConfig } from "vite";

export default defineConfig({
  base: "/my-app/",
});
```

`base` は `import.meta.env.BASE_URL` からも参照できる。画像やリンクのパスを組み立てるときに使う。

### SPA の fallback

React Router などでクライアントサイドルーティングを使う SPA では、`/about` などに直接アクセスしたときも `index.html` を返す設定が必要。ホスティング先によって設定方法が違うため、デプロイ先の SPA fallback / rewrite 設定を確認する。

---

## プラグイン

`vite.config.ts` の `plugins` に追加する。

### @vitejs/plugin-react

React の JSX 変換と Fast Refresh（HMR）を有効にする公式プラグイン。create-vite の React テンプレートでは標準で入る。

```bash
npm install -D @vitejs/plugin-react
```

```ts
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
});
```

### 主要プラグイン

| プラグイン | 用途 |
|---|---|
| `@vitejs/plugin-react` | React の JSX 変換 + Fast Refresh |
| `@vitejs/plugin-vue` | Vue の SFC サポート |
| `vite-plugin-svgr` | SVG を React コンポーネントとして import |
| `vite-tsconfig-paths` | tsconfig の `paths` を自動で Vite に反映 |
| `@vitejs/plugin-legacy` | 古いブラウザ向けのポリフィル生成 |

### vite-tsconfig-paths

tsconfig の `paths` を Vite に手動で書かなくてよくなる。

```bash
npm install -D vite-tsconfig-paths
```

```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
});
```

これを使うと `vite.config.ts` の `resolve.alias` の手動設定が不要になる。

---

## Vitest との連携

Vitest は Vite の設定を共有できるテストフレームワーク。`vite.config.ts` に `test` を追加して設定する。

```bash
npm install -D vitest @vitest/ui jsdom @testing-library/react @testing-library/jest-dom
```

```ts
// vite.config.ts
/// <reference types="vitest/config" />

import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",         // ブラウザ相当の DOM 環境
    globals: true,                // describe / it / expect をグローバルで使う
    setupFiles: "./src/test/setup.ts",
  },
});
```

```ts
// src/test/setup.ts
import "@testing-library/jest-dom";
```

```json
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:run": "vitest run"
  }
}
```

`globals: true` にすると tsconfig にも追加が必要：

```jsonc
{
  "compilerOptions": {
    "types": ["vitest/globals"]
  }
}
```

---

## 用語集

| 用語 | 説明 |
|---|---|
| HMR | Hot Module Replacement。ファイル変更時にページリロードなしでモジュールを差し替える仕組み |
| ESM | ES Modules。`import` / `export` を使うネイティブのモジュール形式 |
| バンドル | 複数ファイルをひとつ（または少数）のファイルにまとめること |
| Rollup | JavaScript のバンドラ。Vite の設定やプラグイン互換性の文脈でよく出てくる |
| Rolldown | Rollup 互換を目指す高速なバンドラ。Vite の新しいバージョンで使われる |
| esbuild | Vite が開発時のトランスパイルに使う高速ビルドツール（Go 製） |
| トランスパイル | TypeScript や JSX をブラウザが解釈できる JavaScript に変換すること |
| public ディレクトリ | ビルド処理をせずにそのまま配信するファイルを置く場所 |
| import.meta.env | Vite が提供する環境変数へのアクセス方法 |
| vite.config.ts | Vite の設定ファイル。プラグイン、エイリアス、サーバー設定などを書く |
| CSS Modules | CSS ファイルのクラス名をコンポーネントスコープに限定する仕組み |

---

## リンク集

### Vite 公式

| リンク | 用途 |
|---|---|
| [Vite 公式サイト](https://vite.dev/) | ドキュメントの入口 |
| [Getting Started](https://vite.dev/guide/) | セットアップ・基本的な使い方 |
| [Features](https://vite.dev/guide/features) | TypeScript、CSS、静的アセット等の機能詳細 |
| [Config Reference](https://vite.dev/config/) | vite.config の全オプション |
| [Env Variables](https://vite.dev/guide/env-and-mode) | .env ファイルと import.meta.env |
| [Plugins](https://vite.dev/plugins/) | 公式プラグイン一覧 |
| [Awesome Vite](https://github.com/vitejs/awesome-vite) | コミュニティプラグイン・テンプレート集 |

### 周辺ツール

| リンク | 用途 |
|---|---|
| [Vitest](https://vitest.dev/) | Vite と統合されたテストフレームワーク |
| [vite-tsconfig-paths](https://github.com/aleclarson/vite-tsconfig-paths) | tsconfig の paths を Vite に自動反映 |
| [vite-plugin-svgr](https://github.com/pd4d10/vite-plugin-svgr) | SVG を React コンポーネントとして扱う |

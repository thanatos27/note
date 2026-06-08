---
tags:
  - Tailwind
  - CSS
  - HTML
created: 2026-06-06
updated: 2026-06-06
---

# Tailwind CSS リファレンス

## Tailwind CSS とは

Tailwind CSS は、あらかじめ用意された小さなクラス（ユーティリティクラス）を HTML に直接書いてスタイルを当てる CSS フレームワーク。`flex`、`p-4`、`text-xl` のように意味が明確なクラスを組み合わせることで、CSS ファイルを別途書かずにスタイリングができる。

独自の CSS を書く量が減り、デザインの一貫性を保ちやすい。テーマや独自ユーティリティなどの設定は CSS ファイル内で定義する。

## 目次

1. [インストール・セットアップ](#インストール・セットアップ)
2. [基本的な書き方](#基本的な書き方)
3. [クラス検出の注意点](#クラス検出の注意点)
4. [レイアウト（Flexbox / Grid）](#レイアウトflexbox--grid)
5. [スペーシング（margin / padding）](#スペーシングmargin--padding)
6. [サイズ（width / height）](#サイズwidth--height)
7. [表示・配置・オーバーフロー](#表示配置オーバーフロー)
8. [テキスト](#テキスト)
9. [色・背景](#色背景)
10. [ボーダー・角丸・影・リング](#ボーダー角丸影リング)
11. [画像・比率](#画像比率)
12. [トランジション・変形・アニメーション](#トランジション変形アニメーション)
13. [フォーム・アクセシビリティ](#フォームアクセシビリティ)
14. [レスポンシブ](#レスポンシブ)
15. [状態バリアント（hover / focus など）](#状態バリアントhover--focus-など)
16. [ダークモード](#ダークモード)
17. [任意の値（Arbitrary Values）](#任意の値arbitrary-values)
18. [カスタマイズ（v4）](#カスタマイズv4)
19. [@apply](#apply)
20. [よく使うパターン](#よく使うパターン)
21. [用語集](#用語集)
22. [リンク集](#リンク集)

---

## インストール・セットアップ

### Vite + React（v4）

```bash
npm install tailwindcss @tailwindcss/vite
```

**vite.config.ts**

```ts
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [tailwindcss()],
});
```

**src/index.css**（エントリポイントの CSS に1行追加するだけ）

```css
@import "tailwindcss";
```

### Next.js（App Router）

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

**postcss.config.mjs**

```js
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

**app/globals.css**

```css
@import "tailwindcss";
```

---

## 基本的な書き方

クラスを HTML 要素に直接付与する。

```html
<div class="flex items-center gap-4 p-6 bg-white rounded-xl shadow">
  <p class="text-lg font-bold text-gray-900">Hello, Tailwind!</p>
</div>
```

React / JSX では `className` を使う。

```tsx
<div className="flex items-center gap-4 p-6 bg-white rounded-xl shadow">
  <p className="text-lg font-bold text-gray-900">Hello, Tailwind!</p>
</div>
```

---

## クラス検出の注意点

Tailwind はソースファイルを自動的にスキャンし、見つかったクラスだけを CSS として生成する。

ただし、クラス名を文字列連結で動的に作ると検出できないことがある。

```tsx
// NG: 完全なクラス名がソース上に存在しない
const className = `bg-${color}-500`;

// OK: 完全なクラス名を用意して選ぶ
const colors = {
  blue: "bg-blue-500 text-white",
  red: "bg-red-500 text-white",
};
```

自動検出されない外部パッケージやテンプレートを含めたい場合は `@source` を使う。

```css
@import "tailwindcss";
@source "../node_modules/@my-company/ui-lib";
```

---

## レイアウト（Flexbox / Grid）

Flexbox は、要素を横または縦に一列で並べるのが得意なレイアウト。ナビゲーション、ボタンの中身、カード内の要素配置などによく使う。

Grid は、行と列を使って二次元に配置するのが得意なレイアウト。カード一覧、ダッシュボード、画像ギャラリーなど、複数列のまとまった配置に向いている。

### Flexbox

| クラス | 説明 |
|---|---|
| `flex` | `display: flex` |
| `inline-flex` | `display: inline-flex` |
| `flex-row` | 横並び（デフォルト） |
| `flex-col` | 縦並び |
| `flex-wrap` | 折り返しあり |
| `items-start` / `items-center` / `items-end` | 交差軸の揃え |
| `justify-start` / `justify-center` / `justify-between` / `justify-end` | 主軸の揃え |
| `gap-4` | 要素間の隙間（1 = 4px） |
| `flex-1` | 残りスペースを均等に埋める |
| `flex-none` | サイズを固定（縮まない） |

```html
<div class="flex items-center justify-between gap-4">
  <span>左</span>
  <span>右</span>
</div>
```

### Grid

| クラス | 説明 |
|---|---|
| `grid` | `display: grid` |
| `grid-cols-3` | 3列 |
| `grid-cols-[repeat(auto-fill,minmax(200px,1fr))]` | 任意の値 |
| `col-span-2` | 2列分占有 |
| `gap-4` | 行列間の隙間 |

```html
<div class="grid grid-cols-3 gap-6">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
```

---

## スペーシング（margin / padding）

数値はデフォルトで `0.25rem` 単位（通常は 1 = 4px）。

| クラス | 説明 |
|---|---|
| `p-4` | padding: 全方向 16px |
| `px-4` | padding: 左右 16px |
| `py-2` | padding: 上下 8px |
| `pt-8` / `pb-8` / `pl-4` / `pr-4` | padding: 各方向 |
| `m-4` | margin: 全方向 16px |
| `mx-auto` | margin: 左右 auto（中央寄せ） |
| `mt-4` / `mb-4` / `ml-4` / `mr-4` | margin: 各方向 |
| `-mt-2` | 負の margin |

よく使うサイズ感の目安：

| 値 | px |
|---|---|
| `1` | 4px |
| `2` | 8px |
| `4` | 16px |
| `6` | 24px |
| `8` | 32px |
| `12` | 48px |
| `16` | 64px |

---

## サイズ（width / height）

| クラス | 説明 |
|---|---|
| `w-full` | width: 100% |
| `w-1/2` | width: 50% |
| `w-64` | width: 256px |
| `w-fit` | width: fit-content |
| `max-w-sm` / `max-w-md` / `max-w-lg` / `max-w-xl` | 最大幅 |
| `max-w-5xl` | 最大幅: 64rem（通常 1024px） |
| `container` | 現在のブレークポイントに応じた最大幅 |
| `h-full` | height: 100% |
| `h-screen` | height: 100vh |
| `h-16` | height: 64px |
| `min-h-screen` | min-height: 100vh |
| `size-8` | width & height: 32px（正方形） |

---

## 表示・配置・オーバーフロー

### Display

| クラス | 説明 |
|---|---|
| `block` / `inline-block` / `inline` | display の基本 |
| `hidden` | 非表示（display: none） |
| `flex` / `grid` | Flexbox / Grid |
| `contents` | 親要素の箱を消して子要素だけを配置 |

### Position

| クラス | 説明 |
|---|---|
| `relative` / `absolute` / `fixed` / `sticky` | position |
| `inset-0` | top/right/bottom/left: 0 |
| `top-0` / `right-0` / `bottom-0` / `left-0` | 各方向の配置 |
| `z-10` / `z-50` | z-index |

### Overflow

| クラス | 説明 |
|---|---|
| `overflow-hidden` | はみ出しを隠す |
| `overflow-auto` | 必要なときだけスクロール |
| `overflow-x-auto` | 横スクロール |
| `overscroll-contain` | スクロール連鎖を抑える |

---

## テキスト

### フォントサイズ

| クラス | サイズ |
|---|---|
| `text-xs` | 12px |
| `text-sm` | 14px |
| `text-base` | 16px |
| `text-lg` | 18px |
| `text-xl` | 20px |
| `text-2xl` | 24px |
| `text-4xl` | 36px |

### フォントウェイト

| クラス | 太さ |
|---|---|
| `font-normal` | 400 |
| `font-medium` | 500 |
| `font-semibold` | 600 |
| `font-bold` | 700 |

### その他

| クラス | 説明 |
|---|---|
| `text-left` / `text-center` / `text-right` | 水平揃え |
| `leading-tight` / `leading-normal` / `leading-loose` | 行間 |
| `tracking-wide` | 字間 |
| `truncate` | はみ出たら … で省略 |
| `line-clamp-2` | 2行で切り詰め |
| `underline` | 下線 |
| `italic` | 斜体 |
| `uppercase` / `lowercase` / `capitalize` | 大文字変換 |

---

## 色・背景

Tailwind のデフォルトパレットは `slate`、`gray`、`zinc`、`red`、`orange`、`amber`、`yellow`、`green`、`teal`、`blue`、`indigo`、`violet`、`purple`、`pink` など。数値は 50〜950。

| クラス | 説明 |
|---|---|
| `text-gray-900` | 文字色 |
| `text-blue-500` | 文字色（青） |
| `text-current` | 親の currentColor を使う |
| `text-inherit` | 親の文字色を継承 |
| `bg-white` | 背景白 |
| `bg-gray-100` | 背景薄グレー |
| `bg-blue-600` | 背景青 |
| `bg-blue-600/50` | 背景青 + 透明度 50% |
| `bg-transparent` | 背景透明 |
| `opacity-50` | 透明度 50% |

---

## ボーダー・角丸・影・リング

### ボーダー

| クラス | 説明 |
|---|---|
| `border` | 1px solid |
| `border-2` | 2px solid |
| `border-gray-300` | ボーダー色 |
| `border-t` / `border-b` / `border-l` / `border-r` | 各辺のみ |
| `divide-y` | 子要素間に水平ボーダー |

### 角丸

| クラス | 説明 |
|---|---|
| `rounded` | 4px |
| `rounded-md` | 6px |
| `rounded-lg` | 8px |
| `rounded-xl` | 12px |
| `rounded-2xl` | 16px |
| `rounded-full` | 円形 |

### 影

| クラス | 説明 |
|---|---|
| `shadow-sm` | 小さな影 |
| `shadow` | 標準的な影 |
| `shadow-md` | 中程度の影 |
| `shadow-lg` | 大きな影 |
| `shadow-none` | 影なし |

### リング・アウトライン

| クラス | 説明 |
|---|---|
| `ring` / `ring-2` | box-shadow ベースのリング |
| `ring-blue-500` | リング色 |
| `ring-offset-2` | リングと要素の間隔 |
| `outline-none` | outline を消す |
| `outline` / `outline-2` | outline を付ける |

---

## 画像・比率

| クラス | 説明 |
|---|---|
| `object-cover` | 縦横比を保って領域を埋める |
| `object-contain` | 縦横比を保って全体を収める |
| `object-center` | object-position: center |
| `aspect-square` | 1:1 |
| `aspect-video` | 16:9 |
| `aspect-[4/3]` | 任意の比率 |

```html
<img class="aspect-video w-full object-cover rounded-lg" src="/image.jpg" alt="" />
```

---

## トランジション・変形・アニメーション

| クラス | 説明 |
|---|---|
| `transition` | よく使うプロパティを遷移 |
| `transition-colors` | 色だけを遷移 |
| `duration-200` | 200ms |
| `ease-out` | イージング |
| `scale-105` | 拡大 |
| `rotate-3` | 回転 |
| `translate-x-2` | X方向移動 |
| `animate-spin` | 回転アニメーション |
| `animate-pulse` | 点滅に近いアニメーション |

```html
<button class="transition-colors duration-200 hover:bg-blue-700">
  送信
</button>
```

---

## フォーム・アクセシビリティ

| クラス | 説明 |
|---|---|
| `cursor-pointer` | ポインターカーソル |
| `pointer-events-none` | ポインターイベント無効 |
| `select-none` | テキスト選択を無効化 |
| `accent-blue-600` | checkbox / radio などのアクセント色 |
| `placeholder-gray-400` | placeholder の色 |
| `caret-blue-600` | 入力カーソル色 |
| `sr-only` | スクリーンリーダーには読ませて視覚的には隠す |
| `not-sr-only` | `sr-only` を解除 |

フォーカスリングはアクセシビリティ上重要なので、`focus-visible:` と組み合わせることが多い。

```html
<button class="focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500">
  保存
</button>
```

---

## レスポンシブ

ブレークポイントはモバイルファーストで動く（指定したサイズ以上に適用）。

| プレフィックス | 幅 |
|---|---|
| `sm:` | 640px〜 |
| `md:` | 768px〜 |
| `lg:` | 1024px〜 |
| `xl:` | 1280px〜 |
| `2xl:` | 1536px〜 |

```html
<!-- モバイル: 縦並び、md以上: 横並び -->
<div class="flex flex-col md:flex-row gap-4">
  <div>A</div>
  <div>B</div>
</div>
```

```html
<!-- モバイル: 1列、lg以上: 3列 -->
<div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
  ...
</div>
```

特定サイズ未満にだけ当てたい場合は `max-*` バリアントを使う。

```html
<div class="grid grid-cols-3 max-md:grid-cols-1">
  ...
</div>
```

親コンテナの幅に応じて切り替えたい場合は Container Queries を使う。

```html
<div class="@container">
  <div class="grid grid-cols-1 @md:grid-cols-2 @lg:grid-cols-3">
    ...
  </div>
</div>
```

---

## 状態バリアント（hover / focus など）

`バリアント:クラス` の形式で書く。

| バリアント | タイミング |
|---|---|
| `hover:` | マウスオーバー |
| `focus:` | フォーカス時 |
| `focus-visible:` | キーボードフォーカス時 |
| `active:` | クリック中 |
| `disabled:` | 無効状態 |
| `checked:` | チェック済み |
| `first:` / `last:` | 最初 / 最後の要素 |
| `odd:` / `even:` | 奇数 / 偶数番目 |
| `aria-expanded:` | ARIA 属性の状態 |
| `data-[state=open]:` | data 属性の状態 |
| `has-checked:` | 子孫の状態に応じて適用 |
| `group-hover:` | 親の `.group` がホバーされたとき |
| `peer-focus:` | 隣接する `.peer` がフォーカスされたとき |

```html
<button class="bg-blue-600 hover:bg-blue-700 active:bg-blue-800 text-white px-4 py-2 rounded-lg">
  クリック
</button>
```

```html
<!-- group-hover の例 -->
<div class="group flex items-center gap-2">
  <span>テキスト</span>
  <span class="opacity-0 group-hover:opacity-100">→</span>
</div>
```

---

## ダークモード

CSS の `@media (prefers-color-scheme: dark)` に対応。`dark:` プレフィックスで指定。

```html
<div class="bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100">
  ダークモード対応
</div>
```

手動切り替えを使う場合（v4 での設定例）：

```css
/* index.css */
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

```html
<!-- html 要素に dark クラスを付与 -->
<html class="dark">
```

---

## 任意の値（Arbitrary Values）

`[値]` で任意の CSS 値を直接指定できる。

```html
<div class="w-[320px] h-[calc(100vh-80px)] bg-[#1a2b3c]">
  任意の値
</div>

<p class="text-[13px] leading-[1.8]">細かい調整</p>

<div class="grid grid-cols-[1fr_2fr_1fr]">...</div>
```

CSS 変数も使える：

```html
<div class="bg-[var(--brand-color)]">...</div>
```

---

## カスタマイズ（v4）

v4 では CSS ファイル内で `@theme` を使ってデザイントークンを定義する。

```css
@import "tailwindcss";

@theme {
  --color-brand: #3b82f6;
  --color-brand-dark: #2563eb;

  --font-sans: "Inter", sans-serif;

  --spacing-18: 4.5rem;
}
```

定義したトークンはそのままクラスとして使える：

```html
<div class="bg-brand text-white font-sans p-18">...</div>
```

よく使う v4 ディレクティブ：

| ディレクティブ | 用途 |
|---|---|
| `@theme` | 色・フォント・間隔などのデザイントークンを定義 |
| `@utility` | 独自ユーティリティを定義 |
| `@custom-variant` | 独自バリアントを定義 |
| `@source` | クラス検出対象を追加 |
| `@reference` | CSS Modules や Vue / Svelte の style からテーマを参照 |

```css
@utility content-auto {
  content-visibility: auto;
}
```

---

## @apply

`@apply` で既存のユーティリティクラスを CSS にまとめられる。使いすぎるとユーティリティの恩恵が薄れるため、ボタンなど繰り返しが多い要素に限定するのが一般的。

```css
.btn-primary {
  @apply inline-flex items-center px-4 py-2 rounded-lg bg-blue-600 text-white font-medium hover:bg-blue-700;
}
```

---

## よく使うパターン

### カード

```html
<div class="bg-white rounded-xl shadow-md p-6 flex flex-col gap-4">
  <h2 class="text-xl font-bold text-gray-900">カードタイトル</h2>
  <p class="text-gray-600 text-sm">説明テキスト</p>
</div>
```

### ボタン

```html
<!-- プライマリ -->
<button class="bg-blue-600 hover:bg-blue-700 text-white font-medium px-5 py-2 rounded-lg transition-colors">
  送信
</button>

<!-- アウトライン -->
<button class="border border-gray-300 hover:bg-gray-50 text-gray-700 font-medium px-5 py-2 rounded-lg transition-colors">
  キャンセル
</button>
```

### 入力フォーム

```html
<input
  type="text"
  class="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
  placeholder="入力してください"
/>
```

### ナビゲーションバー

```html
<nav class="flex items-center justify-between px-6 py-4 bg-white border-b border-gray-200">
  <span class="text-lg font-bold">ロゴ</span>
  <ul class="flex items-center gap-6 text-sm text-gray-600">
    <li><a href="#" class="hover:text-gray-900">ホーム</a></li>
    <li><a href="#" class="hover:text-gray-900">About</a></li>
  </ul>
</nav>
```

### 中央配置（画面いっぱい）

```html
<div class="min-h-screen flex items-center justify-center bg-gray-50">
  <div class="w-full max-w-md bg-white rounded-xl shadow p-8">
    コンテンツ
  </div>
</div>
```

---

## 用語集

| 用語 | 説明 |
|---|---|
| ユーティリティクラス | 単一の CSS プロパティに対応する小さなクラス |
| ブレークポイント | レスポンシブのサイズ切り替え点 |
| バリアント | `hover:`、`focus:` など条件付きでスタイルを当てる仕組み |
| デザイントークン | カラーやスペーシングなどの共通値（v4 では `@theme` で定義） |
| JIT (Just-In-Time) | 使ったクラスだけ CSS を生成する仕組み |
| Arbitrary Values | `[値]` で任意の CSS 値を直接指定できる機能 |
| Container Queries | 画面幅ではなく親コンテナの幅でスタイルを切り替える仕組み |
| `@theme` | v4 でデザイントークンを CSS 内に定義するディレクティブ |

---

## リンク集

- [Tailwind CSS 公式ドキュメント](https://tailwindcss.com/docs)
- [Tailwind CSS v4 アップグレードガイド](https://tailwindcss.com/docs/upgrade-guide)
- [Tailwind UI（公式コンポーネント集）](https://tailwindui.com)
- [Tailwind Play（ブラウザ上で試せる Playground）](https://play.tailwindcss.com)

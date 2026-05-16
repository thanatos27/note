---
tags:
  - React
  - TypeScript
  - TSX
created: 2026-05-14
updated: 2026-05-17
---

# React リファレンス

## Reactとは

Reactは、ユーザーインターフェースを作るためのJavaScriptライブラリ。画面を小さな部品であるコンポーネントに分け、データの変化に合わせて必要な部分を効率よく再描画できる。

Reactでは、Propsで親から子へデータを渡し、Stateでコンポーネント内の変化する値を管理する。TSXを使うと、TypeScriptの型安全性を活かしながらHTMLに近い形でUIを書けるため、複雑な画面でも構造を追いやすくなる。

Next.js、Remix、React Routerなど多くのフレームワークやライブラリの土台にもなっており、単体の画面部品から大規模なWebアプリまで幅広く使われている。

## 目次

1. [インストール・セットアップ](#インストール・セットアップ)
2. [コンポーネント](#コンポーネント)
3. [TSX](#TSX)
4. [Props](#Props)
5. [State（useState）](#State（useState）)
6. [イベントハンドラ](#イベントハンドラ)
7. [条件付きレンダリング](#条件付きレンダリング)
8. [リスト](#リスト)
9. [フォーム](#フォーム)
10. [スタイル](#スタイル)
11. [useEffect](#useEffect)
12. [カスタムフック](#カスタムフック)
13. [main.tsx / createRoot](#maintsx--createroot)
14. [Rules of Hooks](#rules-of-hooks)
15. [React Developer Tools](#React-Developer-Tools)
16. [用語集](#用語集)
17. [リンク集](#リンク集)

---

## インストール・セットアップ

学習用・クライアント中心の小規模アプリでは Vite が定番。TypeScript を使う場合は `react-ts` テンプレートを選ぶ。公式は本格的な新規アプリではフレームワーク利用も推奨している。Create React App は非推奨。

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npm run dev
```

生成されるディレクトリ構成：

```
my-app/
├── public/
├── src/
│   ├── assets/
│   ├── App.tsx       ← メインコンポーネント
│   └── main.tsx      ← エントリーポイント
├── index.html
└── package.json
```

---

## コンポーネント

UI を再利用可能なパーツに分割したもの。関数として定義する（関数コンポーネント）。

```tsx
// 定義
function Greeting() {
  return <h1>Hello, World!</h1>;
}

// 使用
function App() {
  return (
    <div>
      <Greeting />
      <Greeting />
    </div>
  );
}
```

**ルール**
- コンポーネント名は **大文字** で始める（`greeting` は HTML タグとして扱われる）
- return の中で複数の TSX 要素を並べる場合は、親要素または Fragment（`<>...</>`）で囲む

```tsx
// Fragment でラップ
function App() {
  return (
    <>
      <Header />
      <Main />
      <Footer />
    </>
  );
}
```

---

## TSX

TypeScript の中に HTML に似た構文を書ける記法。ブラウザで動く前に JavaScript に変換される。JSX に TypeScript の型チェックを足したものと考えるとわかりやすい。

```tsx
const name: string = "Taro";
const element = <h1>Hello, {name}!</h1>;  // {} で JS 式を埋め込む
```

**HTML との主な違い**

| HTML | TSX |
|------|-----|
| `class` | `className` |
| `for` | `htmlFor` |
| `onclick` | `onClick` |
| 子要素のないタグは閉じなくてもよい | 子要素のないタグも必ず閉じる（`<img />`） |
| スタイルは文字列 | スタイルはオブジェクト |

```tsx
// className, camelCase のイベント名
<button className="btn" onClick={handleClick}>クリック</button>

// 自己終了タグ
<input type="text" />
<img src={url} alt="説明" />
```

---

## Props

親コンポーネントから子コンポーネントへデータを渡す仕組み。**読み取り専用**。

```tsx
// 親
function App() {
  return <UserCard name="Taro" age={25} />;
}

type UserCardProps = {
  name: string;
  age: number;
};

// 子（props を受け取る）
function UserCard({ name, age }: UserCardProps) {
  return (
    <div>
      <p>名前: {name}</p>
      <p>年齢: {age}</p>
    </div>
  );
}
```

props の型は、別で `type` を作らずに引数へ直接書くこともできる。

```tsx
function UserCard({ name, age }: { name: string; age: number }) {
  return (
    <div>
      <p>名前: {name}</p>
      <p>年齢: {age}</p>
    </div>
  );
}
```

**デフォルト値**

```tsx
type ButtonProps = {
  label?: string;
  color?: string;
};

function Button({ label = "送信", color = "blue" }: ButtonProps) {
  return <button style={{ color }}>{label}</button>;
}
```

**children**（コンポーネントのタグの間の内容）

```tsx
import type { ReactNode } from "react";

type CardProps = {
  children: ReactNode;
};

function Card({ children }: CardProps) {
  return <div className="card">{children}</div>;
}

// 使用
<Card>
  <h2>タイトル</h2>
  <p>本文テキスト</p>
</Card>
```

---

## State（useState）

コンポーネント内で変化するデータを管理する。state が変わると再レンダリングが起きる。

```tsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);  // [現在値, 更新関数]

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+1</button>
      <button onClick={() => setCount(0)}>リセット</button>
    </div>
  );
}
```

**オブジェクトの state**（直接変更せず、新しいオブジェクトを渡す）

```tsx
type User = {
  name: string;
  age: number;
};

const [user, setUser] = useState<User>({ name: "Ken", age: 25 });

// NG: 直接変更しない
// user.name = "Taro";

// OK: スプレッド構文で新しいオブジェクトを作る
setUser({ ...user, name: "Taro" });
```

**配列の state**

```tsx
const [items, setItems] = useState<string[]>(["apple", "banana"]);

// 追加
setItems([...items, "cherry"]);

// 削除
setItems(items.filter(item => item !== "banana"));

// 更新
setItems(items.map(item => item === "apple" ? "APPLE" : item));
```

**state 更新の注意点**

- state は直接変更せず、必ず更新関数を使う
- 前の state に依存する更新は、関数形式で書く
- `setState` 直後に state 変数を読んでも、まだ古い値のままの場合がある

```tsx
// NG: 前の count に依存する更新を連続で書くと意図通りにならないことがある
setCount(count + 1);
setCount(count + 1);

// OK: 前の値を受け取って更新する
setCount((c) => c + 1);
setCount((c) => c + 1);
```

---

## イベントハンドラ

ユーザー操作（クリック、入力など）に反応する関数。

イベントを受け取る関数は `handleClick`、`handleChange`、`handleSubmit` のように `handle + イベント名` で書くことが多い。props として渡す場合は `onClick`、`onSubmit` のように `on + イベント名` にすると React の標準イベント名と揃って読みやすい。

```tsx
import type { ChangeEvent } from "react";

function App() {
  function handleClick() {
    alert("クリックされました");
  }

  function handleChange(e: ChangeEvent<HTMLInputElement>) {
    console.log(e.target.value);  // 入力値
  }

  return (
    <>
      <button onClick={handleClick}>クリック</button>
      <input onChange={handleChange} />
    </>
  );
}
```

**引数を渡す場合**

```tsx
// アロー関数でラップする
<button onClick={() => handleDelete(item.id)}>削除</button>
```

**よく使うイベント**

| イベント | 発生タイミング |
|---------|--------------|
| `onClick` | クリック |
| `onChange` | 入力値の変化 |
| `onSubmit` | フォーム送信 |
| `onMouseOver` | マウスオーバー |
| `onKeyDown` | キー押下 |

---

## 条件付きレンダリング

```tsx
type AlertProps = {
  isError: boolean;
};

function Alert({ isError }: AlertProps) {
  // if 文
  if (isError) {
    return <p style={{ color: "red" }}>エラーが発生しました</p>;
  }
  return <p>正常です</p>;
}
```

**三項演算子**（TSX の中に書きたい場合）

```tsx
<div>
  {isLoggedIn ? <p>ようこそ</p> : <p>ログインしてください</p>}
</div>
```

**&& 演算子**（表示/非表示の切り替え）

```tsx
<div>
  {isLoading && <p>読み込み中...</p>}
  {error && <p className="error">{error.message}</p>}
</div>
```

---

## リスト

配列を `.map()` で TSX の配列に変換する。各要素に一意の `key` が必要。

```tsx
const fruits: string[] = ["apple", "banana", "cherry"];

function FruitList() {
  return (
    <ul>
      {fruits.map((fruit) => (
        <li key={fruit}>{fruit}</li>
      ))}
    </ul>
  );
}
```

**オブジェクトの配列**（`key` には ID など一意の値を使う）

```tsx
type User = {
  id: number;
  name: string;
};

const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
] satisfies User[];

function UserList() {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

> `key` に配列のインデックスを使うのは、順序が変わらない静的リスト以外では避ける。

---

## フォーム

input の値を state で管理する（制御コンポーネント）。

```tsx
import { useState, type FormEvent } from "react";

function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();  // ページリロードを防ぐ
    console.log({ email, password });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="メールアドレス"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="パスワード"
      />
      <button type="submit">ログイン</button>
    </form>
  );
}
```

---

## スタイル

**1. CSS ファイルを import**

```tsx
import "./Button.css";

function Button() {
  return <button className="btn">クリック</button>;
}
```

**2. インラインスタイル**（オブジェクトで渡す、プロパティ名は camelCase）

```tsx
import type { CSSProperties } from "react";

const style: CSSProperties = {
  color: "white",
  backgroundColor: "blue",  // background-color ではなく camelCase
  fontSize: 16,              // px は省略可
};

<button style={style}>クリック</button>
```

**3. CSS Modules**（クラス名の衝突を防ぐ）

```tsx
// Button.module.css を作成
import styles from "./Button.module.css";

<button className={styles.btn}>クリック</button>
```

**条件でクラスを切り替える**

```tsx
<button className={isActive ? "btn active" : "btn"}>
  クリック
</button>
```

---

## useEffect

外部システムと同期するための Hook。API 呼び出し、タイマー、ブラウザ API など、レンダリングだけでは完結しない処理を扱う。

```tsx
import { useState, useEffect } from "react";

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setSeconds((s) => s + 1);
    }, 1000);

    return () => clearInterval(id);  // クリーンアップ（コンポーネント削除時に実行）
  }, []);  // [] = マウント時に1回だけ実行

  return <p>{seconds} 秒</p>;
}
```

**依存配列のパターン**

```tsx
useEffect(() => { /* 毎レンダリング後に実行 */ });
useEffect(() => { /* マウント時に1回だけ */ }, []);
useEffect(() => { /* count が変わるたびに実行 */ }, [count]);
```

**API データの取得**

```tsx
type User = {
  name: string;
};

type UserProfileProps = {
  userId: number;
};

function UserProfile({ userId }: UserProfileProps) {
  const [user, setUser] = useState<User | null>(null);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let ignore = false;
    setUser(null);
    setError(null);

    fetch(`https://api.example.com/users/${userId}`)
      .then((res) => res.json())
      .then((data: User) => {
        if (!ignore) setUser(data);
      })
      .catch((error: unknown) => {
        if (!ignore) {
          setError(error instanceof Error ? error : new Error("Unknown error"));
        }
      });

    return () => {
      ignore = true;
    };
  }, [userId]);  // userId が変わったら再取得

  if (error) return <p>エラーが発生しました</p>;
  if (!user) return <p>読み込み中...</p>;
  return <p>{user.name}</p>;
}
```

`userId` が素早く変わると、古いリクエストの結果が後から返ってくることがある。クリーンアップで `ignore` を `true` にしておくと、古い結果で state を更新するのを防げる。

開発時に `StrictMode` を使っていると、Effect が2回実行されたように見えることがある。クリーンアップ処理が正しく書けているか確認するための挙動。

---

## カスタムフック

ロジックを再利用するために自分で作る Hook。名前は `use` で始める。

コンポーネントから state 管理や副作用などの処理だけを切り出したもの。UI は返さず、コンポーネント側で使う値や関数を返すことが多い。

```tsx
// useCounter.ts
import { useState } from "react";

function useCounter(initialValue: number = 0) {
  const [count, setCount] = useState(initialValue);

  return {
    count,
    increment: () => setCount((c) => c + 1),
    decrement: () => setCount((c) => c - 1),
    reset: () => setCount(initialValue),
  };
}

export default useCounter;
```

```tsx
// 使用
import useCounter from "./useCounter";

function App() {
  const { count, increment, decrement, reset } = useCounter(10);

  return (
    <>
      <p>{count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>リセット</button>
    </>
  );
}
```

---

## main.tsx / createRoot

Vite で作成した React + TypeScript アプリでは、`main.tsx` がアプリの入口になる。HTML 側の `#root` に React アプリを描画する。

```tsx
// src/main.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.tsx";
import "./index.css";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```html
<!-- index.html -->
<div id="root"></div>
```

- `createRoot(...)`: React が描画するためのルートを作る
- `render(<App />)`: ルートに `App` コンポーネントを描画する
- `StrictMode`: 開発時に問題を見つけやすくするための仕組み。本番の表示には影響しない

---

## Rules of Hooks

Hook は React が呼び出し順序を使って state などを管理しているため、呼び方にルールがある。

**OK**

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
}
```

**NG**

```tsx
type CounterProps = {
  enabled: boolean;
};

function Counter({ enabled }: CounterProps) {
  if (enabled) {
    const [count, setCount] = useState(0);  // 条件分岐の中で Hook を呼ばない
  }
}
```

**ルール**

- Hook はコンポーネントまたはカスタムフックのトップレベルで呼ぶ
- `if`、`for`、イベントハンドラ、普通の関数の中で Hook を呼ばない
- カスタムフックの名前は `use` で始める

条件によって処理を変えたい場合は、Hook を呼んだ後で条件分岐する。

```tsx
type CounterProps = {
  enabled: boolean;
};

function Counter({ enabled }: CounterProps) {
  const [count, setCount] = useState(0);

  if (!enabled) return null;

  return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
}
```

---

## React Developer Tools

ブラウザの拡張機能。コンポーネントツリーの確認、props / state のリアルタイム確認ができる。

- [Chrome 拡張](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
- [Firefox 拡張](https://addons.mozilla.org/ja/firefox/addon/react-devtools/)

**使い方**
1. DevTools を開く（F12）
2. `Components` タブ → コンポーネントツリーと props/state を確認
3. `Profiler` タブ → レンダリングのパフォーマンスを計測

---

## 用語集

| 用語 | 説明 |
|------|------|
| コンポーネント | UI の再利用可能なパーツ |
| TSX | TypeScript の中に HTML 風の構文を書ける記法 |
| 型注釈 | 変数・引数・戻り値などに型を書くこと |
| Props の型 | コンポーネントが受け取る props の形を `type` や `interface` で定義したもの |
| Props | 親から子へ渡すデータ（読み取り専用） |
| State | コンポーネント内で管理する動的データ |
| Hook | 関数コンポーネントで State などの機能を使うための関数（use〜） |
| レンダリング | コンポーネントが画面に描画されること |
| マウント | コンポーネントが初めて DOM に追加されること |
| アンマウント | コンポーネントが DOM から削除されること |
| 再レンダリング | State や Props の変化でコンポーネントが再描画されること |
| 副作用 | レンダリング以外の処理（API 通信、タイマーなど） |
| 依存配列 | `useEffect` などで、どの値が変わったときに処理を再実行するかを指定する配列 |
| createRoot | React アプリを描画するためのルートを作る API |
| StrictMode | 開発時に React の問題を見つけやすくするための仕組み |
| 制御コンポーネント | input の値を State で管理するフォームの実装パターン |

---

## リンク集

### React 公式

| リンク | 用途 |
|------|------|
| [React 公式ドキュメント](https://react.dev/) | React 公式サイトの入口 |
| [Learn React](https://react.dev/learn) | React の基本を体系的に学ぶ |
| [React Reference](https://react.dev/reference/react) | Hooks、コンポーネント、API の詳細リファレンス |
| [Built-in React Hooks](https://react.dev/reference/react/hooks) | `useState`, `useEffect`, `useRef`, `useMemo` など Hook 一覧 |
| [useEffect](https://react.dev/reference/react/useEffect) | Effect の正しい使い方、依存配列、クリーンアップ |
| [Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks) | Hook を呼ぶ位置のルール |
| [React Blog](https://react.dev/blog) | 新機能、リリース、公式方針の確認 |

### 日本語で読む

| リンク | 用途 |
|------|------|
| [React 公式ドキュメント（日本語）](https://ja.react.dev/learn) | 日本語版の Learn React |
| [React Reference（日本語）](https://ja.react.dev/reference/react) | 日本語版の API リファレンス |
| [React Hooks（日本語）](https://ja.react.dev/reference/react/hooks) | 日本語版の Hooks 一覧 |

### TypeScript と React

| リンク | 用途 |
|------|------|
| [Using TypeScript](https://react.dev/learn/typescript) | React 公式の TypeScript ガイド |
| [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/) | React + TypeScript の実践的な型付け例 |
| [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) | TypeScript 本体の公式ハンドブック |
| [TSConfig Reference](https://www.typescriptlang.org/tsconfig/) | `jsx`, `types`, `moduleResolution` など設定確認 |

### セットアップ・ビルド

| リンク | 用途 |
|------|------|
| [Vite Guide](https://vite.dev/guide/) | Vite の基本セットアップ |
| [Vite TypeScript](https://vite.dev/guide/features.html#typescript) | Vite で TypeScript を使うときの仕様 |
| [create-vite](https://github.com/vitejs/vite/tree/main/packages/create-vite) | `npm create vite@latest` のテンプレート確認 |

### ルーティング・フレームワーク

| リンク | 用途 |
|------|------|
| [React Router](https://reactrouter.com/) | React の定番ルーティングライブラリ |
| [TanStack Router](https://tanstack.com/router/latest/docs/framework/react/overview) | 型安全なルーティングを重視する場合 |
| [Next.js App Router](https://nextjs.org/docs/app) | React ベースのフルスタックフレームワーク |

### データ取得・フォーム・バリデーション

| リンク | 用途 |
|------|------|
| [TanStack Query](https://tanstack.com/query/latest/docs/framework/react/overview) | サーバー状態、キャッシュ、再取得の管理 |
| [React Hook Form](https://react-hook-form.com/) | 軽量なフォーム管理 |
| [Zod](https://zod.dev/) | ランタイムバリデーションと TypeScript 型推論 |

### テスト

| リンク | 用途 |
|------|------|
| [Vitest](https://vitest.dev/) | Vite と相性の良いテストランナー |
| [Testing Library](https://testing-library.com/docs/) | ユーザー操作に近い形で UI をテストする |
| [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) | React コンポーネントのテスト |
| [user-event](https://testing-library.com/docs/user-event/intro/) | クリック・入力などのユーザー操作を再現 |

### 開発ツール・品質

| リンク | 用途 |
|------|------|
| [React Developer Tools](https://react.dev/learn/react-developer-tools) | コンポーネントツリー、props、state の確認 |
| [eslint-plugin-react-hooks](https://react.dev/reference/eslint-plugin-react-hooks) | Hooks ルールや依存配列の静的チェック |
| [typescript-eslint](https://typescript-eslint.io/getting-started/) | TypeScript 向け ESLint 設定 |
| [ESLint Docs](https://eslint.org/docs/latest/) | ESLint 本体の設定・ルール確認 |

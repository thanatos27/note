---
tags:
  - TypeScript
created: 2026-05-14
updated: 2026-05-14
---

# TypeScript リファレンス

## 目次

- [TypeScript とは](#typescript-とは)
- [インストール](#インストール)
- [ビルド](#ビルド)
  - [tsconfig.json](#tsconfigjson)
  - [tsconfig のおすすめ追加設定](#tsconfig-のおすすめ追加設定)
  - [tsc（コンパイル）](#tscコンパイル)
  - [ビルドツール](#ビルドツール)
- [tsx による直接実行](#tsx-による直接実行)
- [型宣言ファイル](#型宣言ファイル)
- [ESLint](#eslint)
- [テストフレームワーク](#テストフレームワーク)
- [JavaScript との違い](#javascript-との違い)
- [型の基礎](#型の基礎)
  - [プリミティブ型](#プリミティブ型)
  - [any / unknown / never](#any--unknown--never)
  - [Union 型・Intersection 型](#union-型intersection-型)
  - [リテラル型](#リテラル型)
  - [null / undefined / オプショナル](#null--undefined--オプショナル)
  - [型アサーションの注意](#型アサーションの注意)
  - [Enum の注意](#enum-の注意)
  - [type vs interface の使い分け](#type-vs-interface-の使い分け)
- [関数・クラスの型](#関数クラスの型)
  - [基本的な関数の型付け](#基本的な関数の型付け)
  - [オプション引数・デフォルト引数](#オプション引数デフォルト引数)
  - [オーバーロード](#オーバーロード)
  - [アクセス修飾子](#アクセス修飾子)
- [ジェネリクス](#ジェネリクス)
  - [基本構文](#基本構文)
  - [制約（extends）](#制約extends)
  - [ユーティリティ型](#ユーティリティ型)
- [型の操作](#型の操作)
  - [keyof / typeof](#keyof--typeof)
  - [Mapped Types](#mapped-types)
  - [Conditional Types](#conditional-types)
  - [Template Literal Types](#template-literal-types)
- [型ガード・Narrowing](#型ガードnarrowing)
  - [typeof / instanceof チェック](#typeof--instanceof-チェック)
  - [カスタム型ガード（is）](#カスタム型ガードis)
  - [判別可能 Union](#判別可能-union)
  - [satisfies 演算子（TS 4.9〜）](#satisfies-演算子ts-49)
- [モジュール](#モジュール)
  - [import / export の型付け](#import--export-の型付け)
  - [パスエイリアス](#パスエイリアスtsconfig-の-paths)
- [ランタイムバリデーション](#ランタイムバリデーション)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## TypeScript とは

JavaScript に静的型付けを追加したスーパーセット言語。  
`.ts` ファイルに書いて、コンパイルすると `.js` に変換される。

- 型エラーをコンパイル時に検出できる
- エディタの補完・リファクタリング支援が強力になる
- 実行時は通常の JavaScript として動く

---

## インストール

```bash
# プロジェクトローカルにインストール（推奨）
npm install -D typescript

# グローバルインストール
npm install -g typescript
```

---

## ビルド

### tsconfig.json

プロジェクトルートに置くコンパイル設定ファイル。

```bash
npx tsc --init  # 雛形生成（初回セットアップのみ npx で実行）
```

```jsonc
{
  "compilerOptions": {
    "target": "ES2020",        // 出力する JS のバージョン
    "module": "ESNext",        // モジュール形式（Node.js 向けなら "NodeNext" も選択肢）
    "moduleResolution": "bundler", // Vite などのバンドラ向け。Node.js 向けなら "NodeNext"
    "outDir": "./dist",        // 出力先ディレクトリ
    "rootDir": "./src",        // ソースのルート
    "strict": true,            // 厳格な型チェックを有効化
    "esModuleInterop": true,   // CommonJS モジュールを default import できるようにする
    "skipLibCheck": true       // node_modules の型チェックをスキップ
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

#### strict による型チェック

`"strict": true` は複数の厳格な型チェックをまとめて有効にする。主なオプションは以下。

| オプション | 内容 |
|---|---|
| `strictNullChecks` | `null` / `undefined` を独立した型として扱う。無効だとすべての型に `null` を代入できてしまうため、実行時の null 参照エラーをコンパイル時に防げる |
| `strictFunctionTypes` | 関数型の引数をより厳密にチェック（狭い型しか受け取れない関数を、広い型用として代入することを防ぐ） |
| `noImplicitAny` | 型推論できない場合に `any` を禁止 |
| `strictPropertyInitialization` | クラスのプロパティ初期化を強制 |

### tsconfig のおすすめ追加設定

プロジェクトの種類によって必要な設定は変わるが、よく検討するオプションは以下。

| オプション | 内容 |
|---|---|
| `moduleResolution` | import をどう解決するか。Vite などのバンドラ前提なら `"bundler"`、Node.js の ESM/CJS 互換を重視するなら `"NodeNext"` |
| `lib` | 利用する標準 API の型定義。例：ブラウザなら `["ES2022", "DOM"]`、Node.js だけなら DOM を入れない |
| `types` | 読み込むグローバル型を明示する。例：`["node", "vitest/globals"]` |
| `noUncheckedIndexedAccess` | 配列・オブジェクトの添字アクセス結果に `undefined` を含める。存在しないキーや範囲外アクセスに強くなる |
| `exactOptionalPropertyTypes` | `foo?: string` を「省略可」として厳密に扱い、`foo: undefined` との違いを区別する |
| `verbatimModuleSyntax` | import / export を書いた通りに扱う。型だけの import は `import type` が必要になる |
| `isolatedModules` | 1ファイル単位で安全に変換できる書き方に制限する。Vite / Babel / swc などで変換する場合に相性が良い |

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true
  }
}
```

### tsc（コンパイル）

`package.json` の scripts に定義して `npm run` で実行するのが基本。

```json
{
  "scripts": {
    "build": "tsc",
    "build:watch": "tsc --watch",
    "typecheck": "tsc --noEmit"
  }
}
```

```bash
npm run build        # tsconfig.json に従ってコンパイル
npm run build:watch  # ファイル変更を監視して自動コンパイル
npm run typecheck    # 型チェックのみ（JS を出力しない）
```


### ビルドツール

| ツール | 特徴 |
|---|---|
| `tsc` | 公式。型チェックも行う |
| `esbuild` | 超高速。型チェックなし |
| `swc` | Rust製で高速。型チェックなし |
| `Vite` | フロントエンド向け。内部で esbuild を使用 |
| `tsup` | ライブラリ向けバンドラ。esbuild ベース |

高速ビルドツールは型チェックを行わないため、`tsc --noEmit` と組み合わせるのが定番。

---

## tsx による直接実行

`tsx` を使うと `.ts` ファイルを事前ビルドなしで直接実行できる。内部でトランスパイルして実行するが、型チェックは行わない。

```bash
npm install -D tsx

npx tsx src/index.ts
npx tsx watch src/index.ts  # ウォッチモード
```

`ts-node` の代替として推奨されることが多い（高速・設定不要）。

---

## 型宣言ファイル

拡張子 `.d.ts` のファイル。型情報だけを定義し、実装は含まない。

- JS ライブラリに型情報を付与するために使う
- `@types/xxx` パッケージはほとんどが `.d.ts` の集まり

```bash
# 例：Node.js の型定義をインストール
npm install -D @types/node
```

自作する場合：

```ts
// globals.d.ts
declare const __APP_VERSION__: string;

declare module "*.svg" {
  const content: string;
  export default content;
}
```

---

## ESLint

コードの問題をコンパイル前に静的解析で検出するツール。型エラーではなく「書き方のルール」を強制する（未使用変数、`any` の禁止など）。TypeScript プロジェクトでは `typescript-eslint` を使う。

```bash
npm install -D eslint @eslint/js typescript-eslint
```

```js
// eslint.config.mjs
import tseslint from "typescript-eslint";

export default tseslint.config(
  ...tseslint.configs.recommended
);
```

主要ルール例：

| ルール | 内容 |
|---|---|
| `@typescript-eslint/no-explicit-any` | `any` の使用を禁止 |
| `@typescript-eslint/no-unused-vars` | 未使用変数を警告 |
| `@typescript-eslint/explicit-function-return-type` | 関数の戻り値型を明示させる |

---

## テストフレームワーク

| フレームワーク | 特徴 |
|---|---|
| Vitest | Vite ベース。TypeScript をそのまま実行できる。現在の主流 |
| Jest | 老舗。`ts-jest` または `babel-jest` が必要 |
| Node.js `--test` | Node 18〜。外部依存なし。tsx と組み合わせて使う |

```bash
# Vitest
npm install -D vitest
```

```ts
// example.test.ts
import { describe, it, expect } from "vitest";

describe("add", () => {
  it("1 + 2 = 3", () => {
    expect(1 + 2).toBe(3);
  });
});
```

---

## JavaScript との違い

| 項目 | JavaScript | TypeScript |
|---|---|---|
| 型 | 動的型付け（実行時に決まる） | 静的型付け（コンパイル時に決まる） |
| 型エラー検出 | 実行時 | コンパイル時 |
| インターフェース | なし | あり |
| アクセス修飾子 | `#private` フィールドあり | `public` / `private` / `protected`（型チェック上の制限） |
| Enum | なし | あり |
| 実行 | そのまま実行 | コンパイルが必要 |

---

## 型の基礎

### プリミティブ型

```ts
const name: string = "Alice";
const age: number = 30;
const active: boolean = true;
const nothing: null = null;
const undef: undefined = undefined;
const id: symbol = Symbol("id");
const big: bigint = 100n;
```

基本的には型推論に任せて良い。明示が必要なのは引数・戻り値・宣言と代入を分ける場合など。

```ts
// 推論に任せる（推奨）
const name = "Alice";  // string と推論される

// 明示が必要な例
function greet(name: string): string {
  return `Hello, ${name}`;
}
```

配列とタプル：

```ts
const nums: number[] = [1, 2, 3];
const strs: Array<string> = ["a", "b"];

// タプル：要素数と各要素の型が固定
const pair: [string, number] = ["Alice", 30];
```

### any / unknown / never

| 型 | 意味 | 使いどき |
|---|---|---|
| `any` | 型チェックを無効化 | 移行期・どうしようもないとき（極力避ける） |
| `unknown` | 何が入るか不明。使う前に型チェック必須 | 外部入力・JSON パースなど |
| `never` | 到達しないことを表す | 網羅チェック・例外専用関数の戻り値 |

```ts
// unknown の使い方
function parse(input: unknown): string {
  if (typeof input === "string") return input;  // ここで string に絞られる
  throw new Error("not a string");
}

// never で網羅チェック
type Shape = "circle" | "square";
function area(shape: Shape): number {
  switch (shape) {
    case "circle": return 1;
    case "square": return 2;
    default:
      const _exhaustive: never = shape;  // 未処理のケースがあるとエラー
      return _exhaustive;
  }
}
```

### Union 型・Intersection 型

```ts
// Union（どちらかの型）
type ID = string | number;
const id1: ID = "abc";
const id2: ID = 123;

// Intersection（両方の型を持つ）
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged;

const p: Person = { name: "Alice", age: 30 };
```

### リテラル型

特定の値だけを許可する型。

```ts
type Direction = "up" | "down" | "left" | "right";
type StatusCode = 200 | 404 | 500;

function move(dir: Direction) {
  // ...
}
move("up");    // OK
move("diagonal");  // エラー
```

`as const` でオブジェクト・配列をリテラル型に固定できる。

```ts
const config = {
  host: "localhost",
  port: 3000,
} as const;
// config.host の型は string ではなく "localhost"
```

### null / undefined / オプショナル

`strictNullChecks` が有効な場合、`null` と `undefined` は通常の値とは別の型として扱われる。

```ts
let name: string = "Alice";
name = null;       // エラー

let maybeName: string | null = null;
maybeName = "Alice";  // OK
```

オプショナルプロパティの `?` は「プロパティが存在しないかもしれない」という意味。

```ts
type User = {
  name: string;
  email?: string;
};

function send(user: User) {
  // user.email は string | undefined
  const email = user.email ?? "no-email@example.com";
}
```

よく使う書き方：

```ts
function normalize(user: User) {
  user.email?.toLowerCase();        // email があるときだけ呼ぶ
  const email = user.email ?? "";   // null / undefined のときだけデフォルト値
}
```

`!` は「ここでは null / undefined ではない」とコンパイラに伝える非nullアサーション。実行時チェックは増えないため、基本は条件分岐で絞るほうが安全。

```ts
const el = document.querySelector("#app");
el!.textContent = "Hello";  // el が null なら実行時エラー
```

### 型アサーションの注意

`as Type` は値の型を強制的に上書きする。実行時の変換や検証は行わない。

```ts
const value: unknown = "123";

const n = value as number;  // コンパイルは通るが、実体は string のまま
n.toFixed();                // 実行時エラー
```

型アサーションは、DOM API や外部ライブラリなどで TypeScript が推論しきれないときの最終手段として使う。外部入力の安全確認には型ガードやバリデーションを使う。

```ts
const input = document.querySelector("input") as HTMLInputElement | null;

if (input) {
  console.log(input.value);
}
```

`as const` は値をリテラル型として固定するためのアサーション。設定オブジェクトや定数マップでよく使う。

```ts
const statuses = ["loading", "success", "error"] as const;
type Status = typeof statuses[number];  // "loading" | "success" | "error"
```

### Enum の注意

`enum` は TypeScript 独自構文で、コンパイル後の JavaScript に実体が出る。

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

Direction.Up;  // 0
```

単純な定数の集合なら、`as const` オブジェクトと Union 型で代替できることが多い。

```ts
const Direction = {
  Up: "up",
  Down: "down",
  Left: "left",
  Right: "right",
} as const;

type Direction = typeof Direction[keyof typeof Direction];

function move(dir: Direction) {
  // ...
}

move(Direction.Up);
```

既存コードやライブラリ API に合わせる場合は `enum`、JavaScript に近い形で軽く持ちたい場合は `as const` オブジェクト、という使い分けがしやすい。

### type vs interface の使い分け

```ts
// interface：オブジェクトの形を定義。拡張・実装向き
interface User {
  name: string;
  age: number;
}
interface AdminUser extends User {
  role: "admin";
}

// type：何でも定義できる。Union・Intersection・プリミティブも可
type ID = string | number;
type Point = { x: number; y: number };
```

| | `interface` | `type` |
|---|---|---|
| オブジェクト型 | ○ | ○ |
| Union / Intersection | ✗ | ○ |
| 拡張（extends） | ○ | `&` で代替可 |
| 宣言マージ | ○（同名で追記可能） | ✗ |
| クラスへの `implements` | ○ | ○ |

一般的な指針：Union やプリミティブの別名は `type`。オブジェクトの形は `interface` / `type` のどちらでも書けるため、チームやプロジェクトの方針に合わせる。公開 API として拡張される前提の型は `interface`、合成・変換しながら使う型は `type` が向いていることが多い。

---

## 関数・クラスの型

### 基本的な関数の型付け

```ts
function add(a: number, b: number): number {
  return a + b;
}

// アロー関数
const multiply = (a: number, b: number): number => a * b;

// 関数型を変数に持つ
const fn: (x: number) => string = (x) => x.toString();
```

### オプション引数・デフォルト引数

```ts
// オプション引数（? をつける）
function greet(name: string, title?: string): string {
  return title ? `${title} ${name}` : name;
}

// デフォルト引数
function greet(name: string, title: string = "Mr."): string {
  return `${title} ${name}`;
}
```

### オーバーロード

同じ関数名で引数パターンごとに型シグネチャを定義する。

```ts
function parse(value: string): number;
function parse(value: number): string;
function parse(value: string | number): string | number {
  return typeof value === "string" ? Number(value) : String(value);
}
```

### アクセス修飾子

```ts
class User {
  public name: string;        // どこからでもアクセス可（デフォルト）
  private password: string;   // クラス内のみ
  protected role: string;     // クラス内 + サブクラスから
  readonly id: number;        // 代入は初期化時のみ

  constructor(name: string, password: string, id: number) {
    this.name = name;
    this.password = password;
    this.role = "user";
    this.id = id;
  }
}

// コンストラクタ引数に修飾子を書く省略記法
class User {
  constructor(
    public name: string,
    private password: string,
    readonly id: number,
  ) {}
}
```

---

## ジェネリクス

型をパラメータとして受け取る仕組み。再利用可能な型安全なコードを書ける。

### 基本構文

```ts
// 配列から最初の要素を返す
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

first([1, 2, 3]);     // number | undefined
first(["a", "b"]);    // string | undefined
```

```ts
// 複数の型パラメータ：キーと値の型を指定して Map を作る
function toMap<K extends string, V>(keys: K[], values: V[]): Record<K, V> {
  return Object.fromEntries(keys.map((k, i) => [k, values[i]])) as Record<K, V>;
}

toMap(["name", "email"], ["Alice", "alice@example.com"]);
// { name: string, email: string }
```

### 制約（extends）

型パラメータが持つべきプロパティを制限する。

```ts
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

getLength("hello");   // OK
getLength([1, 2, 3]); // OK
getLength(42);        // エラー（number に length はない）
```

### ユーティリティ型

TypeScript 組み込みのジェネリクスを使った型変換ツール。

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

Partial<User>       // 全プロパティをオプションにする
Required<User>      // 全プロパティを必須にする
Readonly<User>      // 全プロパティを readonly にする
Pick<User, "id" | "name">    // 指定プロパティだけ取り出す
Omit<User, "email">          // 指定プロパティを除く
Record<string, number>       // キーと値の型を指定したオブジェクト
```

```ts
// 関数・型から型を取り出す
ReturnType<typeof fetch>         // 関数の戻り値の型
Parameters<typeof fetch>         // 関数の引数の型（タプル）
Awaited<Promise<string>>         // Promise が解決した型（= string）
InstanceType<typeof MyClass>     // クラスのインスタンス型
```

---

## 型の操作

### keyof / typeof

```ts
interface User {
  name: string;
  age: number;
}

// keyof：型のプロパティ名を Union 型として取り出す
type UserKey = keyof User;  // "name" | "age"

// typeof：変数・関数から型を取り出す
const config = { host: "localhost", port: 3000 };
type Config = typeof config;  // { host: string; port: number }

// 組み合わせ：obj のキーとして有効な key だけ受け付け、対応する値の型を返す
function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

### Mapped Types

型のプロパティを一括変換して新しい型を作る。配列の `map()` の型版というイメージ。`[K in keyof T]` で T の全プロパティをループする。

```ts
type Optional<T> = {
  [K in keyof T]?: T[K];  // 全プロパティをオプションにする
};

type Stringify<T> = {
  [K in keyof T]: string;  // 全プロパティの値を string にする
};
```

`Partial<T>` などのユーティリティ型は内部的に Mapped Types で実装されている。

### Conditional Types

条件によって型を分岐させる。

```ts
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>;  // "yes"
type B = IsString<number>;  // "no"
```

```ts
// infer で型を取り出す
type UnpackPromise<T> = T extends Promise<infer U> ? U : T;

type X = UnpackPromise<Promise<string>>;  // string
type Y = UnpackPromise<number>;           // number
```

### Template Literal Types

文字列リテラル型をテンプレートで組み合わせる。

```ts
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"
```

---

## 型ガード・Narrowing

Union 型などを特定の型に絞り込む仕組み。

### typeof / instanceof チェック

```ts
function process(value: string | number) {
  if (typeof value === "string") {
    // ここでは value: string
    console.log(value.toUpperCase());
  } else {
    // ここでは value: number
    console.log(value.toFixed(2));
  }
}

class Dog { bark() {} }
class Cat { meow() {} }

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();  // Dog として扱われる
  }
}
```

### カスタム型ガード（is）

戻り値に `value is Type` を使うと、関数の外まで型が絞られる。`boolean` を返すだけでは型は絞られないため、型判定を関数に切り出したいときに使う。

```ts
// boolean だと型が絞られない
function isStringBool(value: unknown): boolean {
  return typeof value === "string";
}

// value is string にすると if の中で string に絞られる
function isString(value: unknown): value is string {
  return typeof value === "string";
}

const input: unknown = "hello";
if (isString(input)) {
  console.log(input.toUpperCase());  // OK：string に絞られている
}
```

### 判別可能 Union

Union の各型に共通の判別用プロパティを持たせると、条件分岐で安全に型を絞り込める。状態管理や API レスポンスでよく使う。

```ts
type LoadState =
  | { status: "loading" }
  | { status: "success"; data: string[] }
  | { status: "error"; message: string };

function render(state: LoadState): string {
  switch (state.status) {
    case "loading":
      return "Loading...";
    case "success":
      return state.data.join(", ");
    case "error":
      return state.message;
    default:
      const _exhaustive: never = state;
      return _exhaustive;
  }
}
```

`status` のようなリテラル型のプロパティを基準にすると、各分岐の中で必要なプロパティだけが使える。

### satisfies 演算子（TS 4.9〜）

型の整合性を検証しつつ、推論した型（より厳密な型）を保持する。

```ts
type Color = "red" | "green" | "blue";
type Palette = Record<Color, string>;

// as で型アサーションすると、余計なキーやタイプミスを見逃すことがある
const p1 = { red: "#f00", green: "#0f0", blue: "#00f", bleu: "#00f" } as Palette;

// satisfies なら型チェックしつつ、元のオブジェクトの推論も残る
const p2 = {
  red: "#f00",
  green: "#0f0",
  blue: "#00f",
} satisfies Palette;

p2.red.toUpperCase();  // OK（string として推論されている）
```

---

## モジュール

### import / export の型付け

`import type` / `export type` を使うと型だけをインポート・エクスポートできる。コンパイル後の JS から完全に消えるため、実行時のコードに影響しない。型だけを使う import は原則 `import type` にすると、値の import と型の import を明確に分けられる。`verbatimModuleSyntax` を使う環境では特に重要。

```ts
// 通常の import（値も型も読み込む。実行時にも残る）
import { User } from "./types";

// 型だけの import（コンパイル後の JS から消える）
import type { User } from "./types";

// 値と型を同じファイルから混在して使う場合
import { createUser } from "./user";
import type { User } from "./user";
```

```ts
// export 側も同様
export { createUser };       // 値のエクスポート
export type { User };        // 型のみのエクスポート
```

### パスエイリアス（tsconfig の paths）

長い相対パスを短縮できる。

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

```ts
// Before
import { User } from "../../types/user";

// After
import { User } from "@/types/user";
```

`paths` は TypeScript の型チェック時の解決を助ける設定で、コンパイル後の import パスを書き換えるものではない。実行環境やバンドラ（Vite など）側にも同じエイリアス設定が必要になる場合がある。

---

## ランタイムバリデーション

TypeScript の型はコンパイル後の JavaScript から消える。つまり、外部入力が本当にその型かどうかは実行時に検証する必要がある。

```ts
type User = {
  id: number;
  name: string;
};

const data = JSON.parse('{"id": 1, "name": "Alice"}') as User;
// as User はコンパイラへの宣言だけ。実際に検証しているわけではない
```

簡単なものは型ガードで検証できる。

```ts
function isUser(value: unknown): value is User {
  if (typeof value !== "object" || value === null) return false;

  const user = value as Record<string, unknown>;
  return typeof user.id === "number" && typeof user.name === "string";
}

const data: unknown = JSON.parse('{"id": 1, "name": "Alice"}');

if (isUser(data)) {
  console.log(data.name);  // data は User
}
```

API レスポンス、フォーム入力、環境変数などは Zod などのスキーマバリデーションライブラリを使うと、実行時検証と TypeScript の型をまとめて扱いやすい。

---

## 用語集

| 用語 | 説明 |
|---|---|
| 型推論 | 値から自動的に型を決める仕組み |
| 型アサーション | `as Type` で型を強制的に上書きする（型チェックを回避） |
| 型ガード | 条件分岐の中で型を絞り込む仕組み |
| Narrowing | Union 型などを特定の型に絞り込むこと |
| ジェネリクス | 型をパラメータとして受け取る仕組み |
| ユーティリティ型 | `Partial`, `Pick` など組み込みの型変換ツール |
| 宣言マージ | 同名の `interface` を複数定義すると自動的に統合される仕組み |
| 型エラー | コンパイル時に検出される型の不整合 |
| `.d.ts` | 型宣言ファイル。実装なしで型情報だけを定義する |
| `strict` モード | 厳格な型チェックを有効にするコンパイラオプションの集合 |
| supertype / subtype | 上位型 / 下位型。`string` は `string \| number` の subtype |

---

## リンク集

### 公式ドキュメント

| リンク | 用途 |
|---|---|
| [TypeScript 公式サイト](https://www.typescriptlang.org/) | 入口。ダウンロード、ドキュメント、Playground への導線 |
| [The TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) | 基本文法から型システムまで体系的に読む |
| [TSConfig Reference](https://www.typescriptlang.org/tsconfig/) | `tsconfig.json` の各オプションを調べる |
| [TypeScript Playground](https://www.typescriptlang.org/play) | 型推論やコンパイル結果をすぐ試す |
| [Release Notes](https://www.typescriptlang.org/docs/handbook/release-notes/overview.html) | バージョンごとの新機能・破壊的変更を確認する |

### よく参照する公式ページ

| リンク | 用途 |
|---|---|
| [Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html) | 基本の型、Union、型エイリアス、interface |
| [Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) | 型ガード、判別可能 Union、網羅チェック |
| [Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html) | ジェネリクスの基本と制約 |
| [Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html) | `Partial`, `Pick`, `Record`, `ReturnType` など |
| [Modules](https://www.typescriptlang.org/docs/handbook/2/modules.html) | import / export とモジュールの考え方 |

### 周辺ツール

| リンク | 用途 |
|---|---|
| [typescript-eslint](https://typescript-eslint.io/getting-started/) | TypeScript 向け ESLint 設定 |
| [ESLint Docs](https://eslint.org/docs/latest/) | ESLint 本体の設定・ルール確認 |
| [Vite TypeScript Guide](https://vite.dev/guide/features.html#typescript) | Vite で TypeScript を使うときの注意点 |
| [Vitest Docs](https://vitest.dev/) | TypeScript と相性の良いテストフレームワーク |
| [tsx](https://tsx.is/) | `.ts` ファイルを事前ビルドなしで実行するツール |
| [Zod](https://zod.dev/) | ランタイムバリデーションと型推論 |

### 日本語で読む

| リンク | 用途 |
|---|---|
| [サバイバルTypeScript](https://typescriptbook.jp/) | 日本語で体系的に学べる実践寄りの入門資料 |
| [TypeScript Deep Dive 日本語版](https://typescript-jp.gitbook.io/deep-dive) | 型システムや設計の背景を深めたいとき |

---
tags:
  - Go
  - Programming
created: 2026-05-19
updated: 2026-05-19
---

# Go リファレンス

## 目次

- [概要](#概要)
- [型・変数](#型変数)
  - [基本型](#基本型)
  - [変数宣言](#変数宣言)
  - [配列・スライス](#配列スライス)
  - [マップ](#マップ)
  - [型変換](#型変換)
  - [iota](#iota)
  - [ビルトイン関数](#ビルトイン関数)
- [制御構文](#制御構文)
  - [条件分岐](#条件分岐)
  - [繰り返し](#繰り返し)
  - [switch](#switch)
- [関数](#関数)
  - [基本](#基本)
  - [多値返却](#多値返却)
  - [可変長引数](#可変長引数)
  - [クロージャ](#クロージャ)
  - [defer](#defer)
- [ポインタ](#ポインタ)
- [構造体](#構造体)
  - [定義と初期化](#定義と初期化)
  - [メソッド](#メソッド)
  - [埋め込み](#埋め込み)
- [インターフェース](#インターフェース)
  - [型アサーション](#型アサーション)
- [ジェネリクス](#ジェネリクス)
- [エラー処理](#エラー処理)
- [ゴルーチン・チャネル](#ゴルーチンチャネル)
  - [goroutine](#goroutine)
  - [channel](#channel)
  - [select](#select)
  - [sync](#sync)
  - [実務パターン](#実務パターン)
- [パッケージ・モジュール](#パッケージモジュール)
- [ビルド・実行](#ビルド実行)
- [テスト・ツール](#テストツール)
- [よく使う標準ライブラリ](#よく使う標準ライブラリ)
- [高度な機能](#高度な機能)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## 概要

Go（Golang）は Google が開発したシンプルで高速な静的型付け言語。並行処理を言語レベルでサポートし、シングルバイナリにコンパイルできる。  
このメモは **Go 1.21 以降** を前提としている。

---

## 型・変数

### 基本型

| 型 | 説明 | 例 |
|----|------|----|
| `int` | プラットフォーム依存整数（32/64bit） | `42` |
| `int8` `int16` `int32` `int64` | 明示サイズ整数 | `int64(42)` |
| `uint` `uint8` ... | 符号なし整数 | `uint(10)` |
| `float32` | 32bit 浮動小数点 | `3.14` |
| `float64` | 64bit 浮動小数点（デフォルト） | `3.14` |
| `complex64` `complex128` | 複素数 | `1+2i` |
| `bool` | 真偽値 | `true` / `false` |
| `byte` | `uint8` の別名 | `'A'` |
| `rune` | `int32` の別名（Unicode コードポイント） | `'あ'` |
| `string` | イミュータブルなバイト列。UTF-8 テキストとして扱うことが多い | `"hello"` |

### 変数宣言

```go
// var で明示宣言
var x int = 10
var name string = "Alice"

// 型推論
var y = 3.14

// 短縮宣言（関数内で宣言と初期化を同時に行う）
z := 42
msg := "hello"

// まとめて宣言
var (
    a int    = 1
    b string = "b"
)

// ゼロ値（宣言のみで初期化されない場合）
var n int    // 0
var s string // ""
var p *int   // nil

// 定数
const Pi = 3.14159
const (
    StatusOK = 200
    StatusNotFound = 404
)
```

### 配列・スライス

配列は固定長。スライスは動的に伸縮できる参照型で、実用上はほぼスライスを使う。

```go
// 配列（固定長）
arr := [3]int{1, 2, 3}
fmt.Println(arr[0])  // 1
fmt.Println(len(arr)) // 3

// スライス
s := []int{10, 20, 30}
s = append(s, 40)
fmt.Println(s[1:3]) // [20 30]

// make でサイズ・容量指定
s2 := make([]int, 5)      // len=5, cap=5
s3 := make([]int, 0, 10)  // len=0, cap=10

// スライスのコピー
dst := make([]int, len(s))
copy(dst, s)

// 2次元スライス
matrix := [][]int{{1, 2}, {3, 4}}
```

### マップ

マップの反復順は保証されない。複数 goroutine から同時に読み書きする場合は `sync.Mutex` などで保護する。

```go
// 初期化
m := map[string]int{
    "alice": 90,
    "bob":   80,
}

// アクセス・追加・削除
m["carol"] = 70
delete(m, "bob")

// キーの存在確認
val, ok := m["alice"]
if ok {
    fmt.Println(val) // 90
}

// make で空マップ
m2 := make(map[string]int)
```

### 型変換

Go は変数同士の暗黙的な型変換をしない。必要に応じて明示的な変換を書く。  

```go
var i int = 42
f := float64(i)
s := strconv.Itoa(i)       // int → string
n, err := strconv.Atoi("42") // string → int

var f2 float64 = 1 // 未型付け整数定数 1 は float64 として代入できる
```

### iota

`iota` は `const` ブロック内で 0 から順に増える値。enum 風の定数を作るときによく使う。
`const` ブロック内で式を省略すると、直前の定数定義の式が再利用される。

```go
type LogLevel int

const (
    Debug LogLevel = iota // 0
    Info                  // 1（LogLevel = iota が再利用される）
    Warn                  // 2
    Error                 // 3
)

const (
    Read = 1 << iota // 1
    Write            // 2（1 << iota が再利用される）
    Execute          // 4
)
```

### ビルトイン関数

Go には import なしで使えるビルトイン関数がある。

| 関数 | 用途 |
|------|------|
| `len` | 文字列、配列、スライス、マップ、チャネルの長さ |
| `cap` | 配列、スライス、チャネルの容量 |
| `append` | スライスへ要素を追加 |
| `copy` | スライス間で要素をコピー |
| `delete` | マップからキーを削除 |
| `clear` | マップまたはスライスをクリア |
| `make` | スライス、マップ、チャネルを初期化 |
| `new` | ゼロ値を確保してポインタを返す |
| `min` / `max` | 順序付け可能な値の最小・最大 |
| `panic` / `recover` | panic の発生と捕捉 |

```go
nums := []int{3, 1, 2}
nums = append(nums, 4)
fmt.Println(len(nums), cap(nums))
fmt.Println(min(10, 3), max(10, 3))

scores := map[string]int{"alice": 90}
clear(scores) // 空のマップになる
```

---

## 制御構文

### 条件分岐

```go
// if（初期化式を書ける）
if x := compute(); x > 0 {
    fmt.Println("positive")
} else if x == 0 {
    fmt.Println("zero")
} else {
    fmt.Println("negative")
}
```

### 繰り返し

Go のループは `for` のみ。`while` はない。

```go
// C スタイル
for i := 0; i < 5; i++ {
    fmt.Println(i)
}

// while 相当
for n > 0 {
    n--
}

// 無限ループ
for {
    break
}

// range（スライス）
nums := []int{10, 20, 30}
for i, v := range nums {
    fmt.Println(i, v)
}

// range（マップ）
for key, val := range m {
    fmt.Println(key, val)
}

// インデックス不要なら _
for _, v := range nums {
    fmt.Println(v)
}

// Go 1.22 以降: 整数に対する range
for i := range 5 {
    fmt.Println(i) // 0, 1, 2, 3, 4
}
```

### switch

```go
switch x {
case 1:
    fmt.Println("one")
case 2, 3:
    fmt.Println("two or three")
default:
    fmt.Println("other")
}

// 条件式なし switch（if-else 代替）
switch {
case x < 0:
    fmt.Println("negative")
case x > 0:
    fmt.Println("positive")
}

// 型スイッチ
switch v := i.(type) {
case int:
    fmt.Println("int:", v)
case string:
    fmt.Println("string:", v)
}
```

---

## 関数

### 基本

```go
func add(a int, b int) int { // 型を個別に書く
    return a + b
}

func multiply(a, b int) int { // 同じ型なら型名をまとめて書ける
    return a * b
}
```

### 多値返却

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

result, err := divide(10, 3)
```

### 可変長引数

```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

sum(1, 2, 3)

// スライスを展開して渡す
nums := []int{1, 2, 3}
sum(nums...)
```

### クロージャ

自分が定義されたスコープの変数を「閉じ込めて」参照し続ける関数。呼び出しをまたいで状態を保持できる。

```go
func counter() func() int {
    n := 0
    return func() int {
        n++
        return n
    }
}

c := counter()
c() // 1
c() // 2
```

### defer

関数が通常終了または panic で抜けるときに実行される。複数ある場合は LIFO 順（後に登録したものが先に実行）。`os.Exit` や `log.Fatal` でプロセスを終了した場合は実行されない。

```go
func process() {
    db := openDB()
    defer db.Close() // 2番目に実行

    f, _ := os.Open("file.txt")
    defer f.Close() // 1番目に実行（後から登録したので先に実行）

    // ... 処理 ...
}
```

---

## ポインタ

```go
x := 42
p := &x      // アドレス取得
fmt.Println(*p) // デリファレンス → 42

*p = 100
fmt.Println(x)  // 100

// new はゼロ値を確保してポインタを返す
p2 := new(int)  // *int, *p2 == 0
```

---

## 構造体

### 定義と初期化

```go
type Point struct {
    X int
    Y int
}

// フィールド名指定（推奨）
p := Point{X: 1, Y: 2}

// 順番指定（非推奨）
p2 := Point{1, 2}

// ポインタ
p3 := &Point{X: 3, Y: 4}

// ゼロ値で初期化
var p4 Point
```

### メソッド

```go
type Rectangle struct {
    Width, Height float64
}

// 値レシーバ（変更なし）
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// ポインタレシーバ（フィールドを変更する場合）
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}
```

### 埋め込み

フィールドを埋め込むと、その型のフィールドやメソッドが外側の型に昇格する。継承ではないが、メソッドの再利用や合成に使える。

```go
type Animal struct {
    Name string
}

func (a Animal) Speak() string {
    return a.Name + " speaks"
}

type Dog struct {
    Animal        // 埋め込み
    Breed string
}

d := Dog{Animal: Animal{Name: "Rex"}, Breed: "Shiba"}
d.Name    // "Rex"（Animal.Name を直接参照できる）
d.Speak() // "Rex speaks"（Animal のメソッドをそのまま呼べる）
```

---

## インターフェース

Go では `implements` のような宣言を書かない。型がインターフェースに必要なメソッドを持っていれば、自動的にそのインターフェースとして扱える。  
ただし、メソッドをポインタレシーバで定義した場合は、そのメソッドを使えるのは基本的にポインタ側（`*T`）。そのため、値（`T`）ではインターフェースを満たせず、ポインタ（`*T`）なら満たせることがある。

```go
type Stringer interface {
    String() string
}

type Person struct {
    Name string
    Age  int
}

func (p Person) String() string {
    return fmt.Sprintf("%s (%d)", p.Name, p.Age)
}

// Person は Stringer を自動的に実装している
var s Stringer = Person{Name: "Alice", Age: 30}
fmt.Println(s.String())

// 空インターフェース（任意の型を受け取る）
func printAny(v any) { // any = interface{}
    fmt.Println(v)
}
```

### 型アサーション

インターフェース型の値から具体的な型を取り出す。

```go
var i any = "hello"

// 単純な型アサーション（失敗すると panic）
s := i.(string)

// ok パターン（panic を避ける推奨形）
s, ok := i.(string)
if ok {
    fmt.Println(s) // "hello"
}

n, ok := i.(int) // ok == false, n == 0
```

型によって処理を分けたい場合は型スイッチを使う（[switch](#switch) 参照）。

---

## ジェネリクス

型パラメータを使うと、複数の型で使える関数や型を定義できる。`any` は任意の型、`comparable` は `==` / `!=` で比較できる型を表す制約。

```go
func First[T any](items []T) (T, bool) {
    var zero T
    if len(items) == 0 {
        return zero, false
    }
    return items[0], true
}

type Set[T comparable] map[T]struct{}

func (s Set[T]) Add(v T) {
    s[v] = struct{}{}
}

func (s Set[T]) Has(v T) bool {
    _, ok := s[v]
    return ok
}
```

型制約には、許可する型の集合を書ける。

```go
type Integer interface {
    ~int | ~int64 | ~uint | ~uint64
}

func Sum[T Integer](values []T) T {
    var total T
    for _, v := range values {
        total += v
    }
    return total
}
```

---

## エラー処理

Go は例外を使わず、エラーを戻り値で返す。

```go
// error はビルトインのインターフェース（import 不要）
type error interface {
    Error() string
}

// エラーを返す関数
func readConfig(path string) ([]byte, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("readConfig: %w", err) // %w でラップ（errors.Is / As で検査できるようになる）
    }
    return data, nil
}

// 呼び出し側
data, err := readConfig("config.json")
if err != nil {
    log.Fatal(err)
}

// カスタムエラー型
type NotFoundError struct {
    Name string
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s: not found", e.Name)
}

// errors: エラーの生成・ラップ・検査を行う標準パッケージ
// errors.As: エラーチェーンの中に指定した型があるか探し、あれば取り出す
var nfe *NotFoundError
if errors.As(err, &nfe) {
    fmt.Println("not found:", nfe.Name) // nfe にエラーが入っているのでフィールドにアクセスできる
}

// errors.Is: エラーチェーンの中に特定の値があるか確認する
if errors.Is(err, os.ErrNotExist) {
    fmt.Println("file does not exist")
}

// errors.Join: 複数のエラーを1つにまとめる
err = errors.Join(
    errors.New("first error"),
    errors.New("second error"),
)
```

---

## ゴルーチン・チャネル

### goroutine

軽量スレッド。`go` キーワードで起動する。

```go
func say(msg string) {
    fmt.Println(msg)
}

go say("hello") // 別ゴルーチンで実行

// 無名関数
go func() {
    fmt.Println("anonymous goroutine")
}()
```

### channel

ゴルーチン間のデータ受け渡しに使う。

```go
// 作成（chan の後ろの型が送受信するデータの型）
ch := make(chan int)       // int を送受信する非バッファチャネル
ch2 := make(chan int, 10)  // int を送受信するバッファ付きチャネル（容量 10）

// 送受信
go func() {
    ch <- 42 // 送信（受信されるまでブロック）
}()
val := <-ch // 受信（送信されるまでブロック）

// close と range
go func() {
    for i := 0; i < 5; i++ {
        ch2 <- i
    }
    close(ch2)
}()

for v := range ch2 { // close されるまで受信し続ける
    fmt.Println(v)
}

// close 済みチャネルからの受信と ok チェック
closedCh := make(chan int)
close(closedCh)

val, ok := <-closedCh
if !ok {
    fmt.Println("channel closed")
}
```

### select

複数チャネルを同時に監視し、最初に受信できた case を実行する。複数同時に受信できる場合はランダムに1つが選ばれる。

```go
// default なし: いずれかの case が受信できるまでブロック
select {
case msg := <-ch1:
    fmt.Println("ch1:", msg)
case msg := <-ch2:
    fmt.Println("ch2:", msg)
case <-time.After(1 * time.Second): // time.After は指定時間後に値を送るチャネルを返す。受信できたら実行
    fmt.Println("timeout")
}

// default あり: 受信できる case がなければ即 default を実行（ブロックしない）
for {
    select {
    case msg := <-ch:
        fmt.Println("received:", msg)
    default:
        doOtherWork() // データがなければ別の処理を続ける
    }
}
```

### sync

```go
// WaitGroup：複数ゴルーチンが全部終わるまで待つ
// Add(n) でカウンターを増やし、Done() で減らす。Wait() はカウンターが 0 になるまでブロック
var wg sync.WaitGroup

for i := 0; i < 5; i++ {
    wg.Add(1)           // ゴルーチン起動前にカウンターを+1
    go func(n int) {
        defer wg.Done() // ゴルーチン終了時に必ずカウンターを-1
        fmt.Println(n)
    }(i)
}
wg.Wait() // 5つのゴルーチンが全て Done() するまでここでブロック

// Mutex：複数ゴルーチンが同じ変数を同時に読み書きすると値が壊れるため、一度に1つだけ触れるようロックする
var mu sync.Mutex
count := 0

mu.Lock()   // ロック取得（他のゴルーチンは Unlock されるまでここで待つ）
count++
mu.Unlock() // ロック解放

// Once：何度呼んでも Do の中身は最初の1回しか実行されない。グローバルな初期化処理などで使う
var once sync.Once
once.Do(func() {
    fmt.Println("初期化") // 複数ゴルーチンから同時に呼ばれても1回だけ実行される
})
```

### 実務パターン

ゴルーチンを使うときは、完了待ち・キャンセル・エラー処理をセットで考える。`context` はキャンセルやタイムアウトを処理の境界を越えて伝えるために使う。

```go
// jobs <-chan int: 受信専用チャネル（この関数内から送信しようとするとコンパイルエラー）
func worker(ctx context.Context, jobs <-chan int) {
    for {
        select {
        case <-ctx.Done():  // タイムアウトまたは cancel() が呼ばれると閉じる → 終了
            return
        case job, ok := <-jobs:
            if !ok {        // jobs チャネルが close されると ok == false → 終了
                return
            }
            fmt.Println(job)
        }
    }
}

// 3秒後に自動キャンセルされる context を作る
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel() // タイムアウト前に終わった場合もリソース解放のため必ず呼ぶ

jobs := make(chan int)
go worker(ctx, jobs) // worker を別ゴルーチンで起動。jobs にデータを送るか close するまで動き続ける
```

共有データを複数 goroutine から読む・書く場合は、channel、`sync.Mutex`、`sync/atomic` などで保護する。データ競合の検出には `go test -race` を使う。

---

## パッケージ・モジュール

```go
// パッケージ宣言（ファイル先頭）
package main

// インポート
import (
    "fmt"
    "os"

    "github.com/user/repo/pkg" // 外部パッケージ
)

// エクスポート：大文字で始まる識別子が外部公開される
func ExportedFunc() {}  // 公開
func privateFunc() {}   // 非公開
```

`main` パッケージは実行可能ファイルの入口になる。`init` 関数はパッケージ初期化時に自動実行されるが、実行順が読みづらくなりやすいので乱用しない。

```go
func main() {
    fmt.Println("start")
}

func init() {
    fmt.Println("initialize")
}
```

```sh
# モジュール初期化
go mod init github.com/user/myapp

# 依存追加
go get github.com/pkg/errors

# 依存整理
go mod tidy
```

`internal` ディレクトリ配下のパッケージは、親ディレクトリ配下からだけ import できる。

```text
myapp/
  go.mod
  cmd/server/main.go
  internal/config/config.go
```

---

## ビルド・実行

```sh
# 実行
go run .

# ビルド
go build ./...
go build -o app .

# クロスコンパイル例
GOOS=linux GOARCH=amd64 go build -o app-linux .
```

PowerShell では環境変数を別行で設定する。

```powershell
$env:GOOS = "linux"
$env:GOARCH = "amd64"
go build -o app-linux .
```

```sh
# インストール
go install github.com/user/tool@latest

# モジュール内のパッケージ一覧
go list ./...
```

`go run` はその場でビルドして実行する。配布用バイナリを作るときは `go build` を使う。

---

## テスト・ツール

Go は標準でテスト、ベンチマーク、ファズテスト、整形ツールを持っている。

`go test` はテスト専用のビルド・実行コマンド。`*_test.go` ファイルだけをコンパイル対象に加え、`TestXxx` / `BenchmarkXxx` などの関数を実行する。`go run` や `go build` では `*_test.go` は無視される。

```go
// 通常のテスト。Test で始まる関数名が必須
func TestAdd(t *testing.T) {
    got := add(2, 3)
    if got != 5 {
        t.Fatalf("got %d, want %d", got, 5) // テスト失敗時にメッセージを出して即終了
    }
}

// テーブル駆動テスト：複数ケースを構造体スライスで管理する慣用パターン
func TestAddTable(t *testing.T) {
    tests := []struct {
        name string
        a, b int
        want int
    }{
        {"positive", 2, 3, 5},
        {"zero", 0, 3, 3},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) { // サブテストとして個別に実行・識別できる
            if got := add(tt.a, tt.b); got != tt.want { // if 初期化式; 条件式
                t.Fatalf("got %d, want %d", got, tt.want)
            }
        })
    }
}

// ベンチマーク。Benchmark で始まる関数名が必須。go test -bench=. で実行
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ { // b.N はフレームワークが自動で調整する反復回数
        _ = add(2, 3)
    }
}

// ファズテスト：ランダムな入力を自動生成してクラッシュを探す。go test -fuzz=. で実行
func FuzzParse(f *testing.F) {
    f.Add("42") // シードコーパス（ランダム生成の起点となる初期入力）
    f.Fuzz(func(t *testing.T, s string) {
        _, _ = strconv.Atoi(s) // panic しなければ OK
    })
}
```

```sh
# テスト
go test ./...

# 詳細表示
go test -v ./...

# ベンチマーク
go test -bench=. ./...

# ファズテスト
go test -fuzz=Fuzz ./...

# データ競合検出
go test -race ./...

# 整形・静的チェック
go fmt ./...
go vet ./...

# ドキュメント表示
go doc fmt.Println
```

---

## よく使う標準ライブラリ

| パッケージ | 主な用途 |
|-----------|---------|
| `fmt` | フォーマット出力・スキャン |
| `os` | ファイル・環境変数・プロセス |
| `io` | Reader/Writer インターフェース |
| `bufio` | バッファリング入出力 |
| `strings` | 文字列操作 |
| `strconv` | 型変換（数値 ↔ 文字列） |
| `math` | 数学関数 |
| `time` | 時刻・タイマー・Duration |
| `encoding/json` | JSON エンコード・デコード |
| `net/http` | HTTP クライアント・サーバー |
| `context` | キャンセル・タイムアウト伝播 |
| `log` | シンプルなロギング |
| `sync` | Mutex / WaitGroup / Once |
| `sync/atomic` | 低レベルなアトミック操作 |
| `errors` | エラー生成・ラップ・検査 |
| `testing` | テスト・ベンチマーク |
| `path/filepath` | ファイルパス操作 |
| `slices` | スライスの検索・ソート・比較など |
| `maps` | マップのコピー・比較・キー列挙など |
| `cmp` | 順序比較・デフォルト値選択 |
| `iter` | イテレータの基本型 |
| `log/slog` | 構造化ロギング |

```go
// JSON
type User struct {
    Name string `json:"name"`
    Age  int    `json:"age,omitempty"`
}

data, _ := json.Marshal(User{Name: "Alice", Age: 30})
fmt.Println(string(data)) // {"name":"Alice","age":30}

var u User
json.Unmarshal(data, &u)

// HTTP サーバー
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello, World!")
})
http.ListenAndServe(":8080", nil)

// context
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// time
now := time.Now()
t := time.Date(2024, time.January, 1, 0, 0, 0, 0, time.UTC)
duration := now.Sub(t)
time.Sleep(100 * time.Millisecond)

// slices / maps / cmp
nums := []int{3, 1, 2}
slices.Sort(nums)
fmt.Println(slices.Contains(nums, 2))

// maps.Keys と slices.Sorted は Go 1.23 以降
m := map[string]int{"alice": 90, "bob": 80}
keys := slices.Sorted(maps.Keys(m))
fmt.Println(keys)

// cmp.Or は Go 1.22 以降
fmt.Println(cmp.Or("", "default"))

// log/slog
slog.Info("request completed", "status", 200, "path", "/")
```

---

## 高度な機能

普段のアプリケーションコードでは多用しないが、ライブラリや低レベル処理で出てくる機能。

| 機能 | 概要 |
|------|------|
| `reflect` | 実行時に型情報を調べる。JSON、ORM、DI などで使われる |
| `unsafe` | 型安全性を外れてメモリを扱う。必要な場面は限定的 |
| `cgo` | Go から C のコードやライブラリを呼び出す |
| build tags | OS やビルド条件ごとにファイルを切り替える |

```go
// reflect
t := reflect.TypeOf(User{})
fmt.Println(t.Name())
```

build tag はファイル先頭に書く。

```go
//go:build linux

package config
```

`unsafe` と `cgo` は移植性・保守性・ビルド速度に影響しやすい。標準ライブラリや通常の Go コードで済むなら、そちらを優先する。

---

## 用語集

| 用語 | 説明 |
|------|------|
| goroutine | Go の軽量スレッド。`go` キーワードで起動する |
| channel | ゴルーチン間のデータ通信路 |
| interface | メソッドセットを定義する型。暗黙的に実装される |
| generics | 型パラメータを使って、複数の型に対応する関数や型を定義する機能 |
| type constraint | 型パラメータに使える型やメソッドを制限するインターフェース |
| defer | 関数終了時に実行されるステートメント |
| panic / recover | 通常のエラー処理ではなく、プログラムの不変条件違反などで使う仕組み。`recover` は defer 内でのみ panic を捕捉できる |
| zero value | 変数のデフォルト初期値（`int` → `0`、`string` → `""`、ポインタ → `nil`） |
| blank identifier `_` | 不要な値を捨てるための識別子 |
| iota | `const` ブロック内で 0 から順に増える定数生成用の識別子 |
| context | キャンセル、タイムアウト、リクエストスコープの値を処理に伝える仕組み |
| data race | 複数 goroutine が同じデータへ同期なしにアクセスし、少なくとも一方が書き込む状態 |
| rune | Unicode コードポイントを表す `int32` の別名 |
| slice | 配列の動的ビュー。`len`（長さ）と `cap`（容量）を持つ |
| embedding | 構造体を別の構造体に埋め込んでメソッドを再利用するパターン |

---

## リンク集

- [公式ドキュメント](https://go.dev/doc/)
- [A Tour of Go](https://go.dev/tour/)
- [Go by Example](https://gobyexample.com/)
- [Effective Go](https://go.dev/doc/effective_go)
- [標準ライブラリリファレンス](https://pkg.go.dev/std)

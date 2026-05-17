---
tags:
  - CSharp
  - Programming
created: 2026-05-18
updated: 2026-05-18
---

# C# リファレンス

## 目次

- [概要](#概要)
- [型・変数](#型変数)
  - [基本型](#基本型)
  - [変数宣言](#変数宣言)
  - [配列](#配列)
  - [型変換](#型変換)
- [制御構文](#制御構文)
  - [条件分岐](#条件分岐)
  - [繰り返し](#繰り返し)
  - [パターンマッチング](#パターンマッチング)
- [メソッド](#メソッド)
- [enum](#enum)
- [クラス](#クラス)
  - [定義と継承](#定義と継承)
  - [アクセス修飾子](#アクセス修飾子)
  - [プロパティ](#プロパティ)
  - [static](#static)
- [インターフェースと抽象クラス](#インターフェースと抽象クラス)
- [ジェネリクス](#ジェネリクス)
- [コレクション](#コレクション)
- [LINQ](#linq)
- [デリゲート・イベント・ラムダ](#デリゲートイベントラムダ)
- [例外処理](#例外処理)
- [null 安全](#null-安全)
- [非同期処理](#非同期処理)
- [record・struct](#recordstruct)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## 概要

C# は Microsoft が開発した静的型付けのオブジェクト指向言語。.NET ランタイム上で動く。  
このメモは **C# 10 以降**（.NET 6+）を前提としている。

---

## 型・変数

### 基本型

| 型 | 説明 | 例 |
|----|------|----|
| `int` | 32bit 整数 | `42` |
| `long` | 64bit 整数 | `42L` |
| `float` | 32bit 浮動小数点 | `3.14f` |
| `double` | 64bit 浮動小数点 | `3.14` |
| `decimal` | 高精度十進数（金融向け） | `3.14m` |
| `bool` | 真偽値 | `true` / `false` |
| `char` | 1 文字 | `'A'` |
| `string` | 文字列（参照型） | `"hello"` |
| `object` | すべての型の基底 | — |

### 変数宣言

```csharp
int x = 10;
var name = "Alice";       // 型推論（コンパイル時に確定する）
const int Max = 100;      // 定数（変更不可）

// タプル
var (a, b) = (1, 2);
(int x, int y) point = (3, 4);
```

### 配列

同じ型の値を固定長で並べる。要素数は `Length` で取得する。

```csharp
int[] scores = { 80, 90, 100 };

Console.WriteLine(scores[0]);      // 80
Console.WriteLine(scores.Length);  // 3

scores[1] = 95;
```

### 型変換

```csharp
// 暗黙の変換（小 → 大は安全）
int i = 10;
double d = i;

// 明示的キャスト（狭い型への変換。情報が失われることがある）
double pi = 3.14;
int n = (int)pi;   // → 3（切り捨て）

// 安全な変換
int result;
bool ok = int.TryParse("123", out result);  // 失敗しても例外を投げない

// as / is
object obj = "hello";
string? s = obj as string;   // 失敗すると null
if (obj is string str) { }   // パターンマッチング（後述）
```

---

## 制御構文

### 条件分岐

```csharp
// if / else if / else
if (hp <= 0)
    Die();
else if (hp < 30)
    Console.WriteLine("Low HP");
else
    Heal();

// switch（値マッチ）
switch (state)
{
    case "idle":
        PlayIdle();
        break;
    case "run":
        PlayRun();
        break;
    default:
        break;
}

// switch 式（C# 8+）
string label = state switch
{
    "idle" => "待機",
    "run"  => "走行",
    _      => "不明",
};

// 三項演算子
string result = hp > 0 ? "生存" : "死亡";
```

### 繰り返し

```csharp
// for
for (int i = 0; i < 10; i++)
    Console.WriteLine(i);

// foreach（コレクション向け）
foreach (var item in list)
    Console.WriteLine(item);

// while
while (isRunning)
    Update();

// do-while（最低 1 回実行）
do
{
    Input();
} while (!confirmed);

// break / continue
for (int i = 0; i < 10; i++)
{
    if (i == 5) break;     // ループ終了
    if (i % 2 == 0) continue;  // 次のイテレーションへ
    Console.WriteLine(i);
}
```

### パターンマッチング

```csharp
// is でパターンマッチ（C# 7+）
if (obj is string s)
    Console.WriteLine(s.Length);

// switch 式でパターンマッチ（C# 8+）
string Describe(object obj) => obj switch
{
    int n when n < 0 => "負の数",
    int n            => $"整数: {n}",
    string s         => $"文字列: {s}",
    null             => "null",
    _                => "その他",
};
```

---

## メソッド

処理を名前付きでまとめる。戻り値がない場合は `void` を使う。

```csharp
int Add(int a, int b)
{
    return a + b;
}

void PrintHp(int hp)
{
    Console.WriteLine($"HP: {hp}");
}

int result = Add(1, 2);
PrintHp(100);
```

同じ名前でも、引数の型や数が違えばオーバーロードできる。

```csharp
int Add(int a, int b) => a + b;
double Add(double a, double b) => a + b;
```

---

## enum

決まった候補の中から値を選ばせたいときに使う。状態や種類を表すのに向いている。

```csharp
public enum GameState
{
    Title,
    Playing,
    GameOver,
}

GameState state = GameState.Playing;

if (state == GameState.Playing)
{
    Console.WriteLine("プレイ中");
}
```

---

## クラス

### 定義と継承

```csharp
// 基底クラス
public class Animal
{
    public string Name { get; set; }

    public Animal(string name)
    {
        Name = name;
    }

    public virtual string Speak() => "...";
}

// 継承
public class Dog : Animal
{
    public Dog(string name) : base(name) { }

    public override string Speak() => "Woof!";
}
```

- `virtual` — サブクラスでオーバーライド可能
- `override` — 親の仮想メソッドを上書き
- `sealed` — これ以上の継承・オーバーライドを禁止
- `: base(...)` — 親コンストラクタの呼び出し

### アクセス修飾子

| 修飾子 | 範囲 |
|--------|------|
| `public` | どこからでもアクセス可 |
| `private` | クラス内のみ（クラスメンバーのデフォルト） |
| `protected` | クラス内 + 派生クラス |
| `internal` | 同一アセンブリ内 |
| `protected internal` | 同一アセンブリ or 派生クラス |

### プロパティ

```csharp
// フルプロパティ（バッキングフィールドあり）
private int _hp;
public int Hp
{
    get => _hp;
    set => _hp = Math.Clamp(value, 0, MaxHp);
}

// 自動プロパティ
public string Name { get; set; } = "Player";
public int Level { get; private set; } = 1;  // 外から set 不可

// 計算プロパティ（バッキングフィールドなし）
public bool IsDead => Hp <= 0;
```

### static

```csharp
public class MathHelper
{
    public static int Add(int a, int b) => a + b;  // インスタンス不要で呼べる
    public static readonly float Pi = 3.14159f;
}

// 呼び出し
int result = MathHelper.Add(1, 2);
```

---

## インターフェースと抽象クラス

```csharp
// インターフェース — 基本的には「何ができるか」を表す契約
public interface IDamageable
{
    int Hp { get; }
    void TakeDamage(int amount);
}

// 抽象クラス — 部分的に実装できる基底クラス
public abstract class Enemy
{
    public abstract void Attack();      // サブクラスで必ず実装
    public virtual void Move() { }     // 任意でオーバーライド
}

// 実装
public class Slime : Enemy, IDamageable
{
    public int Hp { get; private set; } = 30;

    public void TakeDamage(int amount) => Hp -= amount;
    public override void Attack() => Console.WriteLine("体当たり");
}
```

インターフェースは多重実装可能。抽象クラスは単一継承のみ。
近年の C# ではインターフェースに既定実装を書けるが、まずは「実装クラスが満たすべき契約」と考えると分かりやすい。

---

## ジェネリクス

型をパラメータとして受け取る仕組み。コレクションや汎用アルゴリズムに使う。

```csharp
// ジェネリッククラス
public class Box<T>
{
    public T Value { get; set; }
    public Box(T value) => Value = value;
}

var box = new Box<int>(42);
var strBox = new Box<string>("hello");

// ジェネリックメソッド
public T Max<T>(T a, T b) where T : IComparable<T>
    => a.CompareTo(b) > 0 ? a : b;
```

`where T : ...` で型制約を付けられる（`class`、`struct`、`new()`、インターフェース名など）。

---

## コレクション

```csharp
// List<T> — 動的配列
var list = new List<int> { 1, 2, 3 };
list.Add(4);
list.Remove(2);
int count = list.Count;   // 要素数

// Dictionary<TKey, TValue> — キーと値のペア
var dict = new Dictionary<string, int>
{
    ["apple"] = 100,
    ["banana"] = 80,
};
dict["orange"] = 120;
dict.TryGetValue("apple", out int price);  // 安全な取得

// HashSet<T> — 重複なし集合
var set = new HashSet<string> { "a", "b", "c" };
set.Add("a");   // 重複は無視される

// Queue<T> / Stack<T>
var queue = new Queue<int>();
queue.Enqueue(1);
int first = queue.Dequeue();

var stack = new Stack<int>();
stack.Push(1);
int top = stack.Pop();
```

---

## LINQ

コレクションに対するクエリ操作。`using System.Linq;` が必要。

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5, 6 };

// メソッド構文（よく使われる）
var evens = numbers.Where(n => n % 2 == 0);          // フィルタ
var doubled = numbers.Select(n => n * 2);             // 変換
var sorted = numbers.OrderBy(n => n);                 // 昇順ソート
var sum = numbers.Sum();                              // 合計
var first = numbers.First(n => n > 3);               // 最初の一致
var any = numbers.Any(n => n > 5);                   // 条件を満たすものがあるか
var all = numbers.All(n => n > 0);                   // 全部条件を満たすか
var list = numbers.Where(n => n > 2).ToList();       // List に変換

// チェーン
var result = numbers
    .Where(n => n % 2 == 0)
    .Select(n => n * n)
    .OrderByDescending(n => n)
    .ToList();
// → [36, 16, 4]
```

`Where` や `Select` などの LINQ クエリは遅延評価。`ToList()`、`ToArray()`、`Sum()`、`First()`、`Any()` など結果を取り出す操作で実行される。

---

## デリゲート・イベント・ラムダ

```csharp
// デリゲート — メソッドを変数に入れる型
Func<int, int, int> add = (a, b) => a + b;
Action<string> print = msg => Console.WriteLine(msg);
Predicate<int> isPositive = n => n > 0;

// よく使う組み込みデリゲート型
// Func<T, TResult>    — 戻り値あり
// Action<T>           — 戻り値なし（void）
// Predicate<T>        — bool を返す

// ラムダ式
var square = (int x) => x * x;
var greet = () => Console.WriteLine("Hello");

var btn = new Button();
btn.Clicked += () => Console.WriteLine("押された");
btn.Click();
```

```csharp
// イベントを持つクラス
public class Button
{
    public event Action? Clicked;

    public void Click() => Clicked?.Invoke();
}
```

---

## 例外処理

```csharp
try
{
    int result = int.Parse("abc");  // FormatException を投げる
}
catch (FormatException ex)
{
    Console.WriteLine($"フォーマットエラー: {ex.Message}");
}
catch (Exception ex)
{
    Console.WriteLine($"予期しないエラー: {ex.Message}");
    throw;  // 再スロー
}
finally
{
    // 例外の有無に関わらず必ず実行（リソース解放など）
}

// 独自例外
public class GameOverException : Exception
{
    public GameOverException(string message) : base(message) { }
}

throw new GameOverException("HP が 0 になりました");
```

---

## null 安全

C# 8+ の Nullable Reference Types。`<Nullable>enable</Nullable>` を `.csproj` に追加して有効化。

```csharp
string name = "Alice";   // null 不可
string? nickname = null; // null 許容（? を付ける）

// null チェック
if (nickname != null)
    Console.WriteLine(nickname.Length);

// null 条件演算子
int? len = nickname?.Length;  // null なら null を返す

// null 合体演算子
string display = nickname ?? "名無し";         // null なら右辺を使う
nickname ??= "デフォルト";                    // null なら代入

// null 免除演算子（! — 自分で null でないと保証するとき）
Console.WriteLine(nickname!.Length);
```

---

## 非同期処理

I/O 待機などをノンブロッキングで処理する。

```csharp
// async メソッドは Task または Task<T> を返す
public async Task<string> FetchDataAsync(string url)
{
    using var client = new HttpClient();
    string result = await client.GetStringAsync(url);  // ここで一時的に制御を返す
    return result;
}

// 呼び出し
var data = await FetchDataAsync("https://example.com");

// 並列実行
var task1 = FetchDataAsync("url1");
var task2 = FetchDataAsync("url2");
var results = await Task.WhenAll(task1, task2);

// 値を返さない非同期
public async Task SaveAsync()
{
    await File.WriteAllTextAsync("file.txt", "data");
}
```

`async void` はイベントハンドラ以外では使わない（例外をキャッチできない）。

---

## record・struct

### record（C# 9+）

```csharp
// イミュータブルなデータクラス。equals・tostring が自動生成される
public record Point(int X, int Y);

var p1 = new Point(1, 2);
var p2 = new Point(1, 2);
bool eq = p1 == p2;  // → true（値で比較）

// with で一部だけ変更したコピーを作る
var p3 = p1 with { X = 10 };  // → Point(10, 2)
```

### struct

`class` との使い分けの目安は、**個体・状態を持つものは `class`、小さな値のまとまりは `struct`**。たとえば Unity では、プレイヤーや敵のようなゲームオブジェクトは `class` として扱う一方、`Vector2` や `Color` は「座標」「色」という値なので `struct` として定義されている。

```csharp
// 値型。小さなデータ（座標・色など）に適する
public struct Vector2
{
    public float X;
    public float Y;

    public Vector2(float x, float y) { X = x; Y = y; }
    public float Length() => MathF.Sqrt(X * X + Y * Y);
}

var v = new Vector2(3, 4);
```

- `struct` は値型（コピーして渡される）、`class` は参照型（参照を渡す）
- `struct` は値そのものを保持するが、ボックス化された場合やクラスのフィールド・配列要素など、状況によってはヒープ上に置かれる

---

## 用語集

| 用語 | 説明 |
|------|------|
| **値型（Value Type）** | 変数に値そのものが入る型。代入・引数渡しでコピーされる。`int`・`float`・`bool`・`struct` など |
| **参照型（Reference Type）** | 変数にオブジェクトへの参照（アドレス）が入る型。`class`・`string`・配列など |
| **ヒープ / スタック** | ヒープは動的に確保されるメモリ領域（参照型が乗る）。スタックは関数呼び出しに伴う短命なメモリ領域（値型が乗る） |
| **GC（ガベージコレクション）** | 参照されなくなったヒープ上のオブジェクトを自動的に解放する仕組み |
| **null** | 「値がない」ことを表す特殊な値。参照型に代入できる |
| **Nullable Reference Types** | C# 8+ の機能。`?` を付けた型だけ null を許容し、それ以外は null 不可として警告を出す |
| **static** | インスタンスを生成せずにクラス名で直接アクセスできるメンバー |
| **プロパティ** | フィールドへのアクセスを `get`/`set` でカプセル化したもの。外から見るとフィールドのように使える |
| **仮想メソッド（virtual）** | サブクラスでオーバーライドできるメソッド |
| **抽象メソッド（abstract）** | 実装を持たず、サブクラスでの実装を強制するメソッド |
| **インターフェース** | メソッド・プロパティのシグネチャだけを定義する型。多重実装が可能 |
| **ジェネリクス** | 型をパラメータとして受け取る仕組み。`List<T>` の `T` がそれ |
| **デリゲート** | メソッドを変数のように扱うための型。関数ポインタに相当する |
| **ラムダ式** | `(引数) => 処理` の形で書く匿名関数 |
| **イベント** | デリゲートをベースにした通知の仕組み。`+=` で複数のハンドラを登録できる |
| **LINQ** | コレクションに対するクエリ操作を提供するライブラリ群。`Where`・`Select` など |
| **遅延評価** | LINQ のクエリは定義時ではなく、`ToList()` 等で結果を取り出した時点で実行される性質 |
| **配列** | 同じ型の値を固定長で並べるデータ構造。`Length` で要素数を取得する |
| **enum** | 決まった候補を名前付きで表す型。状態や種類を表すのに使う |
| **async / await** | 非同期処理を同期的に書けるようにする構文。`await` で結果が返るまでスレッドをブロックせずに待つ |
| **Task** | 非同期操作を表す型。`Task<T>` は戻り値ありの非同期処理 |
| **record** | C# 9+ のイミュータブルなデータクラス。値ベースの等値比較と `with` によるコピーが特徴 |
| **struct** | 値型のクラスに相当する型。小さなデータ構造に適し、GC の負荷が低い |
| **アセンブリ** | .NET のビルド成果物（`.dll` / `.exe`）。`internal` アクセス修飾子はこの単位で効く |
| **名前空間（namespace）** | クラス名の衝突を防ぐための論理的な区画。`using` で省略して使える |
| **型推論（var）** | コンパイラが右辺の型から変数の型を自動的に推論する機能。型はコンパイル時に確定する |

---

## リンク集

- [C# ドキュメント（Microsoft）](https://learn.microsoft.com/ja-jp/dotnet/csharp/)
- [C# 言語リファレンス](https://learn.microsoft.com/ja-jp/dotnet/csharp/language-reference/)
- [.NET API ブラウザ](https://learn.microsoft.com/ja-jp/dotnet/api/)
- [C# の新機能一覧](https://learn.microsoft.com/ja-jp/dotnet/csharp/whats-new/csharp-version-history)

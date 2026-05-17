---
tags:
  - Godot
  - CSharp
  - GameDev
created: 2026-05-18
updated: 2026-05-18
---

# Godot C# 開発

## 目次

- [セットアップ](#セットアップ)
- [GDScript vs C# 比較](#gdscript-vs-c-比較)
- [基本的な書き方](#基本的な書き方)
  - [クラス定義とライフサイクル](#クラス定義とライフサイクル)
  - [変数・プロパティ](#変数プロパティ)
  - [ノードの参照](#ノードの参照)
  - [シグナル](#シグナル)
  - [入力処理](#入力処理)
- [命名規則](#命名規則)
- [Tips](#tips)

---

## セットアップ

C# を使うには **.NET 対応版の Godot** が必要。通常版（C# 非対応版）とは別バイナリ。

1. [公式サイト](https://godotengine.org/download/) から `Godot Engine - .NET` をダウンロード
2. [.NET SDK](https://dotnet.microsoft.com/download) をインストール（Godot 4.4 では .NET 8.0。基本は最新の安定版を使う）
3. Godot エディタ上でスクリプトを新規作成するとき、言語に `C#` を選択
4. 初回ビルド時に `.sln` と `.csproj` などの C# プロジェクトファイルが自動生成される

エディタには **VS Code** か **JetBrains Rider** が推奨。Godot の `Editor Settings → Dotnet → Editor` で外部エディタを指定する。

---

## GDScript vs C# 比較

| 項目 | GDScript | C# |
|------|----------|----|
| ファイル拡張子 | `.gd` | `.cs` |
| 型付け | 動的（型アノテーション可） | 静的型付け |
| 実行速度 | やや遅い | 速い（JIT コンパイル） |
| 学習コスト | 低い（Godot 専用構文） | 中程度（C# の知識が必要） |
| IDE サポート | Godot エディタ内 | VS Code / Rider 等 |
| ホットリロード | 速い | コンパイルが挟まる分遅い |
| 向いている場面 | 初学者・特にこだわりが無い場合 | Unityからのリソース移行・既存 C# 知識がある場合 |
| 混在 | ○（同プロジェクトで混在可能） | ○ |

GDScript と C# は同じプロジェクト内で混在できるが、**C# から GDScript のクラスを直接型付きで参照することはできない**（`GodotObject` 経由か、シグナル・グループ経由で連携する）。

---

## 基本的な書き方

### クラス定義とライフサイクル

GDScript の `extends` に対応するのは `partial class` の継承。

```gdscript
# GDScript
extends CharacterBody2D

func _ready() -> void:
    pass

func _physics_process(delta: float) -> void:
    pass
```

```csharp
// C#
using Godot;

public partial class Player : CharacterBody2D
{
    public override void _Ready()
    {
        // 初期化
    }

    public override void _PhysicsProcess(double delta)
    {
        // 物理フレーム処理
    }

    public override void _Process(double delta)
    {
        // 毎フレーム処理
    }
}
```

- クラス名はファイル名と一致させる（`Player.cs` → `public partial class Player`）
- `partial` は必須（Godot のコード生成機能と組み合わせるため）
- `delta` の型が GDScript の `float` に対して C# では `double`

### 変数・プロパティ

```gdscript
# GDScript
@export var speed: float = 200.0
@onready var sprite: Sprite2D = $Sprite2D
const MAX_HP = 100
```

```csharp
// C#
[Export]
public float Speed { get; set; } = 200.0f;

// @onready に相当するものは _Ready() で初期化するか GetNode を使う
private Sprite2D _sprite;

public override void _Ready()
{
    _sprite = GetNode<Sprite2D>("Sprite2D");
}

private const int MaxHp = 100;
```

`[Export]` 属性でエディタのインスペクターに公開できる。公開するプロパティは通常 `public` にする。

### ノードの参照

```gdscript
# GDScript
$Sprite2D.visible = false
@onready var anim: AnimationPlayer = $AnimationPlayer
```

```csharp
// C#
GetNode<Sprite2D>("Sprite2D").Visible = false;

// [Export] でエディタからノードをアサインする場合
[Export]
private AnimationPlayer _animationPlayer;
```

```csharp
// _Ready() で取得する場合
private AnimationPlayer _animationPlayer;

public override void _Ready()
{
    _animationPlayer = GetNode<AnimationPlayer>("AnimationPlayer");
}
```

C# では `$ノード名` ショートカットは使えない。代わりに `GetNode<T>("パス")` を使う。

### シグナル

```gdscript
# GDScript
# シグナルの定義
signal health_changed(new_hp: int)

# シグナルの発火
health_changed.emit(hp)

# シグナルへの接続（コードで）
$Player.health_changed.connect(_on_health_changed)

func _on_health_changed(new_hp: int) -> void:
    $HUD/HealthBar.value = new_hp
```

```csharp
// C#
[Signal]
public delegate void HealthChangedEventHandler(int newHp);

// 発火
EmitSignal(SignalName.HealthChanged, hp);

// 接続
GetNode<Player>("Player").HealthChanged += OnHealthChanged;

private void OnHealthChanged(int newHp)
{
    GetNode<ProgressBar>("HUD/HealthBar").Value = newHp;
}
```

`[Signal]` 属性を付けた `delegate` を定義することでシグナルが使える。  
イベント名は `EventHandler` サフィックスを除いたもの（`HealthChanged`）がシグナル名になる。

### 入力処理

```gdscript
# GDScript
var dir = Input.get_axis("ui_left", "ui_right")
if Input.is_action_just_pressed("jump"):
    jump()
```

```csharp
// C#
float dir = Input.GetAxis("ui_left", "ui_right");
if (Input.IsActionJustPressed("jump"))
{
    Jump();
}
```

Input API の呼び出し方はほぼ同じ。メソッド名が PascalCase になる。

---

## 命名規則

GDScript は snake_case、C# は .NET の慣習に従う。

| 対象 | GDScript | C# |
|------|----------|----|
| クラス名 | `MyClass` (PascalCase) | `MyClass` (PascalCase) |
| 関数名 | `do_something()` (snake_case) | `DoSomething()` (PascalCase) |
| 変数名（ローカル） | `my_var` (snake_case) | `myVar` (camelCase) |
| プロパティ（public） | — | `MyProperty` (PascalCase) |
| フィールド（private） | — | `_myField` (アンダースコア + camelCase) |
| 定数 | `MAX_HP` (UPPER_SNAKE) | `MaxHp` (PascalCase) |
| ライフサイクル関数 | `_ready()`, `_process()` | `_Ready()`, `_Process()` |

Godot の組み込みクラスはすべて PascalCase（`CharacterBody2D`, `AnimationPlayer`）なので、C# から使うときも自然に一致する。

---

## Tips

- **プロジェクト全体で言語を統一するのが理想** — C# と GDScript の混在は可能だが、型安全な相互参照がしにくいため
- **Web export には非対応** — Godot 4 の C# プロジェクトは Web にエクスポートできない。Android / iOS は対応しているが experimental 扱い
- **Rider は Godot サポートが充実** — デバッガ・補完・シグナルのコードジャンプが VS Code より強い
- **`[Export]` でノード参照を渡す** — `GetNode()` をハードコードするより、`[Export]` でエディタからアサインする方が柔軟
- **Nullable に注意** — C# 8+ の `nullable reference types` が有効な場合、`_sprite!` のように null 許容を明示する場面がある
- **`float` と `double`** — Godot の C# API は `float` と `double` が混在している。`delta` は `double` だが、`Vector2` のメンバーは `float`
- **ホットリロードは遅い** — スクリプト変更のたびにコンパイルが走るため、ちょっとした確認は GDScript の方が速い

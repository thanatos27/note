---
tags:
  - Godot
  - GameDev
created: 2026-05-17
updated: 2026-05-17
---

# Godot リファレンス

## 目次

- [概要](#概要)
- [インストール](#インストール)
- [プロジェクト構成](#プロジェクト構成)
- [ノードとシーン](#ノードとシーン)
  - [主なノードの種類](#主なノードの種類)
  - [シーンツリー](#シーンツリー)
- [GDScript 基礎](#gdscript-基礎)
  - [変数・型](#変数型)
  - [関数](#関数)
  - [制御構文](#制御構文)
  - [ライフサイクル関数](#ライフサイクル関数)
  - [ノードの参照](#ノードの参照)
- [シグナル](#シグナル)
- [入力処理](#入力処理)
- [2D ゲーム基礎](#2d-ゲーム基礎)
  - [座標系](#座標系)
  - [移動パターン](#移動パターン)
  - [物理・コリジョン](#物理コリジョン)
- [リソース・シーン生成](#リソースシーン生成)
- [グループ・Timer](#グループtimer)
- [アニメーション](#アニメーション)
- [UI](#ui)
- [シーン切替・ゲーム管理](#シーン切替ゲーム管理)
- [デバッグ・セーブデータ](#デバッグセーブデータ)
- [エクスポート](#エクスポート)
- [Tips](#tips)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## 概要

Godot は オープンソースの 2D / 3D ゲームエンジン。  
独自のスクリプト言語 GDScript（Python ライクな構文）を使う。C# も対応。  
このメモは **Godot 4.x** を前提としている。

---

## インストール

公式サイト（godotengine.org）からダウンロード。インストール不要で実行ファイルを置くだけで動く。

エディタ起動後、「新しいプロジェクト」でフォルダを指定すれば始められる。

---

## プロジェクト構成

以下の構成はプロジェクトを整理するときの一例。

```
my-game/
├── project.godot       # プロジェクト設定ファイル
├── assets/             # 画像・音声など
├── scenes/             # .tscn ファイル（シーン）
└── scripts/            # .gd ファイル（スクリプト）
```

- **`.tscn`** — シーンファイル。ノードの構成を保存したもの
- **`.gd`** — GDScript ファイル
- **`project.godot`** — 解像度・入力マップ・レンダラーなどの設定

---

## ノードとシーン

Godot の基本単位は **ノード**。ノードをツリー状に組み合わせたものが **シーン**。

- シーン＝再利用可能なパーツ（プレイヤー、敵、UI など）
- シーンは別のシーンに **インスタンス化** して組み込める
- シーンごとにルートノードが 1 つある

### 主なノードの種類

| ノード | 用途 |
|--------|------|
| `Node2D` | 2D の基底。位置・回転・スケールを持つ |
| `Sprite2D` | テクスチャを表示する |
| `CharacterBody2D` | プレイヤー・NPCなど動くキャラクター向け |
| `StaticBody2D` | 地面・壁など動かない物体 |
| `RigidBody2D` | 物理演算で動く物体 |
| `Area2D` | 当たり判定のみ（ダメージ範囲・アイテム取得など） |
| `CollisionShape2D` | 当たり判定の形状を定義（Body/Area にセットで使う） |
| `AnimationPlayer` | アニメーション再生 |
| `Camera2D` | カメラ |
| `CanvasLayer` | UI を常にカメラ位置に関わらず表示する |
| `Control` | UI 系ノードの基底（Button、Label、TextureRect など） |
| `AudioStreamPlayer2D` | 2D 空間上の音声再生 |
| `Timer` | 一定時間後に処理を発火させる |

### シーンツリー

```
Main (Node2D)
├── Player (CharacterBody2D)
│   ├── Sprite2D
│   └── CollisionShape2D
├── TileMapLayer
└── HUD (CanvasLayer)
    └── HealthBar (ProgressBar)
```

---

## GDScript 基礎

### 変数・型

```gdscript
var speed = 200.0          # 型推論
var name: String = "Player"
var hp: int = 100
const MAX_HP = 100         # 定数

@export var move_speed: float = 200.0  # エディタから編集可能にする
```

### 関数

```gdscript
func greet(name: String) -> String:
    return "Hello, " + name

func _on_hit(damage: int) -> void:
    hp -= damage
```

関数名の先頭 `_` は「内部用・コールバック用」という命名習慣としてよく使われる。特に `_on_ノード名_シグナル名` は、シグナル接続時に自動生成される関数名の形。

ただし、`_on_hit()` のような自作名はただの慣習で、名前自体に特別な効果はない。一方で `_ready()`、`_process()`、`_physics_process()` などは Godot が自動で呼ぶ特別な関数なので、名前を変えると呼ばれない。

### 制御構文

```gdscript
# 条件分岐。上から順に条件を判定する
if hp <= 0:
    die()
elif hp < 30:
    print("Low HP")
else:
    pass

# 繰り返し。range(5) は 0〜4 を順に返す
for i in range(5):
    print(i)

# 条件が true の間、繰り返す
while is_running:
    move()

# 値のパターンに応じて処理を分ける
match state:
    "idle":
        play_idle()
    "run":
        play_run()
    _:           # default
        pass
```

### ライフサイクル関数

これらは `Node` クラスに定義された仮想メソッドのオーバーライド。GDScript に `override` キーワードはなく、同名の関数を定義するだけで Godot が自動で呼んでくれる。

```gdscript
func _ready() -> void:
    # ノードがシーンツリーに入ったとき（初期化）
    pass

func _process(delta: float) -> void:
    # 毎フレーム呼ばれる（描画・入力チェックなど）
    pass

func _physics_process(delta: float) -> void:
    # 物理フレームごとに呼ばれる（移動・物理演算はここ）
    pass
```

`delta` はひとつ前のフレームからの経過秒数。速度に掛けることでフレームレート非依存な動きになる。

### ノードの参照

```gdscript
# $ ショートカット（子ノード名で直接取得）
$Sprite2D.visible = false
$AnimationPlayer.play("run")

# 相対パスで取得
var node = get_node("子ノード名")
var node = get_node("../兄弟ノード")

# 型アノテーション付きで宣言しておく方法（推奨）
@onready var sprite: Sprite2D = $Sprite2D
```

---

## シグナル

イベント駆動の仕組み。ノード間を疎結合にするために使う。
たとえば Button の `pressed` シグナルを接続すると、「ボタンが押されたときにこの関数を呼ぶ」という処理を書ける。

```gdscript
# StartButton の pressed シグナルに接続された関数
func _on_start_button_pressed() -> void:
    get_tree().change_scene_to_file("res://scenes/game.tscn")
```

エディタでは、Button ノードを選択して「ノード」タブから `pressed()` をダブルクリックすると、接続先の関数を作成できる。

自分でシグナルを定義して、別ノードへ通知することもできる。

```gdscript
# シグナルの定義
signal health_changed(new_hp: int)

# シグナルの発火
health_changed.emit(hp)

# シグナルへの接続（コードで）
$Player.health_changed.connect(_on_health_changed)

func _on_health_changed(new_hp: int) -> void:
    $HUD/HealthBar.value = new_hp
```

接続したシグナルは `_on_ノード名_シグナル名` という関数名で自動生成される。

---

## 入力処理

```gdscript
func _process(delta: float) -> void:
    # アクションの押下チェック（Project Settings > Input Map で定義）
    if Input.is_action_pressed("ui_right"):
        position.x += speed * delta

    # 押した瞬間だけ
    if Input.is_action_just_pressed("jump"):
        jump()

    # 離した瞬間
    if Input.is_action_just_released("attack"):
        end_attack()

    # ベクトルで方向入力をまとめて取得（-1〜1）
    var dir = Input.get_axis("ui_left", "ui_right")
    var dir2d = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
```

入力アクションは **Project Settings → Input Map** で定義する。

---

## 2D ゲーム基礎

### 座標系

- 原点（0, 0）は画面左上
- X 軸は右方向が正、Y 軸は下方向が正

### 移動パターン

```gdscript
# CharacterBody2D での移動
extends CharacterBody2D

const SPEED = 200.0
const JUMP_VELOCITY = -400.0

func _physics_process(delta: float) -> void:
    # 重力
    if not is_on_floor():
        velocity += get_gravity() * delta

    # ジャンプ
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = JUMP_VELOCITY

    # 左右移動
    var direction = Input.get_axis("ui_left", "ui_right")
    velocity.x = direction * SPEED

    move_and_slide()
```

`move_and_slide()` が衝突判定・スライド処理を自動でやってくれる。

### 物理・コリジョン

| ノード | 用途 |
|--------|------|
| `CharacterBody2D` | `move_and_slide()` で自分でコントロール |
| `RigidBody2D` | 物理エンジンに任せる（重力・反発など） |
| `StaticBody2D` | 動かない地形 |
| `Area2D` | 物理は持たず、重なり検知だけする |

```gdscript
# Area2D の重なり検知
func _on_area_entered(area: Area2D) -> void:
    if area.is_in_group("bullet"):
        take_damage(10)

func _on_body_entered(body: Node2D) -> void:
    if body.is_in_group("enemy"):
        take_damage(10)
```

`area_entered` は `Area2D` 同士の重なり検知、`body_entered` は `CharacterBody2D`、`RigidBody2D`、`StaticBody2D` などの物理ボディ検知に使う。

コリジョンレイヤーとマスクで「どのノード同士が衝突・検知するか」を制御できる。

---

## リソース・シーン生成

画像、音声、シーンなどのファイルは **リソース** として扱う。パスはプロジェクトルートを表す `res://` から書く。

```gdscript
# 起動時に読み込む。常に使うもの向け
const BULLET_SCENE = preload("res://scenes/bullet.tscn")

# 実行時に読み込む。条件付きで使うもの向け
var enemy_scene = load("res://scenes/enemy.tscn")
```

`.tscn` を読み込むと `PackedScene` になる。`instantiate()` でノードを生成し、`add_child()` でシーンツリーに追加する。

```gdscript
@export var bullet_scene: PackedScene

func shoot() -> void:
    var bullet = bullet_scene.instantiate()
    bullet.global_position = global_position
    get_tree().current_scene.add_child(bullet)
```

`preload()` はパス間違いに早く気づけるので、固定で使うシーンや画像に向いている。`load()` は必要になったタイミングで読みたい場合に使う。

---

## グループ・Timer

### グループ

グループは、複数のノードを名前でまとめて扱う仕組み。敵、弾、アイテムなどをまとめて処理したいときに便利。

```gdscript
func _ready() -> void:
    add_to_group("enemy")

func damage_all_enemies() -> void:
    for enemy in get_tree().get_nodes_in_group("enemy"):
        enemy.take_damage(10)

func _on_body_entered(body: Node2D) -> void:
    if body.is_in_group("player"):
        body.take_damage(10)
```

### Timer

`Timer` ノードは一定時間後や一定間隔で処理したいときに使う。`timeout` シグナルに処理をつなぐ。

```gdscript
func _on_spawn_timer_timeout() -> void:
    spawn_enemy()
```

一度だけ待ちたい場合は、ノードを置かずに `create_timer()` を使える。

```gdscript
func flash_damage() -> void:
    $Sprite2D.modulate = Color.RED
    await get_tree().create_timer(0.2).timeout
    $Sprite2D.modulate = Color.WHITE
```

---

## アニメーション

`AnimationPlayer` ノードを使う。エディタでキーフレームを打つか、コードで再生。

```gdscript
@onready var anim: AnimationPlayer = $AnimationPlayer

func _physics_process(delta: float) -> void:
    if velocity.x != 0:
        anim.play("run")
    else:
        anim.play("idle")
```

複雑な状態管理には `AnimationTree`（ステートマシン）が便利。

---

## UI

UI ノードは `CanvasLayer` の下に置くとカメラに追従せず常に表示される。

よく使うノード：

| ノード | 用途 |
|--------|------|
| `Label` | テキスト表示 |
| `Button` | ボタン |
| `TextureRect` | 画像表示 |
| `ProgressBar` | HP バーなど |
| `VBoxContainer` / `HBoxContainer` | 縦・横に並べるレイアウト |
| `MarginContainer` | 余白付きレイアウト |

```gdscript
# ラベルのテキストを更新
$HUD/ScoreLabel.text = "Score: %d" % score

# ボタンのシグナルで処理
func _on_start_button_pressed() -> void:
    get_tree().change_scene_to_file("res://scenes/game.tscn")
```

アンカーとコンテナを組み合わせると解像度に依存しないレイアウトにできる。

---

## シーン切替・ゲーム管理

```gdscript
# シーンを切り替える
get_tree().change_scene_to_file("res://scenes/game_over.tscn")

# ゲームを終了
get_tree().quit()

# ゲームを一時停止
get_tree().paused = true
```

グローバルな状態管理には **Autoload**（シングルトン）を使う。  
Project Settings → Autoload でスクリプトを登録すると、どこからでも `GameManager.score` のように参照できる。

```gdscript
# autoload/game_manager.gd
extends Node

var score: int = 0
var lives: int = 3
```

---

## デバッグ・セーブデータ

### デバッグ

まずは `print()` で値を確認する。ノード参照、座標、速度、シグナルが呼ばれているかを見るだけでも原因を絞りやすい。

```gdscript
print("hp:", hp)
print("player position:", global_position)
```

よく使う確認:

- Debugger パネルでエラー、警告、スタックトレースを見る
- Remote Scene Tree で実行中のノード構成を見る
- **Debug → Visible Collision Shapes** で当たり判定の形を表示する

### セーブデータ

ユーザーごとの保存データは `user://` に書く。簡単な設定や進行状況は JSON にすると扱いやすい。

```gdscript
const SAVE_PATH = "user://save.json"

func save_game() -> void:
    var data = {
        "score": score,
        "lives": lives,
    }
    var file = FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    file.store_string(JSON.stringify(data))

func load_game() -> void:
    if not FileAccess.file_exists(SAVE_PATH):
        return

    var file = FileAccess.open(SAVE_PATH, FileAccess.READ)
    var data = JSON.parse_string(file.get_as_text())
    score = data.get("score", 0)
    lives = data.get("lives", 3)
```

---

## エクスポート

**Project → Export** でプラットフォームを選んでビルドする。

エクスポートには各プラットフォームの **Export Template** が必要。初回は Export Template Manager から `Download and Install` するか、`.tpz` ファイルを指定してインストールする。

主な出力先：
- Windows / macOS / Linux（デスクトップ）
- Web（HTML5 / WebGL）
- Android / iOS（別途 SDK 設定が必要）

---

## Tips

- **シーンは細かく分ける** — Player、Enemy、Bullet をそれぞれ独立したシーンにすると再利用・管理しやすい
- **シグナルで疎結合に** — `get_node("../../Player")` のようなパス直書きは脆いのでシグナルを使う
- **Autoload はなるべく少なく** — グローバル状態が増えると追いにくくなる
- **`@export` を活用** — 数値や参照をエディタから調整できるようにしておくとデバッグが速い
- **グループを活用** — `add_to_group("enemy")` しておくと `get_tree().get_nodes_in_group("enemy")` で一括取得できる
- **`print()` でデバッグ** — デバッガも使えるが手軽な確認は `print(velocity)` が速い
- **公式ドキュメント** — docs.godotengine.org が充実している。クラスリファレンスは F1 キーでエディタから引ける

---

## 用語集

| 用語 | 説明 |
|------|------|
| **ノード（Node）** | Godot の最小構成単位。すべてのオブジェクトはノード |
| **シーン（Scene）** | ノードをツリー状に組み合わせたもの。`.tscn` ファイルに保存される |
| **シーンツリー** | 実行中のシーン全体の階層構造。`get_tree()` でアクセスできる |
| **インスタンス化** | シーンを別のシーンに子として配置すること。プレハブに相当する概念 |
| **GDScript** | Godot 独自のスクリプト言語。Python ライクな構文で書ける |
| **`_ready()`** | ノードがシーンツリーに追加されたときに一度だけ呼ばれる初期化関数 |
| **`_process(delta)`** | 毎フレーム呼ばれる更新関数。描画・UI・入力チェックに使う |
| **`_physics_process(delta)`** | 物理フレームごとに呼ばれる更新関数。移動・物理演算はここに書く |
| **delta** | 前フレームからの経過秒数。速度に掛けてフレームレート非依存にする |
| **シグナル（Signal）** | イベント通知の仕組み。ノード間を疎結合につなぐために使う |
| **`@export`** | 変数をエディタのインスペクターから編集可能にするアノテーション |
| **`@onready`** | `_ready()` のタイミングで変数を初期化するアノテーション |
| **Autoload** | どのシーンからでも参照できるグローバルなシングルトン。Project Settings で登録する |
| **`CharacterBody2D`** | `move_and_slide()` を使って自分でコントロールするキャラクター用ノード |
| **`RigidBody2D`** | 物理エンジンに動きを任せるノード。重力・反発などが自動でかかる |
| **`Area2D`** | 物理的な衝突は持たず、重なり（侵入・退出）の検知だけを行うノード |
| **コリジョンレイヤー / マスク** | どのノード同士が衝突・検知するかを制御するビットフラグ設定 |
| **`move_and_slide()`** | `CharacterBody2D` の移動メソッド。壁・床との衝突を自動で処理する |
| **TileMapLayer** | タイルセット画像を使ってマップを効率的に配置するノード（Godot 4.3 以降） |
| **CanvasLayer** | カメラに追従しない描画レイヤー。HUD・UI を常に画面に表示するために使う |
| **Input Map** | キー・ボタンに「アクション名」を割り当てる設定。Project Settings で管理する |
| **Resource** | 画像、音声、シーンなど Godot が読み込んで扱うデータ |
| **PackedScene** | `.tscn` を読み込んだリソース。`instantiate()` でノードを生成できる |
| **グループ** | ノードを名前で分類する仕組み。`get_nodes_in_group()` でまとめて取得できる |
| **Timer** | 一定時間後や一定間隔で `timeout` を発火するノード |
| **Remote Scene Tree** | 実行中のシーンツリーをエディタから確認するデバッグ機能 |
| **Export Template** | 各プラットフォーム向けにビルドするために必要なテンプレートファイル |
| **`res://`** | プロジェクトルートを指す仮想パス。ファイル参照に使う |
| **`user://`** | セーブデータなどユーザー固有のデータを書き込む領域を指す仮想パス |

---

## リンク集

### 公式

- [Godot 公式サイト](https://godotengine.org/)
- [公式ドキュメント（Godot 4）](https://docs.godotengine.org/en/stable/)
- [公式ドキュメント 日本語版（Godot 4.4）](https://docs.godotengine.org/ja/4.4/index.html)
- [クラスリファレンス](https://docs.godotengine.org/en/stable/classes/)
- [GitHub リポジトリ](https://github.com/godotengine/godot)
- [公式デモプロジェクト集](https://github.com/godotengine/godot-demo-projects)

### 学習

- [公式チュートリアル（Your first 2D game）](https://docs.godotengine.org/en/stable/getting_started/first_2d_game/)
- [GDScript リファレンス](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html)
- [GDQuest（チュートリアル動画・無料コース）](https://www.gdquest.com/)
- [KidsCanCode（初心者向け解説）](https://kidscancode.org/godot_recipes/)

### アセット・コミュニティ

- [Godot Asset Library](https://godotengine.org/asset-library/asset)
- [Reddit r/godot](https://www.reddit.com/r/godot/)
- [公式 Discord](https://discord.gg/4JBkykG)

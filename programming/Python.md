---
tags:
  - Python
  - Programming
created: 2026-05-24
updated: 2026-05-25
---

# Python 初心者向けリファレンス

## 目次

- [概要](#概要)
- [型・変数](#型変数)
  - [基本型](#基本型)
  - [変数宣言](#変数宣言)
  - [リスト](#リスト)
  - [タプル](#タプル)
  - [辞書](#辞書)
  - [セット](#セット)
  - [型変換](#型変換)
- [文字列操作](#文字列操作)
- [制御構文](#制御構文)
  - [条件分岐](#条件分岐)
  - [繰り返し](#繰り返し)
  - [内包表記](#内包表記)
- [関数](#関数)
  - [基本](#基本)
  - [デフォルト引数・キーワード引数](#デフォルト引数キーワード引数)
  - [可変長引数](#可変長引数)
  - [ラムダ](#ラムダ)
  - [スコープ](#スコープ)
  - [イテレータ・ジェネレータ](#イテレータジェネレータ)
- [型ヒント](#型ヒント)
- [クラス](#クラス)
  - [定義と初期化](#定義と初期化)
  - [継承](#継承)
  - [特殊メソッド](#特殊メソッド)
  - [dataclass](#dataclass)
- [モジュール・パッケージ](#モジュールパッケージ)
- [エラー処理](#エラー処理)
- [ファイル操作](#ファイル操作)
- [非同期処理](#非同期処理)
- [よく使う標準ライブラリ](#よく使う標準ライブラリ)
- [仮想環境・パッケージ管理](#仮想環境パッケージ管理)
  - [pyproject.toml](#pyprojecttoml)
- [テスト・品質ツール](#テスト品質ツール)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## 概要

Python は読みやすさを重視したインタープリタ型言語。Web 開発、データサイエンス、機械学習、スクリプト自動化など幅広い分野で使われる。  
このメモは **Python 3.10 以降** を前提としている。

---

## 型・変数

### 基本型

| 型 | 説明 | 例 |
|----|------|----|
| `int` | 整数（サイズ制限なし） | `42`, `-10` |
| `float` | 浮動小数点（64bit） | `3.14`, `1e-5` |
| `complex` | 複素数 | `1+2j` |
| `bool` | 真偽値（`int` のサブクラス） | `True` / `False` |
| `str` | 文字列（イミュータブル・Unicode） | `"hello"`, `'world'` |
| `bytes` | バイト列（イミュータブル） | `b"hello"` |
| `NoneType` | 値なしを表す型 | `None` |

### 変数宣言

Python に型宣言は不要だが、型ヒント（アノテーション）を書くことで IDE や静的解析ツールが型チェックしてくれる。

```python
# 代入するだけで変数が作られる
x = 10
name = "Alice"

# 型ヒント（実行時には強制されないが、mypy などで静的チェックできる）
age: int = 30
message: str = "hello"

# 複数代入
a, b, c = 1, 2, 3

# スワップ
a, b = b, a

# 定数は慣習的に大文字で書く（言語仕様では定数はない）
MAX_SIZE = 100
PI = 3.14159

# 変数の型を調べる
print(type(42))    # <class 'int'>
print(type("hi"))  # <class 'str'>
```

### リスト

順序付きのミュータブルなコレクション。異なる型の値を混在させられる。

```python
# 作成
nums = [1, 2, 3, 4, 5]
mixed = [1, "hello", True, 3.14]

# インデックスアクセス（0 始まり、負数は末尾から）
print(nums[0])   # 1
print(nums[-1])  # 5

# スライス [start:stop:step]（stop は含まない）
print(nums[1:3])  # [2, 3]
print(nums[::-1]) # [5, 4, 3, 2, 1]（逆順）

# 追加・削除
nums.append(6)         # 末尾に追加
nums.insert(0, 0)      # 指定位置に挿入
nums.remove(3)         # 値で削除（最初の一致）
popped = nums.pop()    # 末尾を取り出して削除
popped = nums.pop(0)   # 指定インデックスを取り出して削除

# 結合・繰り返し
a = [1, 2] + [3, 4]   # [1, 2, 3, 4]
b = [0] * 3            # [0, 0, 0]

# 検索・ソート
print(3 in nums)       # True / False
nums.sort()            # 昇順に並び替え（元のリストを変更）
sorted_nums = sorted(nums, reverse=True)  # 降順（元のリストは変更しない）

# 長さ
print(len(nums))
```

### タプル

順序付きのイミュータブルなコレクション。変更しないデータの組み合わせに使う。

```python
# 作成（括弧は省略可）
point = (1, 2)
rgb = 255, 128, 0

# 要素が1つのタプル（末尾のカンマが必要）
single = (42,)

# アンパック
x, y = point
r, g, b = rgb

# 要素がすべてハッシュ可能なら辞書のキーに使える
coords = {(0, 0): "origin", (1, 0): "right"}
```

### 辞書

キーと値のペア。キーはハッシュ可能な型（`str`, `int`, 要素がすべてハッシュ可能な `tuple` など）。Python 3.7 以降は挿入順が保証される。

```python
# 作成
user = {"name": "Alice", "age": 30}

# アクセス・追加・更新
print(user["name"])      # "Alice"
user["email"] = "a@b.c"  # 追加
user["age"] = 31          # 更新

# キーが存在しない場合のデフォルト値
score = user.get("score", 0)  # キーがなければ 0

# キーの存在確認
print("name" in user)  # True

# 削除
del user["email"]
popped = user.pop("age", None)  # キーがなくても None を返す

# イテレーション
for key in user:
    print(key)

for key, value in user.items():
    print(key, value)

for value in user.values():
    print(value)

# まとめて更新
user.update({"city": "Tokyo", "age": 25})

# 辞書のマージ（Python 3.9 以降）
merged = user | {"extra": True}
```

### セット

順序なし・重複なしのコレクション。集合演算に使う。

```python
# 作成（空のセットは set()、{} は空の辞書になるので注意）
fruits = {"apple", "banana", "cherry"}
empty = set()

# 追加・削除
fruits.add("mango")
fruits.discard("banana")  # なくても例外が出ない
fruits.remove("cherry")   # なければ KeyError

# 集合演算
a = {1, 2, 3}
b = {2, 3, 4}
print(a | b)   # 和集合: {1, 2, 3, 4}
print(a & b)   # 積集合: {2, 3}
print(a - b)   # 差集合: {1}
print(a ^ b)   # 対称差: {1, 4}

# 重複除去によく使う
unique = list(set([1, 2, 2, 3, 3]))
```

### 型変換

```python
int("42")       # 42
int(3.9)        # 3（切り捨て）
float("3.14")   # 3.14
str(100)        # "100"
bool(0)         # False（0, "", [], {}, None は False）
bool("hello")   # True
list((1, 2, 3)) # [1, 2, 3]
tuple([1, 2])   # (1, 2)
set([1, 1, 2])  # {1, 2}
```

---

## 文字列操作

Python の文字列はイミュータブル。加工メソッドは元の文字列を変更せず、新しい文字列を返す。

```python
name = "Alice"
age = 30

# f-string（推奨される文字列埋め込み）
message = f"{name} is {age} years old"

# よく使うメソッド
text = "  hello, python  "
text.strip()              # "hello, python"
text.upper()              # "  HELLO, PYTHON  "
text.replace("python", "world")
text.startswith("  he")   # True
"python" in text          # True

# 分割と結合
items = "red,green,blue".split(",")  # ["red", "green", "blue"]
", ".join(items)                    # "red, green, blue"

# 複数行文字列
sql = """
SELECT *
FROM users
WHERE active = true
"""
```

---

## 制御構文

### 条件分岐

Python はインデントでブロックを表す。`{}`は使わない。

```python
x = 10

if x > 0:
    print("positive")
elif x == 0:
    print("zero")
else:
    print("negative")

# 三項演算子
result = "even" if x % 2 == 0 else "odd"

# match 文（Python 3.10 以降。値だけでなく、データの形にもマッチできる）
command = "quit"
match command:
    case "quit":
        print("終了")
    case "go" | "move":
        print("移動")
    case _:
        print("不明なコマンド")

point = (1, 2)
match point:
    case (0, 0):
        print("原点")
    case (x, y):
        print(f"x={x}, y={y}")
```

### 繰り返し

```python
# for（イテラブルを順に処理）
for i in range(5):        # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):     # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 10, 2): # 0, 2, 4, 6, 8
    print(i)

# リストのイテレーション
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# インデックスと値を同時に取得
for i, fruit in enumerate(fruits):
    print(i, fruit)

# 複数のリストを同時に処理
names = ["Alice", "Bob"]
scores = [90, 80]
for name, score in zip(names, scores):
    print(name, score)

# while
n = 5
while n > 0:
    print(n)
    n -= 1

# break / continue / else
for i in range(10):
    if i == 3:
        continue   # このイテレーションをスキップ
    if i == 7:
        break      # ループを抜ける
else:
    # break で抜けなかった場合に実行される
    print("完了")
```

### 内包表記

リスト、辞書、セット、ジェネレータを簡潔に書くための構文。

```python
# リスト内包表記
squares = [x ** 2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

evens = [x for x in range(20) if x % 2 == 0]
# [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# 辞書内包表記
word_len = {word: len(word) for word in ["apple", "banana"]}
# {"apple": 5, "banana": 6}

# セット内包表記
unique_lens = {len(word) for word in ["apple", "banana", "cherry"]}
# {5, 6}

# ジェネレータ式（括弧を使う。評価が遅延される）
gen = (x ** 2 for x in range(5))
print(next(gen))  # 0（要求するたびに1つずつ計算される）
print(next(gen))  # 1
print(next(gen))  # 4

# for や sum など、イテラブルを受け取る関数にそのまま渡せる
total = sum(x ** 2 for x in range(100))
# 328350
```

---

## 関数

### 基本

```python
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))  # "Hello, Alice!"

# 型ヒント付き
def add(a: int, b: int) -> int:
    return a + b

# 戻り値なし（None が返る）
def say_hello():
    print("Hello!")
```

### デフォルト引数・キーワード引数

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice")             # "Hello, Alice!"
greet("Bob", "Hi")         # "Hi, Bob!"
greet(greeting="Hey", name="Carol")  # キーワード引数（順不同）

# デフォルト引数にミュータブルな値を使うと意図しない動作になる
# NG: デフォルト値はモジュール読み込み時に一度だけ評価される
def bad(items=[]):
    items.append(1)
    return items

# OK: None をデフォルト値にして関数内で初期化
def good(items=None):
    if items is None:
        items = []
    items.append(1)
    return items
```

### 可変長引数

```python
# *args: 余った位置引数をタプルとしてまとめて受け取る
def total(*args):
    return sum(args)

total(1, 2, 3)

# **kwargs: 余ったキーワード引数を辞書としてまとめて受け取る
def show(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

show(name="Alice", age=30)

# 組み合わせ（通常引数 → *args → キーワード専用引数 → **kwargs）
def func(a, b, *args, flag=False, **kwargs):
    pass # 何もしない

func(1, 2, 3, 4, flag=True, x=10)

# アンパック: リスト・辞書の頭に * / ** をつけると中身をバラして渡せる
nums = [1, 2, 3]
total(*nums)         # total(1, 2, 3) と同じ（* なしだとリストごと1引数になる）

opts = {"name": "Alice", "age": 30}
show(**opts)         # show(name="Alice", age=30) と同じ（** なしだと辞書ごと1引数になる）
```

### ラムダ

`lambda` は1行の無名関数。シンプルなコールバックに使う。

```python
square = lambda x: x ** 2
print(square(5))  # 25

# sorted のキー関数としてよく使う
users = [{"name": "Bob", "age": 30}, {"name": "Alice", "age": 25}]
sorted_users = sorted(users, key=lambda u: u["age"])
```

### スコープ

変数を探す順番は LEGB ルール（Local → Enclosing → Global → Built-in）。通常はローカル変数を使い、必要な場合だけ `global` や `nonlocal` を使う。

```python
x = "global"

def outer():
    x = "outer"

    def inner():
        x = "inner"
        print(x)  # "inner"

# グローバル変数を書き換える
count = 0

def increment():
    global count
    count += 1

# 外側の関数のローカル変数を書き換える
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

### イテレータ・ジェネレータ

イテレータは `next()` で次の値を取り出せるオブジェクト。ジェネレータは `yield` で値を少しずつ返す関数。

```python
def count_up(limit):
    n = 0
    while n < limit:
        yield n
        n += 1

for n in count_up(3):
    print(n)  # 0, 1, 2

gen = count_up(2)
print(next(gen))  # 0
print(next(gen))  # 1
```

大量のデータを一度にリスト化せず処理したいときに便利。

---

## 型ヒント

型ヒントは実行時の型を強制しないが、IDE、mypy、pyright などの静的解析でバグを見つけやすくする。

```python
from collections.abc import Callable, Iterable
from typing import Any

type UserId = int  # Python 3.12 以降の型エイリアス

def greet(name: str) -> str:
    return f"Hello, {name}"

def total(nums: Iterable[int]) -> int:
    return sum(nums)

def apply(value: int, fn: Callable[[int], str]) -> str:
    return fn(value)

names: list[str] = ["Alice", "Bob"]
scores: dict[str, int] = {"Alice": 90}
maybe_name: str | None = None
anything: Any = "type check is disabled here"
```

外部入力や JSON のように実行時まで型が分からない値は、型ヒントだけで安全にはならない。必要に応じて `isinstance()` やバリデーションライブラリで検証する。

Python 3.10 / 3.11 で型エイリアスを明示したい場合は `from typing import TypeAlias` と `UserId: TypeAlias = int` を使えるが、Python 3.12 以降では `type UserId = int` が推奨される。

---

## クラス

### 定義と初期化

```python
class Dog:
    # クラス変数（全インスタンスで共有。Dog.species でアクセスできる）
    species = "Canis familiaris"

    def __init__(self, name: str, age: int):
        # __init__: インスタンス生成後の初期化メソッド。Dog("Rex", 3) したときに自動で呼ばれる
        # self: このインスタンス自身を指す。メソッドの第1引数に必ず書く（名前は慣習）
        # self.name のように書くとインスタンス変数として保存される
        self.name = name
        self.age = age

    # 通常のメソッド（引数あり）
    def is_older_than(self, other_age: int) -> bool:
        # self 経由でインスタンス変数にアクセスする
        return self.age > other_age

    def bark(self) -> str:
        return f"{self.name} says Woof!"

    def __repr__(self) -> str:
        # __repr__: repr() やデバッグ表示で使われる文字列表現
        # !r は repr() を適用する書式（文字列なら引用符つきで表示される）
        return f"Dog(name={self.name!r}, age={self.age})"

dog = Dog("Rex", 3)
print(dog.bark())              # "Rex says Woof!"
print(dog.is_older_than(5))    # False
print(dog.is_older_than(1))    # True
print(dog.name)                # "Rex"
print(dog)                     # Dog(name='Rex', age=3)
print(Dog.species)             # "Canis familiaris"
```

### 継承

```python
class Animal:
    def __init__(self, name: str):
        self.name = name

    def speak(self) -> str:
        # サブクラスで必ずオーバーライドしてほしいメソッドに書く
        # 呼び出されたら「このメソッドは実装されていない」というエラーを投げる
        raise NotImplementedError

class Cat(Animal):
    def speak(self) -> str:
        return f"{self.name} says Meow!"

class Dog(Animal):
    def speak(self) -> str:
        return f"{self.name} says Woof!"

# super() で親クラスのメソッドを呼ぶ
class Kitten(Cat):
    def __init__(self, name: str, color: str):
        super().__init__(name)  # Cat（→ Animal）の __init__ を呼ぶ
        self.color = color

# isinstance で型を確認
cat = Cat("Whiskers")
print(isinstance(cat, Animal))  # True
print(isinstance(cat, Dog))     # False
```

### 特殊メソッド

クラスに定義することで、Python の組み込み演算子や関数と連携させられるメソッド。

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __str__(self):
        return f"({self.x}, {self.y})"

    def __add__(self, other):        # other は右辺のオブジェクト
        return Vector(self.x + other.x, self.y + other.y)

    def __len__(self):
        return 2

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)   # (4, 6)       ← v1.__add__(v2) が呼ばれ、結果を print() するので __str__ が使われる
print(repr(v1))  # Vector(1, 2)  ← __repr__ が使われる
print(len(v1))   # 2
print(v1 == v1)  # True
```

| メソッド | 対応する操作 |
|----------|------------|
| `__init__` | インスタンス初期化 |
| `__repr__` | `repr()`, デバッグ表示 |
| `__str__` | `str()`, `print()` |
| `__len__` | `len()` |
| `__getitem__` | `obj[key]` |
| `__eq__` | `==` |
| `__lt__` | `<` |
| `__add__` | `+` |
| `__contains__` | `in` |
| `__enter__` / `__exit__` | `with` 文 |

### dataclass

`@dataclass` を使うと `__init__`, `__repr__`, `__eq__` などを自動生成できる。

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: float
    y: float
    label: str = ""  # デフォルト値

@dataclass
class Polygon:
    # list や dict をデフォルト値にするときは = [] と直接書けない（全インスタンスで共有されてしまうため）
    # field(default_factory=...) でインスタンスを作るたびに新しいリストを生成する
    vertices: list[Point] = field(default_factory=list)

p = Point(1.0, 2.0)
print(p)  # Point(x=1.0, y=2.0, label='')
```

---

## モジュール・パッケージ

```python
# 標準ライブラリのインポート
import os
import sys
from pathlib import Path
from datetime import datetime, timedelta

# 別名をつける
import numpy as np
import pandas as pd

# 特定の名前だけインポート
from math import sqrt, pi

# 自作モジュール（同じディレクトリの utils.py）
from utils import helper_func

# パッケージ構造
# myproject/
#   __init__.py
#   core/
#     __init__.py
#     engine.py
from myproject.core.engine import Engine

# 条件付き実行（スクリプトとして直接実行された場合のみ）
if __name__ == "__main__":
    print("直接実行")
```

---

## エラー処理

```python
# 基本の try / except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("ゼロ除算エラー")

# 複数の例外を捕捉
try:
    value = int("abc")
except (ValueError, TypeError) as e:
    print(f"変換エラー: {e}")

# else（例外が発生しなかった場合）、finally（必ず実行）
try:
    with open("file.txt", encoding="utf-8") as f:
        content = f.read()
except FileNotFoundError:
    print("ファイルが見つかりません")
else:
    print(content)
finally:
    print("処理終了")  # 例外の有無にかかわらず実行

# 例外を投げる
def divide(a, b):
    if b == 0:
        raise ValueError("ゼロ除算はできません")
    return a / b

# 例外の再送出
try:
    divide(1, 0)
except ValueError:
    print("ログに記録")
    raise  # 同じ例外を再度送出

# カスタム例外
class AppError(Exception):
    def __init__(self, message, code=None):
        super().__init__(message)
        self.code = code

raise AppError("認証失敗", code=401)
```

---

## ファイル操作

`with` 文を使うと、処理の途中で例外が発生してもファイルのクローズなどの後始末が自動で行われる。このような後始末の仕組みをコンテキストマネージャと呼ぶ。

```python
from pathlib import Path

# Path オブジェクト（文字列の結合より安全）
p = Path("data") / "file.txt"

# ファイルの読み書き（with 文でクローズを自動化）
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()        # 全体を文字列で読む
    lines = f.readlines()     # 行ごとにリストで読む

# 行ごとに処理（大きいファイルでもメモリ効率がよい）
with open("file.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(line.strip())

# 書き込み
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Hello, World!\n")

# 追記
with open("log.txt", "a", encoding="utf-8") as f:
    f.write("追加行\n")

# Path でのファイル操作
p = Path("file.txt")
p.write_text("内容", encoding="utf-8")
text = p.read_text(encoding="utf-8")
p.exists()          # ファイルの存在確認
p.is_file()         # ファイルか
p.is_dir()          # ディレクトリか
p.suffix            # ".txt"
p.stem              # "file"
p.parent            # 親ディレクトリ

# ディレクトリの操作
Path("new_dir").mkdir(parents=True, exist_ok=True)
for f in Path(".").glob("**/*.py"):  # 再帰的にファイルを探す
    print(f)
```

---

## 非同期処理

`async` / `await` は、ネットワークアクセスやファイル待ちなどの I/O 待ちを効率よく扱うための構文。CPU を多く使う計算を速くする仕組みではない。

```python
import asyncio

async def fetch(name: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"{name} done"

async def main():
    # 複数の非同期処理を並行して待つ
    results = await asyncio.gather(
        fetch("A", 1.0),
        fetch("B", 1.0),
    )
    print(results)

asyncio.run(main())
```

通常の関数から `await` は使えない。`await` は `async def` の中で使い、プログラムの入口では `asyncio.run()` を使う。

---

## よく使う標準ライブラリ

| モジュール | 主な用途 |
|-----------|---------|
| `os` | OS 機能（環境変数・プロセス・ファイルパス） |
| `sys` | インタープリタ情報・コマンドライン引数 |
| `pathlib` | ファイルパス操作（推奨） |
| `math` | 数学関数 |
| `random` | 乱数生成 |
| `datetime` | 日付・時刻 |
| `time` | タイマー・スリープ |
| `string` | 文字列定数 |
| `re` | 正規表現 |
| `json` | JSON エンコード・デコード |
| `csv` | CSV 読み書き |
| `collections` | Counter, defaultdict, deque など |
| `itertools` | 高度なイテレータ |
| `functools` | 関数操作（lru_cache, reduce など） |
| `dataclasses` | データクラスの自動生成 |
| `typing` | 型ヒント |
| `asyncio` | 非同期 I/O |
| `logging` | ロギング |
| `unittest` | テスト |
| `argparse` | コマンドライン引数の解析 |
| `subprocess` | 外部コマンドの実行 |
| `copy` | オブジェクトのコピー |

```python
# json
import json

data = {"name": "Alice", "scores": [90, 85]}
json_str = json.dumps(data, ensure_ascii=False, indent=2)  # dict → str
parsed = json.loads(json_str)                              # str → dict

with open("data.json", "w") as f:
    json.dump(data, f, ensure_ascii=False)

# collections
from collections import Counter, defaultdict, deque

counter = Counter(["a", "b", "a", "c", "a"])
print(counter.most_common(2))  # [('a', 3), ('b', 1)]

dd = defaultdict(list)
dd["key"].append(1)  # キーがなくても list() が自動で作られる

dq = deque([1, 2, 3], maxlen=5)
dq.appendleft(0)  # 左端に追加
dq.popleft()      # 左端を取り出す

# datetime
from datetime import datetime, timedelta

now = datetime.now()
formatted = now.strftime("%Y-%m-%d %H:%M:%S")
parsed_dt = datetime.strptime("2026-01-01", "%Y-%m-%d")
tomorrow = now + timedelta(days=1)

# re（正規表現）
import re

pattern = re.compile(r"\d+")
print(pattern.findall("abc123def456"))  # ['123', '456']
match = re.search(r"(\w+)@(\w+)", "user@example.com")
if match:
    print(match.group(0))  # "user@example"
    print(match.group(1))  # "user"

# functools
from functools import lru_cache

@lru_cache(maxsize=None)  # 計算結果をキャッシュする
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

# logging
import logging

logging.basicConfig(level=logging.INFO)
logging.info("処理を開始しました")
```

---

## 仮想環境・パッケージ管理

```sh
# 仮想環境の作成と有効化
python -m venv .venv

# Windows
.venv\Scripts\activate

# Mac / Linux
source .venv/bin/activate

# パッケージのインストール・アンインストール
pip install requests
pip install requests==2.31.0  # バージョン指定
pip uninstall requests

# インストール済みパッケージの一覧
pip list
pip freeze > requirements.txt  # ファイルに書き出す

# requirements.txt から一括インストール
pip install -r requirements.txt

# 仮想環境の無効化
deactivate
```

`uv` を使ったパッケージ管理。

```sh
uv init myproject     # プロジェクト作成
uv add requests       # パッケージ追加
uv run main.py        # スクリプトを実行する
uv sync               # pyproject.toml / uv.lock に従って環境を同期
```

### pyproject.toml

`pyproject.toml` は Python プロジェクトの標準的な設定ファイル。依存関係、ビルド設定、ruff や pytest などのツール設定をまとめられる。

```toml
[project]
name = "myproject"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
  "requests>=2.31.0",
]

[dependency-groups]
dev = [
  "pytest",
  "ruff",
  "mypy",
]

[tool.ruff]
line-length = 100

[tool.pytest.ini_options]
testpaths = ["tests"]
```

---

## テスト・品質ツール

Python の開発では、テスト、フォーマット、Lint、型チェックを組み合わせると保守しやすくなる。

| ツール | 用途 |
|--------|------|
| `pytest` | テストフレームワーク |
| `unittest` | 標準ライブラリのテストフレームワーク |
| `ruff` | 高速な Linter / Formatter |
| `black` | コードフォーマッタ |
| `mypy` | 静的型チェック |
| `pyright` | 静的型チェック |

```python
# tests/test_math.py
def add(a: int, b: int) -> int:
    return a + b

def test_add():
    assert add(1, 2) == 3
```

```sh
pytest
ruff check .
ruff format .
mypy .
```

---

## 用語集

| 用語 | 説明 |
|------|------|
| インタープリタ | Python コードを逐次実行するプログラム。コンパイルなしに動く |
| 動的型付け | 変数の型は実行時に決まる。同じ変数に異なる型の値を代入できる |
| イミュータブル | 作成後に変更できないオブジェクト。`str`, `int`, `tuple` など |
| ミュータブル | 作成後に変更できるオブジェクト。`list`, `dict`, `set` など |
| イテラブル | `for` 文で使える要素を順に返せるオブジェクト |
| ジェネレータ | `yield` を使って値を順に生成する関数。一度に全部を作らないのでメモリ効率が良い |
| デコレータ | 関数やクラスを修飾して機能を追加する仕組み。`@` で記述する |
| 内包表記 | リスト・辞書・セット・ジェネレータを一行で作る構文 |
| スコープ | 変数が参照できる範囲。LEGB ルール（Local → Enclosing → Global → Built-in）で決まる |
| GIL | Global Interpreter Lock。CPython での同時実行を制限するロック。I/O バウンドな処理はスレッドが有効 |
| `__dunder__` | 特殊メソッド（ダンダー）。Python の組み込み操作に連携させるために定義する |
| `None` | 値がないことを表すオブジェクト。他言語の `null` に相当 |
| `self` | インスタンスメソッドの第1引数。インスタンス自身を指す（名前は慣習） |
| 仮想環境 | プロジェクトごとにパッケージを分離するための独立した Python 環境 |
| REPL | Read-Eval-Print Loop。`python` を引数なしで起動すると使える対話的実行環境 |

---

## リンク集

- [公式ドキュメント（日本語）](https://docs.python.org/ja/3/)
- [Python Tutorial](https://docs.python.org/ja/3/tutorial/)
- [標準ライブラリリファレンス](https://docs.python.org/ja/3/library/)
- [Real Python](https://realpython.com/)
- [Python Cheat Sheet](https://www.pythoncheatsheet.org/)

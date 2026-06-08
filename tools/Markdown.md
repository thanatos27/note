---
tags:
  - Markdown
  - Tools
created: 2026-05-27
updated: 2026-05-29
---

# Markdown リファレンス

## 目次

- [見出し](#見出し)
- [テキスト装飾](#テキスト装飾)
- [リスト](#リスト)
- [リンク・画像](#リンク画像)
- [コード](#コード)
- [テーブル](#テーブル)
- [引用](#引用)
- [区切り線](#区切り線)
- [HTML](#html)
- [GitHub Flavored Markdown (GFM)](#github-flavored-markdown-gfm)
- [エスケープ](#エスケープ)
- [リンク集](#リンク集)

> Markdown は処理系により対応記法が異なる。ここでは基本記法に加えて、GitHub・GitLab・Obsidian などでよく使われる GFM 系の拡張も扱う。

---

## 見出し

```markdown
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

代替記法（H1・H2のみ）:

```markdown
見出し1
=======

見出し2
-------
```

---

## テキスト装飾

| 記法 | 結果 |
|------|------|
| `**太字**` または `__太字__` | **太字** |
| `*斜体*` または `_斜体_` | *斜体* |
| `***太字斜体***` | ***太字斜体*** |
| `~~打ち消し~~` | ~~打ち消し~~（GFM） |
| `` `インラインコード` `` | `インラインコード` |

---

## リスト

### 箇条書き

```markdown
- 項目1
- 項目2
  - ネスト（スペース2つ）
  - ネスト
- 項目3
```

`-` の代わりに `*` や `+` も使用可。

### 番号付き

```markdown
1. 項目1
2. 項目2
   1. ネスト
3. 項目3
```

多くの処理系では、2行目以降の番号は自動的に調整される（`1. 1. 1.` でも連番表示される）。最初の番号は開始番号として使われることがある。

### タスクリスト（GFM）

```markdown
- [x] 完了
- [ ] 未完了
```

---

## リンク・画像

### インラインリンク

```markdown
[テキスト](https://example.com)
[テキスト](https://example.com "タイトル")
```

### 参照リンク

```markdown
[テキスト][ref]

[ref]: https://example.com "タイトル"
```

### 自動リンク

```markdown
<https://example.com>
<user@example.com>
```

### 画像

```markdown
![alt テキスト](image.png)
![alt テキスト](image.png "タイトル")
```

画像にリンクを付ける場合:

```markdown
[![alt テキスト](image.png)](https://example.com)
```

### 見出しリンク

見出しには自動で ID が付くことが多い。

```markdown
[見出しへ移動](#見出し)
```

日本語・記号・重複見出しの ID 生成ルールは処理系により異なるため、プレビューで確認する。

---

## コード

### インラインコード

```markdown
`コード`
```

バッククォートを含む場合は2つで囲む: `` ` ``

### コードブロック

````markdown
```言語名
コード
```
````

言語名を省略するとシンタックスハイライトなし。主な言語名: `js`, `ts`, `python`, `go`, `bash`, `json`, `yaml`, `sql`, `html`, `css`

インデント記法（スペース4つまたはタブ）:

```markdown
    コード行1
    コード行2
```

---

## テーブル

GFM などの拡張で使える。

```markdown
| 左揃え | 中央揃え | 右揃え |
|:-------|:--------:|-------:|
| セル   |   セル   |   セル |
| セル   |   セル   |   セル |
```

- ヘッダー行とボディは `---` で区切る
- `:` の位置で揃えを指定
- セル内の `|` は `\|` でエスケープ

---

## 引用

```markdown
> 引用テキスト
> 複数行

> ネスト
>> 二重引用
```

引用ブロック内に他の記法も使用可。

---

## 区切り線

```markdown
---
***
___
```

---

## HTML

Markdownの中にHTMLを直接記述できる。

```markdown
<details>
<summary>折りたたみ</summary>
内容
</details>

<br>（改行）

<kbd>Ctrl</kbd> + <kbd>C</kbd>（キー表示）
```

> ブロックレベルHTML要素の前後に空行を入れると、Markdown と HTML の解釈が安定しやすい。

---

## GitHub Flavored Markdown (GFM)

GitHub・GitLab等で使える拡張記法。

### タスクリスト

```markdown
- [x] 完了タスク
- [ ] 未完了タスク
```

### 打ち消し線

```markdown
~~削除テキスト~~
```

### テーブル

標準Markdownに含まれないが GFM では必須機能。

### 脚注

```markdown
本文中の脚注参照[^1]

[^1]: 脚注の内容
```

### 改行

通常の改行は同じ段落として扱われる。明示的に改行したい場合は、行末にスペース2つ、バックスラッシュ、または `<br>` を使う。

```markdown
1行目  
2行目

1行目\
2行目

1行目<br>
2行目
```

### メンション・参照

```markdown
@username          # ユーザーメンション
#123               # Issue / PR 参照
owner/repo#123     # 別リポジトリの Issue 参照
SHA                # コミット参照
```

### 絵文字

```markdown
:smile: :rocket: :white_check_mark:
```

---

## エスケープ

バックスラッシュ `\` でエスケープできる文字:

```
\ ` * _ { } [ ] ( ) # + - . !
```

例:

```markdown
\*アスタリスク\*  →  *アスタリスク*（斜体にならない）
```

---

## リンク集

### 公式・仕様

- [Markdown: Syntax](https://daringfireball.net/projects/markdown/syntax) - John Gruber による元の Markdown 構文ドキュメント
- [CommonMark Spec](https://spec.commonmark.org/) - Markdown の曖昧さを減らすための標準仕様
- [GitHub Flavored Markdown Spec](https://github.github.com/gfm/) - GFM の仕様

### 主要サービス・ガイド

- [GitHub Docs: Basic writing and formatting syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) - GitHub 上で使える Markdown 記法
- [GitLab Docs: GitLab Flavored Markdown](https://docs.gitlab.com/user/markdown/) - GitLab 上で使える Markdown 記法
- [Obsidian Help: Basic formatting syntax](https://help.obsidian.md/syntax) - Obsidian の基本的な Markdown 記法
- [Markdown Guide: Basic Syntax](https://www.markdownguide.org/basic-syntax/) - 実用例がまとまった定番ガイド

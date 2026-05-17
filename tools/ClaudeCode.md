---
tags:
  - ClaudeCode
  - AI
created: 2026-05-17
updated: 2026-05-17
---

# Claude Code リファレンス

## 目次

- [概要](#概要)
- [インストール・起動](#インストール起動)
- [基本操作](#基本操作)
  - [キーボードショートカット](#キーボードショートカット)
- [スラッシュコマンド](#スラッシュコマンド)
- [カスタムスラッシュコマンド](#カスタムスラッシュコマンド)
- [CLAUDE.md による設定](#claudemd-による設定)
- [設定ファイル（settings.json）](#設定ファイルsettingsjson)
  - [パーミッション](#パーミッション)
  - [Hooks](#hooks)
- [MCP サーバー](#mcp-サーバー)
- [メモリシステム](#メモリシステム)
- [エージェント・サブエージェント](#エージェントサブエージェント)
- [Skills / Plugins / Extensions](#skills--plugins--extensions)
- [ヘッドレスモード](#ヘッドレスモード)
- [GitHub Actions](#github-actions)
- [モデル](#モデル)
- [セッション管理・会話の引き継ぎ](#セッション管理会話の引き継ぎ)
- [Tips](#tips)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## 概要

Claude Code は Anthropic が提供する AI コーディング CLI ツール。  
ターミナル・IDE（VS Code / JetBrains）・Web（claude.ai/code）から使える。  
ファイル読み書き、シェル実行、Git 操作、Web 検索などのツールを持つ。

---

## インストール・起動

```bash
npm install -g @anthropic-ai/claude-code
claude          # 起動
claude <file>   # ファイルを渡して起動
claude -p "..."  # ワンショット実行（非インタラクティブ）
```

---

## 基本操作

- メッセージを送るだけで OK。自然言語で指示する
- 長いコンテキストは自動的に圧縮される
- `Ctrl+C` で現在の処理をキャンセル（会話は続く）

### キーボードショートカット

| キー | 動作 |
|------|------|
| `Enter` | メッセージ送信 |
| `Shift+Enter` | 改行 |
| `↑` / `↓` | 入力履歴の前後 |
| `Ctrl+C` | 処理キャンセル |
| `Ctrl+L` | 画面クリア |

---

## スラッシュコマンド

| コマンド | 説明 |
|----------|------|
| `/help` | ヘルプ表示 |
| `/clear` | 会話をリセット |
| `/compact` | コンテキストを圧縮して要約 |
| `/config` | テーマ・モデルなどの設定 UI |
| `/model` | 使用モデルを選択・変更 |
| `/agents` | カスタムサブエージェントを管理 |
| `/review` | 現在ブランチの PR レビュー |
| `/init` | CLAUDE.md を自動生成 |
| `/memory` | メモリ内容の確認・編集 |
| `/mcp` | MCP サーバーの確認 |
| `/permissions` | 現在のパーミッション設定確認 |
| `/cost` | 現セッションのトークン使用量・コスト |
| `/status` | ステータスライン設定 |
| `/doctor` | インストール状態の診断 |
| `/login` / `/logout` | Anthropic アカウントの切替・ログアウト |
| `/add-dir` | Claude がアクセスできる作業ディレクトリを追加 |
| `/vim` | Vim モード切替 |
| `/bug` | バグレポート送信 |

---

## カスタムスラッシュコマンド

繰り返し使う作業手順は、`.claude/commands/` に Markdown ファイルとして置くと `/コマンド名` で呼び出せる。

```text
.claude/
└── commands/
    ├── review.md        # /review
    └── docs/
        └── update.md    # /docs:update
```

例:

```markdown
# /docs:update

次の観点でドキュメントを更新する。

1. 現在の実装との差分を確認する
2. 古い説明を直す
3. 変更点を短くまとめる
```

コマンド名の後ろに書いたテキストは引数として渡せる。作業の型を固定したい場合はカスタムコマンド、必要な知識やファイルが多い場合は Skill 化を検討する。

---

## CLAUDE.md による設定

プロジェクトの振る舞いを自然言語で指定するファイル。  
配置場所により優先度が変わる。

| ファイル | 適用範囲 |
|----------|----------|
| `~/.claude/CLAUDE.md` | グローバル（全プロジェクト） |
| `<project>/CLAUDE.md` | プロジェクト共有（Git 管理） |
| `<project>/CLAUDE.local.md` | プロジェクトローカル（非推奨。個人設定は import 推奨） |
| `C:\ProgramData\ClaudeCode\CLAUDE.md` | 企業管理ポリシー（Windows） |

よく書く内容の例：
- コーディング規約（命名規則、コメントの書き方）
- 使用技術スタック
- テストの実行方法
- やってはいけないこと（`--no-verify` 禁止など）

`CLAUDE.md` は `@path/to/file` で他のファイルを import できる。個人用のプロジェクト設定は `CLAUDE.local.md` より、`@~/.claude/my-project-instructions.md` のように user 領域のファイルを import する運用が推奨される。

---

## 設定ファイル（settings.json）

| ファイル | 適用範囲 |
|----------|----------|
| `~/.claude/settings.json` | グローバル |
| `<project>/.claude/settings.json` | プロジェクト共有 |
| `<project>/.claude/settings.local.json` | プロジェクトローカル |

```jsonc
{
  "model": "sonnet",
  "permissions": {
    "allow": ["Bash(npm run test:*)", "Bash(git log:*)"],
    "deny": []
  },
  "hooks": {
    "PreToolUse": [...],
    "PostToolUse": [...],
    "Stop": [...]
  }
}
```

### パーミッション

ツール呼び出しのたびに承認を求めるかどうかを制御できる。

```jsonc
"permissions": {
  "allow": [
    "Bash(git diff:*)",    // git diff 系
    "Bash(git log:*)",     // git log 系
    "Bash(npm run test)",  // 特定コマンドのみ
    "Read",                // 読み取り全般
    "Edit"                 // 編集全般
  ],
  "deny": [
    "Bash(rm *)"
  ]
}
```

### Hooks

特定のイベントに合わせてシェルコマンドを自動実行できる。

| イベント | タイミング |
|----------|-----------|
| `PreToolUse` | ツール実行前 |
| `PostToolUse` | ツール実行後 |
| `Stop` | Claude の応答が止まったとき |
| `Notification` | 通知発生時 |
| `UserPromptSubmit` | ユーザーがメッセージ送信時 |

```jsonc
"hooks": {
  "Stop": [
    {
      "matcher": "",
      "hooks": [
        {
          "type": "command",
          "command": "powershell -c \"[System.Media.SystemSounds]::Beep.Play()\""
        }
      ]
    }
  ]
}
```

---

## MCP サーバー

Model Context Protocol。外部ツールやデータソースを Claude に接続するプロトコル。

```jsonc
// settings.json
{
  "mcpServers": {
    "my-server": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "my-mcp-package"]
    }
  }
}
```

確認コマンド：`/mcp`

---

## メモリシステム

Claude Code のメモリは主に `CLAUDE.md` ファイルとして管理され、起動時に自動ロードされる。

主な場所：
- Enterprise memory — 組織全体の指示。Windows では `C:\ProgramData\ClaudeCode\CLAUDE.md`
- Project memory — `./CLAUDE.md`
- User memory — `~/.claude/CLAUDE.md`
- Project local memory — `./CLAUDE.local.md`（非推奨。import の利用が推奨）

Claude Code はカレントディレクトリから親ディレクトリへ再帰的に `CLAUDE.md` / `CLAUDE.local.md` を探す。サブツリー内の `CLAUDE.md` は、その配下のファイルを読むタイミングで追加される。

明示的に覚えてほしい場合は、入力を `#` で始めるか `/memory` で編集する。

---

## エージェント・サブエージェント

Claude Code は特定タスク向けのカスタムサブエージェントを利用できる。  
`/agents` で作成・編集・削除でき、明示的に呼び出すことも、説明文に基づいて自動的に使われることもある。

配置場所：
- プロジェクト共有：`.claude/agents/`
- ユーザー共通：`~/.claude/agents/`

サブエージェントは Markdown ファイルで定義し、YAML frontmatter に `name`、`description`、必要に応じて `tools` を書く。

```markdown
---
name: code-reviewer
description: Use proactively after code changes to review quality and security.
tools: Read, Grep, Glob, Bash
---

You are a senior code reviewer. Focus on bugs, security, maintainability, and test gaps.
```

---

## Skills / Plugins / Extensions

Claude Code は、標準機能に加えて Skills、Plugins、Extensions で機能を拡張できる。

| 種類 | 役割 |
|------|------|
| Skill | 特定作業の手順・知識・参照ファイルをまとめる |
| Plugin | commands、agents、hooks、Skills、MCP servers などをまとめて配布する |
| Extension | CLI やエディタ連携など、環境ごとの拡張をまとめる呼び方 |

Claude Code の plugin は、カスタムスラッシュコマンド、サブエージェント、hooks、Skills、MCP サーバーをまとめて導入できる。外部 plugin を入れるときは、README と manifest を確認し、hooks や MCP サーバーが何を実行するかを把握してから使う。

詳細は [CodingAgentExtensions.md](./CodingAgentExtensions.md) にまとめる。

---

## ヘッドレスモード

`-p` フラグで非インタラクティブに実行する。CI・スクリプト・他ツールへの組み込みに使う。

### 基本

```bash
claude -p "src/utils.ts のすべての関数に JSDoc を追加して"
echo "このコードをレビューして" | claude -p  # stdin から渡す
```

### 主なオプション

| オプション | 説明 |
|------------|------|
| `-p` / `--print` | ヘッドレスモードで実行（応答を出力して終了） |
| `--output-format <fmt>` | 出力形式：`text`（デフォルト）/ `json` / `stream-json` |
| `--input-format <fmt>` | 入力形式：`text` / `stream-json` |
| `--max-turns <n>` | エージェントのターン数上限（無限ループ防止） |
| `--allowedTools <tools>` | 許可するツールを指定（複数指定可） |
| `--disallowedTools <tools>` | 使用を禁止するツールを指定 |
| `--model <model>` | セッションで使うモデルを指定（`sonnet` / `opus` / `haiku` など） |
| `--permission-mode <mode>` | 権限モードを指定 |
| `--verbose` | 詳細ログを出力 |
| `--no-color` | カラー出力を無効化（ログ保存時など） |

### 出力形式

```bash
# JSON で受け取ってスクリプトで処理
claude -p "エラーの原因を一言で" --output-format json | jq '.result'

# ストリーミング JSON（逐次処理したいとき）
claude -p "長い処理..." --output-format stream-json
```

### ツールを絞って実行

```bash
# 読み取りと Bash だけ許可（書き込み系を禁止したい場合など）
claude -p "テストを実行して結果を教えて" --allowedTools "Bash,Read"

# コマンド単位で許可
claude -p "履歴を要約して" --allowedTools "Bash(git log:*)" "Read"
```

---

## GitHub Actions

`anthropics/claude-code-action` を使うと GitHub のワークフロー内で Claude Code を動かせる。

### PR コメントで @claude を呼ぶ

PR や Issue のコメントに `@claude` と書くと Claude が応答・実装してくれる。

```yaml
# .github/workflows/claude.yml
name: Claude Code

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

jobs:
  claude:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@beta
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

使い方例：
- `@claude このバグを直して` → コードを修正してコミット
- `@claude レビューして` → コードレビューコメントを投稿
- `@claude テストを追加して` → テストコードを追加してコミット

### ワンショットで自動実行

`claude -p` を使って CI の中で決まった処理を自動化する。

```yaml
- name: Run Claude Code
  run: claude -p "テストが失敗している原因を調べてファイルを修正して"
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### 注意点

- `ANTHROPIC_API_KEY` を GitHub Secrets に登録する必要がある
- Actions の `permissions` で `contents: write` / `pull-requests: write` が必要
- `claude-code-action` は 2025 年時点で `@beta` タグ

---

## モデル

Claude Code では、CLI や settings.json でモデルを指定できる。通常は最新系へ追従しやすいエイリアスを使うと管理しやすい。

| 指定例 | 説明 |
|--------|------|
| `sonnet` | 日常的なコーディング向けのバランス型 |
| `opus` | 高難度タスク向け |
| `haiku` | 軽量・高速タスク向け |
| フルモデル名 | 特定バージョンに固定したい場合 |

`/model`、`--model`、または `settings.json` の `"model"` キーで変更できる。

---

## セッション管理・会話の引き継ぎ

長い作業では、会話をいつ圧縮するか、いつ切り直すかを決めると迷子になりにくい。

- `/compact` は現在の会話を要約してコンテキストを軽くする。長い実装や調査の途中で使う
- `/clear` は会話をリセットする。別タスクへ移るときや、前提を切り離したいときに使う
- 作業の節目では「完了したこと」「未完了のこと」「次に見るファイル」を短く残す
- 次回も使う前提や手順は、チャットだけでなく `CLAUDE.md` や関連ドキュメントに書く
- 大きい作業は「調査」「実装」「レビュー」「修正」のように区切ると引き継ぎやすい
- セッションをまたぐ可能性がある作業では、最後に `git status` と変更ファイルを確認する

---

## Tips

- **CLAUDE.md に「やってはいけないこと」を書く** — 暴走防止に効果的
- **`/compact` を活用** — 長い会話でコンテキストが詰まってきたら使う
- **パーミッションの allowlist を整備する** — よく使うコマンドは事前に許可しておくとストレスが減る（`/fewer-permission-prompts` スキルで自動化も可）
- **Hooks で通知** — 長い処理が終わったときに音や通知を出すと便利
- **`claude -p` でスクリプト組み込み** — ワンショットモードで CI や自動化に使える
- **カスタムスラッシュコマンド** — 繰り返す作業は `.claude/commands/` にテンプレ化
- **個人用メモリは import で分離** — 共有する `CLAUDE.md` から `@~/.claude/...` を参照すると、チーム設定と個人設定を分けやすい

---

## 用語集

| 用語 | 説明 |
|------|------|
| **CLAUDE.md** | Claude に読み込ませるプロジェクト指示ファイル。自然言語でルールや文脈を書く |
| **settings.json** | モデル・パーミッション・Hooks などを JSON で設定するファイル |
| **Hooks** | ツール実行前後などのイベントに合わせて任意のシェルコマンドを自動実行する仕組み |
| **MCP（Model Context Protocol）** | Claude に外部ツールやデータソースを接続するための標準プロトコル |
| **MCP サーバー** | MCP に従って Claude にツールを提供する外部プロセス |
| **ヘッドレスモード** | `-p` フラグで起動する非インタラクティブモード。CI やスクリプトから使う |
| **ワンショット** | 1 回の `-p` 実行で応答を返して終了する使い方 |
| **パーミッション** | Claude がツールを呼ぶ際に都度確認するかどうかを制御する設定 |
| **allowlist / denylist** | 実行を自動許可 / 禁止するツールやコマンドのリスト |
| **サブエージェント** | Claude Code が内部で起動する特化エージェント。並列処理やコンテキスト分離に使う |
| **カスタムエージェント** | `.claude/agents/` に置く Markdown ファイルで定義した独自のサブエージェント |
| **カスタムスラッシュコマンド** | `.claude/commands/` に置く Markdown ファイルで追加できる `/xxx` コマンド |
| **Skill** | 特定作業の手順・知識・参照ファイルをまとめた拡張単位 |
| **Plugin** | commands、agents、hooks、Skills、MCP servers などをまとめて導入する拡張パッケージ |
| **Tool use** | Claude がファイル読み書き・シェル実行などの操作を行う機能の総称 |
| **コンテキスト** | Claude が現在の会話で参照している情報全体。長くなると `/compact` で圧縮する |
| **セッション** | Claude Code を起動してから続いている会話・作業単位 |
| **`/clear`** | 現在の会話をリセットし、新しい前提で始めるコマンド |
| **`/compact`** | 長い会話を要約して、コンテキストを軽くするコマンド |
| **Fast モード** | `/fast` で切替できる、Opus モデルを高速出力するモード |
| **`@` import** | `CLAUDE.md` 内で `@path/to/file` と書くことで別ファイルを読み込む記法 |
| **`claude-code-action`** | GitHub Actions で Claude Code を動かす公式アクション（`anthropics/claude-code-action`） |

---

## リンク集

### 公式ドキュメント

- [Claude Code 公式ドキュメント](https://docs.anthropic.com/en/docs/claude-code/overview)
- [クイックスタート](https://docs.anthropic.com/en/docs/claude-code/quickstart)
- [CLAUDE.md リファレンス](https://docs.anthropic.com/en/docs/claude-code/memory)
- [設定・settings.json リファレンス](https://docs.anthropic.com/en/docs/claude-code/settings)
- [Hooks リファレンス](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [MCP リファレンス](https://docs.anthropic.com/en/docs/claude-code/mcp)
- [GitHub Actions リファレンス](https://docs.anthropic.com/en/docs/claude-code/github-actions)
- [ヘッドレスモード / SDK リファレンス](https://docs.anthropic.com/en/docs/claude-code/sdk)

### リポジトリ

- [claude-code（npm）](https://www.npmjs.com/package/@anthropic-ai/claude-code)
- [claude-code-action（GitHub）](https://github.com/anthropics/claude-code-action)
- [anthropics/skills（公式 Skills）](https://github.com/anthropics/skills)
- [anthropics/claude-plugins-official（公式プラグイン）](https://github.com/anthropics/claude-plugins-official)

### その他

- [Model Context Protocol 公式](https://modelcontextprotocol.io/)
- [MCP サーバー一覧（公式）](https://github.com/modelcontextprotocol/servers)
- [Issues・フィードバック](https://github.com/anthropics/claude-code/issues)

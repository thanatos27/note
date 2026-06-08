---
tags:
  - Codex
  - OpenAI
  - AI
created: 2026-06-01
updated: 2026-06-01
---

# OpenAI Codex リファレンス

> Codex は更新が速い。CLI オプション、モデル、認証、権限まわりは公式ドキュメントと `codex --help` を優先して確認する。

## 目次

- [概要](#概要)
- [インストール](#インストール)
- [認証](#認証)
- [基本的な使い方](#基本的な使い方)
- [非対話実行 codex exec](#非対話実行-codex-exec)
- [権限・サンドボックス](#権限サンドボックス)
- [主要コマンド](#主要コマンド)
- [主なオプション](#主なオプション)
- [AGENTS.md による指示](#agentsmd-による指示)
- [設定ファイル](#設定ファイル)
- [MCP](#mcp)
- [Plugins](#plugins)
- [Skills](#skills)
- [モデル](#モデル)
- [用語集](#用語集)
- [Tips](#tips)
- [Claude Code との比較](#claude-code-との比較)
- [リンク集](#リンク集)

---

## 概要

OpenAI Codex は、OpenAI が提供するソフトウェア開発向けのコーディングエージェント。コードの生成・修正、コードベース理解、レビュー、デバッグ、テスト実行、リファクタリングなどを支援する。

主な利用面:

- **Codex CLI**: ターミナルで使うローカルエージェント
- **Codex IDE extension**: VS Code / Cursor / Windsurf などのエディタ統合
- **Codex app / Web**: アプリやブラウザ上で使う Codex

このメモでは主に Codex CLI を扱う。

- GitHub: [openai/codex](https://github.com/openai/codex)
- ライセンス: Apache 2.0

---

## インストール

公式のインストーラを使う方法:

```bash
# macOS / Linux
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

```powershell
# Windows PowerShell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

パッケージマネージャーを使う方法:

```bash
# npm
npm install -g @openai/codex

# Homebrew
brew install --cask codex
```

インストール後:

```bash
codex
```

---

## 認証

Codex CLI は主に2つの認証方式をサポートする。

| 方式 | 用途 |
|------|------|
| ChatGPT でサインイン | 個人利用・通常の対話利用。ChatGPT プランやワークスペース設定に従う |
| API キーでサインイン | API 課金ベース。CI/CD などのプログラム実行で扱いやすい |

CLI では、有効なセッションがない場合は ChatGPT サインインが既定の導線になる。

API キーを使う場合は OpenAI Platform の API キーを用意する。自動化では環境変数の露出に注意し、リポジトリ由来のスクリプトやテストが同じ環境変数を読める状態にしない。

`codex exec` では一時的に `CODEX_API_KEY` を渡せる。

```bash
CODEX_API_KEY=<api-key> codex exec "triage open bug reports"
```

---

## 基本的な使い方

```bash
# 対話モード
codex

# 作業ディレクトリを指定して起動
codex --cd path/to/project

# モデルを指定
codex --model gpt-5.5

# 画像を添付して相談
codex --image ./screenshot.png
```

CLI オプションはバージョンで変わることがあるため、詳細は手元の `codex --help` と公式ドキュメントを確認する。

---

## 非対話実行 codex exec

`codex exec` は、CI、スクリプト、パイプライン、ワンショット処理向けの非対話モード。

```bash
codex exec "リポジトリ構成を要約して、リスクが高い箇所を5つ挙げて"
```

`codex exec` は進捗を `stderr` に流し、最終メッセージを `stdout` に出す。ほかの CLI ツールと組み合わせやすい。

```bash
git log --oneline -10 | codex exec "リリースノートを作って"
```

stdin 全体をプロンプトとして渡したい場合:

```bash
cat prompt.txt | codex exec -
```

機械処理しやすい出力が必要な場合:

```bash
codex exec --json "リポジトリ構成を要約して"
codex exec "Extract project metadata" --output-schema ./schema.json -o ./metadata.json
```

---

## 権限・サンドボックス

Codex がローカルで実行するコマンドには、ファイルシステムやネットワークへのアクセス権限を設定できる。

代表的なサンドボックス指定:

| 指定 | 説明 |
|------|------|
| `read-only` | 読み取り中心。変更を避けたい調査向け |
| `workspace-write` | ワークスペース内の書き込みを許可 |
| `danger-full-access` | サンドボックス制限を外す。隔離された環境でのみ使う |

```bash
codex exec --sandbox read-only "この変更の影響範囲を調べて"
codex exec --sandbox workspace-write "失敗しているテストを修正して"
```

`--full-auto` は互換用の非推奨フラグとして残っている。新しいスクリプトでは `--sandbox workspace-write` など、明示的な `--sandbox` 指定を使う。

より細かい制御には permission profiles を使う。組み込みプロファイルには `:read-only`、`:workspace`、`:danger-full-access` がある。

---

## 主要コマンド

Codex CLI は、サブコマンドなしで起動すると対話モードになる。ワンショット実行や設定管理ではサブコマンドを使う。

よく使うコマンド:

| コマンド | 用途 |
|----------|------|
| `codex` | 対話モードを起動 |
| `codex "..."` | 初期プロンプト付きで対話モードを起動 |
| `codex exec "..."` / `codex e "..."` | 非対話でタスクを実行 |
| `codex review` | 非対話でコードレビューを実行 |
| `codex login` | 認証情報を設定 |
| `codex logout` | 保存済み認証情報を削除 |
| `codex resume` | 過去の対話セッションを再開 |
| `codex fork` | 過去の対話セッションから分岐 |
| `codex apply` / `codex a` | Codex が生成した最新 diff を `git apply` として適用 |
| `codex doctor` | インストール、設定、認証、実行環境を診断 |
| `codex update` | Codex を最新版へ更新 |
| `codex completion` | shell completion script を生成 |
| `codex help` | ヘルプを表示 |

設定・拡張まわり:

| コマンド | 用途 |
|----------|------|
| `codex mcp list` | 登録済み MCP サーバーを一覧表示 |
| `codex mcp get <name>` | MCP サーバー設定の詳細を表示 |
| `codex mcp add <name> -- <command>` | MCP サーバーを追加 |
| `codex mcp remove <name>` | MCP サーバーを削除 |
| `codex mcp login <name>` | MCP サーバーへ認証 |
| `codex mcp logout <name>` | MCP サーバーの認証を解除 |
| `codex plugin list` | marketplace から利用可能な plugin を一覧表示 |
| `codex plugin add <name>` | configured marketplace から plugin をインストール |
| `codex plugin remove <name>` | インストール済み plugin を削除 |
| `codex plugin marketplace add <source>` | plugin marketplace を追加 |
| `codex plugin marketplace list` | configured marketplace を一覧表示 |
| `codex plugin marketplace upgrade` | marketplace snapshot を更新 |
| `codex plugin marketplace remove <name>` | marketplace を削除 |

その他・上級者向け:

| コマンド | 用途 |
|----------|------|
| `codex app` | Codex desktop app を起動、未導入なら installer を開く |
| `codex mcp-server` | Codex を stdio MCP server として起動 |
| `codex sandbox` | Codex が提供する sandbox 内でコマンドを実行 |
| `codex features` | feature flags を確認 |
| `codex debug` | デバッグ用ツール |
| `codex cloud` | Codex Cloud の task を参照し、変更をローカルへ適用 |
| `codex app-server` | 実験的な app server / 関連 tooling |
| `codex remote-control` | remote control 有効の app-server daemon を管理 |
| `codex exec-server` | 実験的な standalone exec-server service |

TUI 内でよく使うコマンド:

| コマンド | 用途 |
|----------|------|
| `/help` | TUI 内ヘルプを表示 |
| `/mcp` | 有効な MCP サーバーを確認 |
| `/plugins` | plugin browser を開く |
| `/skills` | 利用可能な skill を確認 |
| `$skill-name` | skill を明示的に呼び出す |
| `@plugin-creator` | plugin 作成支援を呼び出す |
| `$skill-creator` | skill 作成支援を呼び出す |
| `$skill-installer` | skill のインストール支援を呼び出す |

---

## 主なオプション

| オプション | 説明 |
|------------|------|
| `--cd`, `-C` | 作業ディレクトリを指定 |
| `--model`, `-m` | 使用モデルを指定 |
| `--sandbox`, `-s` | サンドボックス方針を指定: `read-only` / `workspace-write` / `danger-full-access` |
| `--config`, `-c` | 一時的な設定上書き。例: `-c model=gpt-5.5` |
| `--profile`, `-p` | `$CODEX_HOME/<profile>.config.toml` を重ねる |
| `--image`, `-i` | 画像ファイルを添付 |
| `--json` | `codex exec` で JSON Lines イベントを出力 |
| `--output-last-message`, `-o` | `codex exec` の最終応答をファイルへ出力 |
| `--output-schema` | `codex exec` の最終応答を JSON Schema に従わせる |
| `--skip-git-repo-check` | Git リポジトリ外での `codex exec` 実行を許可 |

危険なオプション:

| オプション | 注意 |
|------------|------|
| `--dangerously-bypass-approvals-and-sandbox`, `--yolo` | 承認とサンドボックスを迂回する。隔離された runner 以外では避ける |
| `--full-auto` | 非推奨の互換フラグ。新規利用では `--sandbox` を使う |

---

## AGENTS.md による指示

`AGENTS.md` は Codex にプロジェクト固有の作業ルールや背景知識を渡すためのファイル。

```markdown
# AGENTS.md

## スタック
- TypeScript / Node.js
- Jest でテスト

## ルール
- `src/` 配下のファイルのみ変更する
- コミットメッセージは日本語で書く
- `--no-verify` は使わない

## テスト実行
npm run test
```

読み込みの概要:

- グローバル: `~/.codex/AGENTS.md` または `~/.codex/AGENTS.override.md`
- プロジェクト: Git ルートから現在ディレクトリまでの `AGENTS.md` / `AGENTS.override.md`
- `AGENTS.override.md` がある場合は、同じ階層の `AGENTS.md` より優先される
- 空ファイルはスキップされ、合計サイズには上限がある
- `project_doc_fallback_filenames` で別名ファイルも指示ファイルとして扱える

---

## 設定ファイル

Codex のユーザー設定は既定で `~/.codex/config.toml` に保存される。プロジェクト単位では `.codex/config.toml` を使える。

```toml
model = "gpt-5.5"
default_permissions = ":workspace"

[features]
shell_snapshot = true
```

設定の優先順位は高い順におおむね以下。

1. CLI flags / `--config`
2. プロジェクト設定: `.codex/config.toml`
3. プロファイル設定: `~/.codex/<profile>.config.toml`
4. ユーザー設定: `~/.codex/config.toml`
5. システム設定
6. 組み込み既定値

プロジェクト設定は、信頼済みプロジェクトでのみ読み込まれる。

---

## MCP

Codex は Model Context Protocol (MCP) をサポートする。外部ツール、ドキュメント、Figma、Playwright、Sentry などを Codex に接続できる。

CLI で追加する例:

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

設定ファイルで追加する例:

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]

[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
bearer_token_env_var = "FIGMA_OAUTH_TOKEN"
```

TUI では `/mcp` で有効な MCP サーバーを確認できる。

---

## Plugins

Plugins は、Skills、アプリ連携、MCP サーバー設定などをまとめて Codex に追加するための配布単位。個人・チーム・リポジトリで再利用したいワークフローや外部サービス連携を、インストールしやすい形にまとめられる。

Plugin に含められるもの:

| 要素 | 用途 |
|------|------|
| Skills | 特定タスク向けの再利用手順 |
| Apps / connectors | GitHub、Slack、Google Drive など外部サービスとの連携 |
| MCP servers | Codex が利用する外部ツールや共有情報源 |
| Hooks | セッション開始時などに動く lifecycle hook |
| Assets | Marketplace 表示用のアイコン、ロゴ、スクリーンショットなど |

使い方:

```bash
codex
/plugins
```

CLI の plugin browser では、marketplace ごとに plugin を探し、詳細を確認してインストール・アンインストール・有効/無効を切り替えられる。

Plugin と Skill の使い分け:

| 目的 | 推奨 |
|------|------|
| 1つのローカル手順を試す | Skill |
| リポジトリ内だけで共有する | Repo-scoped skill または repo marketplace plugin |
| 複数の skills をまとめたい | Plugin |
| MCP 設定や app 連携も一緒に配りたい | Plugin |
| チームや workspace に配布したい | Plugin |

Plugin の基本構成:

```text
my-plugin/
  .codex-plugin/
    plugin.json  # 必須: plugin manifest
  skills/
    my-skill/
      SKILL.md
  hooks/
    hooks.json
  .app.json      # 任意: app / connector mapping
  .mcp.json      # 任意: MCP server configuration
  assets/
```

最小の `plugin.json` 例:

```json
{
  "name": "my-first-plugin",
  "version": "1.0.0",
  "description": "Reusable greeting workflow",
  "skills": "./skills/"
}
```

作成には組み込みの `@plugin-creator` を使える。

```text
@plugin-creator
```

`@plugin-creator` は `.codex-plugin/plugin.json` の作成や、ローカル marketplace への登録を手伝う。

Marketplace:

- Plugin marketplace は、Codex が読める plugin の JSON カタログ
- リポジトリ単位では `$REPO_ROOT/.agents/plugins/marketplace.json`
- 個人用では `~/.agents/plugins/marketplace.json`
- Codex CLI から marketplace を追加する場合は `codex plugin marketplace add` を使う

```bash
codex plugin marketplace add owner/repo
codex plugin marketplace add owner/repo --ref main
codex plugin marketplace add ./local-marketplace-root
codex plugin marketplace list
codex plugin marketplace upgrade
codex plugin marketplace remove marketplace-name
```

インストールされた plugin の有効/無効状態は `~/.codex/config.toml` に保存される。Plugin に含まれる MCP サーバーは、plugin scope の設定で個別に有効化や tool approval policy を調整できる。

注意:

- Plugin は他者のコード・設定・hooks を取り込む単位なので、出所と内容を確認してから有効化する
- Plugin に含まれる hooks は、インストールしただけでは自動的に信頼されない
- ローカルで試行錯誤している段階では、まず Skill として作り、配布したくなったら Plugin 化する

---

## Skills

Skills は、Codex に特定タスク向けの手順・知識・補助スクリプトを追加する仕組み。繰り返し使うワークフローを `SKILL.md` としてまとめておくと、Codex が必要に応じて読み込み、同じ手順を安定して実行しやすくなる。

Skills は Codex CLI、IDE extension、Codex app で利用できる。

Skills と Plugins の違い:

| 項目 | 用途 |
|------|------|
| Skills | ワークフローそのものを書く形式 |
| Plugins | Skills や MCP 設定、アプリ連携などを配布・インストールする単位 |

Skill の基本構成:

```text
my-skill/
  SKILL.md      # 必須: メタデータと手順
  scripts/      # 任意: 実行スクリプト
  references/   # 任意: 参照ドキュメント
  assets/       # 任意: テンプレートや画像など
  agents/
    openai.yaml # 任意: 表示名、依存ツール、呼び出しポリシーなど
```

`SKILL.md` には `name` と `description` が必要。

```markdown
---
name: skill-name
description: この skill をいつ使うべきか、いつ使うべきでないかを具体的に書く。
---

Codex が従う手順を書く。
```

Codex は最初から全 skill の本文を読むわけではなく、名前・説明・ファイルパスを見て必要な skill を選び、選ばれたときに `SKILL.md` を読み込む。明示的に使う場合は、CLI/IDE で `/skills` や `$skill-name` を使う。`description` がタスクに合う場合は、Codex が暗黙的に使うこともある。

保存場所:

| スコープ | 場所 | 用途 |
|----------|------|------|
| Repository | `.agents/skills` | リポジトリや特定ディレクトリ向けの共有 skill |
| User | `$HOME/.agents/skills` | 個人用にどのリポジトリでも使う skill |
| Admin | `/etc/codex/skills` | 共有マシンやコンテナの既定 skill |
| System | Codex に同梱 | OpenAI が提供する組み込み skill |

作成・インストール:

```text
$skill-creator
$skill-installer linear
```

`$skill-creator` は、新しい skill の目的、発火条件、スクリプト要否を確認しながら作成する。`$skill-installer` は組み込み・ curated skill や外部リポジトリの skill を入れるために使う。

無効化する場合は `~/.codex/config.toml` に書く。

```toml
[[skills.config]]
path = "/path/to/skill/SKILL.md"
enabled = false
```

ベストプラクティス:

- 1つの skill は1つの仕事に絞る
- 決定的な処理や外部ツールが必要な場合以外は、まず手順書ベースにする
- `description` は発火条件が分かるように具体的に書く
- 入力、出力、確認手順を明示する
- チームで配布したい場合は plugin 化を検討する

---

## モデル

Codex の既定モデルは、CLI のバージョン、設定、サインイン方式、利用プランによって変わる可能性がある。固定の「デフォルトモデル」をメモに書き切るより、以下を優先して確認する。

- `codex --help`
- `~/.codex/config.toml`
- OpenAI Developers の Codex / Models ドキュメント
- 利用中の ChatGPT プランや API 組織の利用可能モデル

明示的に指定する場合:

```bash
codex --model gpt-5.5
codex exec --model gpt-5.5 "このアルゴリズムを最適化して"
```

---

## 用語集

| 用語 | 意味 |
|------|------|
| Codex | OpenAI のソフトウェア開発向けコーディングエージェント |
| Codex CLI | ターミナル上で動く Codex クライアント |
| Codex IDE extension | VS Code / Cursor / Windsurf などで使う Codex 拡張 |
| Codex app / Web | アプリやブラウザから使う Codex |
| TUI | Terminal User Interface。ターミナル内で動く対話画面 |
| `codex exec` | CI やスクリプト向けの非対話実行コマンド |
| sandbox | Codex が実行するコマンドのファイル・ネットワークアクセスを制限する仕組み |
| `read-only` | 読み取り中心のサンドボックス設定 |
| `workspace-write` | ワークスペース内の書き込みを許可するサンドボックス設定 |
| `danger-full-access` | サンドボックス制限を外す設定。隔離環境向け |
| permission profile | ファイルシステムやネットワークの許可範囲を名前付きで定義する設定 |
| `AGENTS.md` | Codex にプロジェクト固有のルールや背景情報を渡す指示ファイル |
| `AGENTS.override.md` | 同じ階層の `AGENTS.md` より優先される一時・上書き用の指示ファイル |
| `config.toml` | Codex のモデル、権限、MCP などを設定する TOML ファイル |
| `CODEX_HOME` | Codex の設定・認証・指示ファイルなどを置くホームディレクトリを指定する環境変数 |
| MCP | Model Context Protocol。外部ツールやデータソースをエージェントへ接続するためのプロトコル |
| MCP server | Codex から利用できる外部ツールや情報源を提供するサーバー |
| Skills | 特定作業の手順や知識を Codex に追加する仕組み |
| Plugins | Codex に追加機能や MCP サーバーなどをまとめて導入する仕組み |
| JSON Lines / JSONL | 1行に1つの JSON オブジェクトを書く形式。`codex exec --json` で使う |
| output schema | `codex exec` の最終出力を指定した JSON Schema に従わせる仕組み |
| access token | ChatGPT / Enterprise などの権限で Codex を使うための認証トークン |
| API key | OpenAI Platform の従量課金ベースで Codex を使うためのキー |

---

## Tips

- **最初は権限を絞る**: 調査は `read-only`、編集は `workspace-write` から始める
- **`danger-full-access` は隔離環境で使う**: ローカル実機では慎重に扱う
- **`AGENTS.md` で作業ルールを固定**: テストコマンド、変更禁止ファイル、レビュー観点を書く
- **自動化は `codex exec` を使う**: `--json` や `--output-schema` で後続処理しやすくする
- **Git で差分管理**: 作業前にブランチを切り、差分を確認できる状態にする
- **環境変数の漏えいに注意**: CI で API キーをジョブ全体に渡さない

---

## Claude Code との比較

| 比較項目 | Codex | Claude Code |
|----------|--------|-------------|
| 提供元 | OpenAI | Anthropic |
| 主な利用面 | CLI / IDE extension / app / web | CLI / IDE 統合 |
| オープンソース | Codex CLI は Apache 2.0 | プロプライエタリ |
| プロジェクト指示 | `AGENTS.md` | `CLAUDE.md` |
| 非対話実行 | `codex exec` | `-p` など |
| 権限制御 | sandbox / permission profiles | permissions / settings |
| MCP 対応 | あり | あり |
| カスタム拡張 | MCP / Skills / Plugins など | MCP / subagents / commands など |

どちらも更新が速いため、比較表は固定的な優劣ではなく「確認観点」として扱う。

---

## リンク集

- [OpenAI Codex 公式ドキュメント](https://developers.openai.com/codex)
- [Codex CLI command line options](https://developers.openai.com/codex/cli/reference)
- [Codex authentication](https://developers.openai.com/codex/auth)
- [Codex permissions](https://developers.openai.com/codex/permissions)
- [Custom instructions with AGENTS.md](https://developers.openai.com/codex/guides/agents-md)
- [Codex MCP](https://developers.openai.com/codex/mcp)
- [Codex Plugins](https://developers.openai.com/codex/plugins)
- [Build Codex plugins](https://developers.openai.com/codex/plugins/build)
- [Codex Skills](https://developers.openai.com/codex/skills)
- [openai/skills GitHub](https://github.com/openai/skills)
- [openai/codex GitHub](https://github.com/openai/codex)
- [@openai/codex npm](https://www.npmjs.com/package/@openai/codex)

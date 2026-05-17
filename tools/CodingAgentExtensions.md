---
tags:
  - CodingAgent
  - Anthropic
  - OpenAI
  - Gemini
---
# Coding Agent Extensions

Coding Agentでは、特定の作業に必要な手順・知識・ツール連携をまとめたものを、スキル、プラグイン、拡張として追加できる。

大まかには、スキルは「作業手順や専門知識の追加」、プラグインは「Claude Codeなどの環境に機能を追加する仕組み」、拡張は「CLIやエディタにコマンド・MCP・設定をまとめて追加する仕組み」と考えるとよい。

## 導入前の確認

Skills、plugins、extensionsは、単なるプロンプト集ではなく、MCPサーバー、hooks、custom commands、sub-agents、実行スクリプト、ローカル設定、外部サービス認証を含むことがある。

導入前に確認すること:

- 公式・コミュニティ・第三者提供のどれか
- README、manifest、設定ファイルに書かれた権限や実行内容
- MCPサーバー、hooks、bin/script、custom commandの有無
- APIキー、OAuth、GitHub CLIなどの認証が必要か
- 自動更新や外部リポジトリ参照の有無

一覧やマーケットプレイスの内容は頻繁に変わるため、このメモは基本的に「代表例」として扱い、最新情報は各公式リポジトリとドキュメントを確認する。

## 優先ソース

### Anthropic

- [anthropics/skills](https://github.com/anthropics/skills)
  - AnthropicのAgent Skills公式リポジトリ
  - Claude向けのスキル例、document skills、skill template、specがまとまっている
- [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)
  - Anthropic管理のClaude Code Pluginディレクトリ
  - 内製プラグインと外部プラグインが分かれている
- [Agent Skills standard](https://agentskills.io/)
  - Anthropic / OpenAI / Gemini系で共通して参照されるAgent Skillsの標準

### OpenAI

- [OpenAI: Agent Skills - Codex](https://developers.openai.com/codex/skills)
  - CodexのSkills公式ドキュメント
  - Codex CLI、IDE拡張、Codex appで利用可能
  - `$skill-installer` でcurated skillを入れられる
- [openai/skills](https://github.com/openai/skills)
  - OpenAI公式のCodex向けSkills Catalog
  - `.system` はCodexに同梱、`.curated` は追加インストール候補

### Gemini

- [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)
  - Gemini CLI公式リポジトリ
- [Gemini CLI extensions docs](https://github.com/google-gemini/gemini-cli/blob/main/docs/extensions/index.md)
  - Gemini CLI拡張の公式ドキュメント
  - prompts、MCP servers、custom commands、themes、hooks、sub-agents、agent skillsを拡張としてまとめられる
- [gemini-cli-extensions](https://github.com/gemini-cli-extensions)
  - Google Extensions for Gemini CLI
  - Gemini CLI用の公式拡張群

## Anthropic公式系

### anthropics/skills

インストール:

```text
# Anthropic公式の `anthropics/skills` GitHubリポジトリを、Claude Codeのプラグイン取得先として登録する
/plugin marketplace add anthropics/skills

# 登録した `anthropic-agent-skills` から、文書処理系の `document-skills` をインストールする
/plugin install document-skills@anthropic-agent-skills

# 同じ取得先から、サンプル兼実用スキル集の `example-skills` をインストールする
/plugin install example-skills@anthropic-agent-skills
```

#### 主なSkills例

`example-skills`、`document-skills`、`claude-api` を含めた、Anthropic公式skillsの代表例。

- [algorithmic-art](https://github.com/anthropics/skills/tree/main/skills/algorithmic-art)
  - アルゴリズムアート生成
- [brand-guidelines](https://github.com/anthropics/skills/tree/main/skills/brand-guidelines)
  - ブランドガイドライン適用
- [canvas-design](https://github.com/anthropics/skills/tree/main/skills/canvas-design)
  - Canvasデザイン作成
- [claude-api](https://github.com/anthropics/skills/tree/main/skills/claude-api)
  - Claude API / SDK実装
- [doc-coauthoring](https://github.com/anthropics/skills/tree/main/skills/doc-coauthoring)
  - 文書の共同執筆・編集
- [docx](https://github.com/anthropics/skills/tree/main/skills/docx)
  - Word文書作成・編集
- [frontend-design](https://github.com/anthropics/skills/tree/main/skills/frontend-design)
  - フロントエンドUI作成
- [internal-comms](https://github.com/anthropics/skills/tree/main/skills/internal-comms)
  - 社内コミュニケーション文書作成
- [mcp-builder](https://github.com/anthropics/skills/tree/main/skills/mcp-builder)
  - MCPサーバー作成
- [pdf](https://github.com/anthropics/skills/tree/main/skills/pdf)
  - PDF処理
- [pptx](https://github.com/anthropics/skills/tree/main/skills/pptx)
  - PowerPoint作成・編集
- [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator)
  - スキル作成支援
- [slack-gif-creator](https://github.com/anthropics/skills/tree/main/skills/slack-gif-creator)
  - Slack向けGIF作成
- [theme-factory](https://github.com/anthropics/skills/tree/main/skills/theme-factory)
  - テーマ・スタイル作成
- [web-artifacts-builder](https://github.com/anthropics/skills/tree/main/skills/web-artifacts-builder)
  - Web Artifact作成
- [webapp-testing](https://github.com/anthropics/skills/tree/main/skills/webapp-testing)
  - Webアプリのテスト
- [xlsx](https://github.com/anthropics/skills/tree/main/skills/xlsx)
  - Excel処理

### claude-plugins-official

インストール例:

```text
# Anthropic公式のClaude Codeプラグイン集から、コードレビュー用プラグインをインストールする
/plugin install code-review@claude-plugins-official

# 同じプラグイン集から、フロントエンドUI作成・改善向けプラグインをインストールする
/plugin install frontend-design@claude-plugins-official

# 同じプラグイン集から、TypeScriptのLSP連携プラグインをインストールする
/plugin install typescript-lsp@claude-plugins-official
```

#### 公式プラグイン

Claude Codeの開発ワークフローを補助する、Anthropic公式マーケットプレイス掲載のプラグイン。

主な対応プラグイン:

- [code-review](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-review)
  - コードレビュー
- [pr-review-toolkit](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/pr-review-toolkit)
  - PRレビュー支援
- [feature-dev](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/feature-dev)
  - 機能開発ワークフロー
- [code-modernization](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-modernization)
  - 古いコードの近代化
- [code-simplifier](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier)
  - コード単純化
- [security-guidance](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)
  - セキュリティ観点の確認
- [frontend-design](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/frontend-design)
  - フロントエンドUI作成
- [mcp-server-dev](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/mcp-server-dev)
  - MCPサーバー開発
- [commit-commands](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/commit-commands)
  - コミット操作支援
- [claude-md-management](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-md-management)
  - CLAUDE.md管理
- [session-report](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/session-report)
  - 作業セッションのまとめ

#### LSP系

LSPはLanguage Server Protocolのこと。型情報、定義ジャンプ、参照検索、診断などを使って、Claude Codeのコード理解を補助する。

対応プラグイン:

- [typescript-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/typescript-lsp)
- [pyright-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/pyright-lsp)
- [rust-analyzer-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/rust-analyzer-lsp)
- [gopls-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/gopls-lsp)
- [clangd-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/clangd-lsp)
- [csharp-lsp](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/csharp-lsp)

#### 外部サービス・ツール連携

Claude CodeからGitHub、ブラウザ操作、Firebase、Linear、Terraformなどの外部サービスや開発ツールを扱うためのプラグイン。

利用時は、認証情報・MCPサーバー・hooks・ローカル実行権限が関わる場合があるので、導入前にREADMEと設定ファイルを確認する。

対応プラグイン:

- [github](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/github)
  - GitHub連携
- [playwright](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/playwright)
  - ブラウザ操作・E2Eテスト
- [firebase](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/firebase)
  - Firebase連携
- [linear](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/linear)
  - Linear連携
- [terraform](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/terraform)
  - Terraform / IaC支援
- [context7](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/context7)
  - 最新ドキュメント参照
- [serena](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/serena)
  - セマンティックコード解析・編集支援

## OpenAI公式系

### Codex built-in / system skills

OpenAI公式ドキュメントによると、Codexにはsystem skillが同梱される。

- [imagegen](https://github.com/openai/skills/tree/main/skills/.system/imagegen)
  - 画像生成・画像編集
- [openai-docs](https://github.com/openai/skills/tree/main/skills/.system/openai-docs)
  - OpenAI API / Codex / Agents SDKなどの公式ドキュメント参照
- [plugin-creator](https://github.com/openai/skills/tree/main/skills/.system/plugin-creator)
  - Codex plugin作成
- [skill-creator](https://github.com/openai/skills/tree/main/skills/.system/skill-creator)
  - skill作成
- [skill-installer](https://github.com/openai/skills/tree/main/skills/.system/skill-installer)
  - curated skillのインストール

### OpenAI curated skills

インストール例:

```text
# GitHub PRコメント対応用のcurated skillをインストールする
$skill-installer gh-address-comments

# CI失敗の原因調査・修正用のcurated skillをインストールする
$skill-installer gh-fix-ci

# OpenAI公式ドキュメント参照用のcurated skillをインストールする
$skill-installer openai-docs
```

インストール後は、Codexを再起動すると追加したskillが読み込まれる。

Curated skills例:

- [gh-address-comments](https://github.com/openai/skills/tree/main/skills/.curated/gh-address-comments)
  - GitHub PRコメント対応
- [gh-fix-ci](https://github.com/openai/skills/tree/main/skills/.curated/gh-fix-ci)
  - CI失敗の修正
- [openai-docs](https://github.com/openai/skills/tree/main/skills/.curated/openai-docs)
  - OpenAI公式ドキュメント参照
- [playwright](https://github.com/openai/skills/tree/main/skills/.curated/playwright)
  - Playwright操作
- [playwright-interactive](https://github.com/openai/skills/tree/main/skills/.curated/playwright-interactive)
  - ブラウザ操作を含む確認
- [screenshot](https://github.com/openai/skills/tree/main/skills/.curated/screenshot)
  - スクリーンショット確認
- [security-best-practices](https://github.com/openai/skills/tree/main/skills/.curated/security-best-practices)
  - セキュリティのベストプラクティス
- [security-threat-model](https://github.com/openai/skills/tree/main/skills/.curated/security-threat-model)
  - 脅威モデリング
- [security-ownership-map](https://github.com/openai/skills/tree/main/skills/.curated/security-ownership-map)
  - セキュリティ責任範囲の整理
- [figma](https://github.com/openai/skills/tree/main/skills/.curated/figma)
  - Figma連携
- [figma-implement-design](https://github.com/openai/skills/tree/main/skills/.curated/figma-implement-design)
  - Figmaデザイン実装
- [figma-generate-design](https://github.com/openai/skills/tree/main/skills/.curated/figma-generate-design)
  - Figmaデザイン生成
- [jupyter-notebook](https://github.com/openai/skills/tree/main/skills/.curated/jupyter-notebook)
  - Notebook作業
- [cloudflare-deploy](https://github.com/openai/skills/tree/main/skills/.curated/cloudflare-deploy)
  - Cloudflareへデプロイ
- [vercel-deploy](https://github.com/openai/skills/tree/main/skills/.curated/vercel-deploy)
  - Vercelへデプロイ
- [netlify-deploy](https://github.com/openai/skills/tree/main/skills/.curated/netlify-deploy)
  - Netlifyへデプロイ
- [render-deploy](https://github.com/openai/skills/tree/main/skills/.curated/render-deploy)
  - Renderへデプロイ
- [linear](https://github.com/openai/skills/tree/main/skills/.curated/linear)
  - Linear連携
- [sentry](https://github.com/openai/skills/tree/main/skills/.curated/sentry)
  - Sentry連携
- [notion-spec-to-implementation](https://github.com/openai/skills/tree/main/skills/.curated/notion-spec-to-implementation)
  - Notion仕様から実装へ
- [notion-research-documentation](https://github.com/openai/skills/tree/main/skills/.curated/notion-research-documentation)
  - Notionで調査・ドキュメント化

## Gemini公式系

Gemini CLIでは「Skill単体」というより、拡張機能にskills、commands、MCP、hooks、sub-agentsなどをまとめる形。

インストール例:

```text
# Google Workspace連携用のGemini CLI拡張をインストールする
gemini extensions install https://github.com/gemini-cli-extensions/workspace

# 仕様化・計画・実装ワークフロー用のGemini CLI拡張をインストールする
gemini extensions install https://github.com/gemini-cli-extensions/conductor

# コードレビュー用のGemini CLI拡張をインストールする
gemini extensions install https://github.com/gemini-cli-extensions/code-review
```

公式拡張例:

- [conductor](https://github.com/gemini-cli-extensions/conductor)
  - 仕様化、計画、実装のワークフロー
- [code-review](https://github.com/gemini-cli-extensions/code-review)
  - コードレビュー
- [security](https://github.com/gemini-cli-extensions/security)
  - 脆弱性チェック、PR確認
- [workspace](https://github.com/gemini-cli-extensions/workspace)
  - Google Workspace連携
- [stitch](https://github.com/gemini-cli-extensions/stitch)
  - Stitch MCP server連携
- [sre](https://github.com/gemini-cli-extensions/sre)
  - SRE / Google Cloud調査
- [cicd](https://github.com/gemini-cli-extensions/cicd)
  - CI/CD関連
- [mcp-toolbox](https://github.com/gemini-cli-extensions/mcp-toolbox)
  - MCP toolbox
- [vertex](https://github.com/gemini-cli-extensions/vertex)
  - Vertex AIのprompt管理
- [looker](https://github.com/gemini-cli-extensions/looker)
  - Looker連携
- [oracledb](https://github.com/gemini-cli-extensions/oracledb)
  - Oracle DB用skills

---

## 用語集

| 用語 | 説明 |
|------|------|
| **Skill** | 作業手順・専門知識・ツール連携をひとまとめにしたもの。Anthropic / OpenAI 共通の概念 |
| **Plugin** | Claude Code などの環境に機能を追加する仕組み。MCP・hooks・commands などを含むことがある |
| **Extension** | CLI やエディタにコマンド・MCP・設定をまとめて追加する仕組み。Gemini CLI での呼び方 |
| **Agent Skills standard** | Anthropic / OpenAI / Gemini 系で共通参照される Skill の標準仕様（agentskills.io） |
| **MCP（Model Context Protocol）** | Coding Agent に外部ツールやデータソースを接続するための標準プロトコル |
| **MCP サーバー** | MCP に従ってエージェントにツールを提供する外部プロセス |
| **Hooks** | ツール実行前後などのイベントに合わせて任意のシェルコマンドを自動実行する仕組み |
| **Sub-agent** | 特定タスクに特化して呼び出せる、親エージェントから独立したエージェント |
| **Custom command** | スラッシュコマンドなど、エージェントに追加できる独自コマンド |
| **LSP（Language Server Protocol）** | 型情報・定義ジャンプ・診断などをエディタや AI に提供するプロトコル。Coding Agent の理解精度向上に使われる |
| **System skill** | Codex など一部のエージェントに最初から同梱されている組み込みスキル |
| **Curated skill** | 公式が精選して追加インストールできるようにしているスキル |
| **`$skill-installer`** | Codex の system skill。curated skill をコマンドでインストールできる |
| **`/plugin`** | Claude Code のプラグイン管理コマンド（`install` / `marketplace add` など） |
| **Marketplace** | スキル・プラグインの配布・検索が行える場所。プロバイダーごとに提供形態が異なる |
| **External plugin** | 外部サービス（GitHub / Firebase / Linear など）と連携するプラグイン。認証情報が必要なことが多い |

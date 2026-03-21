# プラグイン — 機能をパッケージ化して共有するアプストア

## プラグインとは？

これまで学んできたスキル、フック、MCPサーバー、エージェントはすべて `.claude/` ディレクトリ内の個別のファイルとして存在していました。うまく動作しますが、**他のプロジェクトやチームに共有するにはファイルを一つ一つコピー**しなければなりませんでした。

プラグインはこの問題を解決します。**複数の機能（スキル + フック + MCP + エージェント）を1つのフォルダにまとめてパッケージとして作り、Marketplaceを通じてインストール/配布**できるようにする仕組みです。

> まとめ: プラグイン = 機能まとめパッケージ + アプストア配布

## ノーコードツールとの比較

| ノーコードツール | プラグインの対応 |
|---|---|
| **ZapierのTemplate** | 他の人が作ったプラグインをインストール（`/plugin install`） |
| **NotionのTemplate Gallery** | Plugin Marketplace（公式/コミュニティ） |
| **Chromeウェブストアの拡張機能** | プラグインパッケージ（スキル + フック + MCP統合） |
| **WordPressのプラグイン** | Claude Codeプラグイン（インストール → 機能追加 → 有効化/無効化） |

主な違い: ノーコードツールは**アプリ内でマーケットプレイスをクリック**しますが、Claude Codeは**`/plugin` コマンドでインストールして管理**します。

## スタンドアロン vs プラグイン — いつどちらを使うか

| | **スタンドアロン**（`.claude/` に直接設定） | **プラグイン**（パッケージとしてまとめる） |
|---|---|---|
| **スキル名** | `/hello` | `/plugin-name:hello` |
| **適した状況** | 個人用、プロジェクト専用、クイック実験 | チーム共有、コミュニティ配布、複数プロジェクトで再利用 |
| **管理** | 手動コピー | バージョン管理 + 自動更新 |
| **例え** | 自分のPCに保存したマクロ | 会社全体が使う公式ツール |

> **ヒント**: `.claude/` で先に作ってテストし、共有が必要な場合にプラグインに移行するのが最も自然な流れです。

## プラグインの構造

プラグインは**1つのフォルダ**にすべてを収めます。

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          ← メタデータ（名前、説明、バージョン）
├── commands/                ← スラッシュコマンド（マークダウンファイル）
├── skills/                  ← エージェントスキル（SKILL.md）
├── agents/                  ← カスタムエージェント定義
├── hooks/
│   └── hooks.json           ← イベントフック設定
├── .mcp.json                ← MCPサーバー設定
├── .lsp.json                ← LSPサーバー設定（コードインテリジェンス）
└── settings.json            ← デフォルト設定値
```

### plugin.json — プラグインの身分証明書

```json
{
  "name": "my-plugin",
  "description": "コードレビュー自動化プラグイン",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

| フィールド | 役割 |
|---|---|
| `name` | プラグインの一意な識別子（= スキル名の前に付くネームスペース） |
| `description` | Marketplaceで表示される説明 |
| `version` | バージョン管理（更新の検出に使用） |

> **注意**: `commands/`、`skills/`、`agents/` などは `.claude-plugin/` ディレクトリの**外**（プラグインのルート）に置く必要があります。`.claude-plugin/` 内には `plugin.json` のみ配置します。

## プラグインの作り方 — 最小限の例

### 1. ディレクトリの作成

```bash
mkdir -p my-plugin/.claude-plugin
mkdir -p my-plugin/skills/review
```

### 2. plugin.json の作成

```json
// my-plugin/.claude-plugin/plugin.json
{
  "name": "review-plugin",
  "description": "コードレビュー自動化",
  "version": "1.0.0"
}
```

### 3. スキルの作成

```markdown
<!-- my-plugin/skills/review/SKILL.md -->
---
description: Review code for bugs, security, and performance
disable-model-invocation: true
---

Review the code for:
1. Potential bugs or edge cases
2. Security concerns
3. Performance issues
```

### 4. ローカルテスト

```bash
claude --plugin-dir ./my-plugin
```

Claude Code内で `/review-plugin:review` として実行できます。

## Marketplace — プラグインのアプストア

Marketplaceはプラグインのカタログです。**「ストアを追加 → 欲しいプラグインをインストール」**という2ステップの仕組みです。

### 公式 Marketplace

Claude Codeを起動すると**公式Anthropic Marketplace**が自動的に登録されています。`/plugin` を実行して **Discover** タブからすぐに探せます。

### 公式 Marketplace の主なプラグインカテゴリ

**1. Code Intelligence（コードインテリジェンス）**

言語サーバー（LSP）を接続して、Claudeが型エラー、定義ジャンプ、参照検索を即座に実行できるようにします。

| 言語 | プラグイン名 | 必要なバイナリ |
|---|---|---|
| TypeScript | `typescript-lsp` | `typescript-language-server` |
| Python | `pyright-lsp` | `pyright-langserver` |
| Go | `gopls-lsp` | `gopls` |
| Rust | `rust-analyzer-lsp` | `rust-analyzer` |

**2. External Integrations（外部サービス連携）**

MCPサーバーを事前に設定済みのプラグインです。手動設定なしにすぐ使えます。

- GitHub、GitLab（ソース管理）
- Slack（コミュニケーション）
- Notion、Linear、Jira（プロジェクト管理）
- Figma（デザイン）
- Vercel、Supabase（インフラ）

**3. Development Workflows（開発ワークフロー）**

- `commit-commands` — Gitコミット/プッシュ/PRワークフロー
- `pr-review-toolkit` — PRレビューエージェント
- `plugin-dev` — プラグイン開発ツール

## プラグインのインストール/管理フロー

```
/plugin                              ← プラグイン管理UIを開く
├── Discoverタブ: インストール可能なプラグインを探す
├── Installedタブ: インストール済みプラグインを管理
├── Marketplacesタブ: Marketplaceの追加/削除
└── Errorsタブ: エラーを確認
```

### 基本コマンド

```bash
# Marketplaceを追加（GitHubリポジトリ）
/plugin marketplace add owner/repo

# プラグインのインストール
/plugin install plugin-name@marketplace-name

# プラグインの無効化（削除ではなく無効化）
/plugin disable plugin-name@marketplace-name

# プラグインの再有効化
/plugin enable plugin-name@marketplace-name

# プラグインの完全削除
/plugin uninstall plugin-name@marketplace-name
```

### インストール範囲（Scope）

| Scope | 誰が使用？ | 保存場所 |
|---|---|---|
| **User** | 自分のみ、すべてのプロジェクトで | ユーザー設定 |
| **Project** | このリポジトリのすべてのコラボレーター | `.claude/settings.json` |
| **Local** | 自分のみ、このリポジトリでのみ | ローカル設定 |

## 既存の設定をプラグインに移行する

すでに `.claude/` 内にスキルやフックを作っている場合、簡単にプラグインに移行できます。

```bash
# 1. プラグインディレクトリを作成
mkdir -p my-plugin/.claude-plugin

# 2. plugin.jsonを作成
# { "name": "my-plugin", "version": "1.0.0", ... }

# 3. 既存ファイルをコピー
cp -r .claude/commands my-plugin/
cp -r .claude/skills my-plugin/

# 4. テスト
claude --plugin-dir ./my-plugin
```

| 移行前（スタンドアロン） | 移行後（プラグイン） |
|---|---|
| このプロジェクトでのみ使用 | Marketplaceで共有可能 |
| `.claude/commands/` にファイル | `plugin-name/commands/` にファイル |
| 手動コピーで共有 | `/plugin install` でインストール |

## チームのMarketplace運営

チーム全体が同じプラグインを使うようにするには、プロジェクトの `.claude/settings.json` にMarketplaceを登録します。

```json
{
  "extraKnownMarketplaces": {
    "team-tools": {
      "source": {
        "source": "github",
        "repo": "our-org/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "formatter@team-tools": true,
    "reviewer@team-tools": true
  }
}
```

チームメンバーがプロジェクトを開くと、Marketplaceのインストール案内が自動的に表示されます。

## 既存の学習内容との関係

```
プラグイン = パッケージング + 配布レイヤー
  ├── スキル（03）     ← プラグイン内の skills/ ディレクトリに含める
  ├── フック（08）     ← プラグイン内の hooks/hooks.json に含める
  ├── MCP（07）        ← プラグイン内の .mcp.json に含める
  ├── エージェント（04, 05）← プラグイン内の agents/ ディレクトリに含める
  └── LSP（新規）      ← プラグインだけの機能: コードインテリジェンスサーバー連携
```

以前学んだスキル、フック、MCPサーバーはすべてプラグインの**構成要素**です。プラグインはこれらを1つにまとめて**パッケージとして配布するための層**を追加したものです。

## まとめ

- プラグイン = 既存の機能（スキル + フック + MCP + エージェント）を**1つのフォルダにパッケージ化**したもの
- Marketplace = プラグインを**検索/インストール/更新**するアプストア
- `/plugin` コマンドでインストール、無効化、削除などを管理
- `.claude/` で先に実験 → 共有が必要な場合にプラグインに移行するのが自然な流れ
- チームのMarketplaceを設定すればチームメンバー全員に標準ツールを自動配布可能
- プラグインの必要バージョン: **Claude Code v1.0.33+**

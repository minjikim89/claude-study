# MCP（Model Context Protocol）— 外部世界とつなぐプラグ

## MCP とは？

Claude Code は基本的に**ローカルファイルシステムの中**で動作します。ファイルの読み書きやターミナルコマンドの実行はできますが、外部サービス（Slack、Google Drive、データベースなど）には直接アクセスできません。

MCP（Model Context Protocol）はこの限界を解決する**標準化された外部接続プロトコル**です。Claude Code に外部サービスへのアクセス能力をプラグ形式で追加する構造です。

## ノーコードツールでの対応概念

| ノーコードツール | MCP の対応 |
|---|---|
| **Zapier の App Connection** | MCP サーバー接続（Slack、GitHub など） |
| **Make のモジュール** | MCP が提供する個別ツール（tool） |
| **n8n の Credential ＋ Node** | MCP サーバー設定 ＋ ツール呼び出し |
| **Notion の Integration** | MCP サーバーで Notion API を接続 |

核心的な違い：ノーコードツールは **GUI でクリックして接続**しますが、MCP は**設定ファイルにサーバーアドレスを登録**する方式です。

## 仕組み

```
┌─ Claude Code ─────────────────────────────────┐
│                                                │
│  「Slack にメッセージを送って」                   │
│       │                                        │
│       ▼                                        │
│  MCP クライアント（内蔵）                        │
│       │                                        │
│       ▼                                        │
│  ┌─ MCP サーバー：Slack ─────┐                  │
│  │  ツール：send_message    │ ──▶ Slack API 呼び出し│
│  │  ツール：read_channel    │                  │
│  │  ツール：list_channels   │                  │
│  └───────────────────────────┘                  │
│                                                │
│  ┌─ MCP サーバー：GitHub ────┐                  │
│  │  ツール：create_issue    │ ──▶ GitHub API 呼び出し│
│  │  ツール：list_prs        │                  │
│  └───────────────────────────┘                  │
│                                                │
│  ┌─ MCP サーバー：PostgreSQL ┐                  │
│  │  ツール：query           │ ──▶ DB への直接アクセス│
│  │  ツール：list_tables     │                  │
│  └───────────────────────────┘                  │
└────────────────────────────────────────────────┘
```

**構造まとめ：**
- **MCP クライアント**：Claude Code に内蔵
- **MCP サーバー**：各外部サービスと通信する中間ブリッジ
- **ツール（Tool）**：MCP サーバーが提供する個別機能（メッセージ送信、イシュー作成など）

## MCP サーバーの設定方法

### プロジェクト単位の設定

`.claude/mcp.json` ファイルに登録：

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-..."
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    }
  }
}
```

### グローバル設定

`~/.claude/mcp.json` に登録するとすべてのプロジェクトで使用可能。

### 設定の構造

| フィールド | 説明 | 例 |
|---|---|---|
| `command` | MCP サーバーの実行コマンド | `npx`、`python`、`docker` |
| `args` | 実行引数 | パッケージ名、設定ファイルパスなど |
| `env` | 環境変数（API キーなど） | `SLACK_BOT_TOKEN`、`GITHUB_TOKEN` |

## 主な MCP サーバーの例

| サービス | MCP サーバー | 提供ツール | 活用シナリオ |
|---|---|---|---|
| **Slack** | @anthropic/mcp-server-slack | メッセージ送信、チャンネル読み取り | 日次報告の自動化 |
| **GitHub** | @anthropic/mcp-server-github | イシュー / PR 作成、コード検索 | PR レビューの自動化 |
| **PostgreSQL** | @anthropic/mcp-server-postgres | SQL クエリ、テーブル参照 | データ分析レポート |
| **Google Drive** | コミュニティ提供 | ドキュメントの読み書き | 文書の自動生成 |
| **Notion** | コミュニティ提供 | ページ / DB の操作 | プロジェクト管理の同期 |
| **Filesystem** | @anthropic/mcp-server-filesystem | ファイルの読み書き | 特定ディレクトリへのアクセス制限 |

> Anthropic 公式サーバー以外にも、コミュニティが作成した MCP サーバーが多数存在します。

## MCP と他の Claude Code 機能の関係

```
Claude Code のツール体系
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
内蔵ツール     Read、Write、Bash、Glob、Grep など
               → ローカルファイルシステムの操作

MCP ツール     send_message、query、create_issue など
               → 外部サービスの操作

スキル         MCP ツールを活用するレシピを定義可能
               → 「毎週月曜に Slack に週次報告を送信」スキル

Subagent       MCP ツールを使える Subagent を生成可能
               → DB 照会専用 Subagent（query ツールのみ許可）
```

## スキル ＋ MCP の組み合わせ例

MCP で外部サービスにアクセスできるようになると、スキルと組み合わせて強力な自動化を構築できます。

### 週次報告自動化スキル

```markdown
# /weekly-report スキル

## やること
1. GitHub MCP で今週マージされた PR リストを照会
2. 各 PR の変更内容をまとめる
3. Slack MCP で #team-updates チャンネルにフォーマットされた報告を送信

## 出力形式
**[週次開発報告]**
- 完了：N 件
- 進行中：N 件
- 主な変更：...
```

## セキュリティの考慮事項

| 項目 | 注意事項 |
|---|---|
| **API キーの管理** | `env` に直接書かず、環境変数の参照を推奨 |
| **権限の最小化** | MCP サーバーに必要最小限の権限だけ付与（read-only など） |
| **プロジェクト vs グローバル** | 機密性の高い接続はプロジェクト単位に制限 |
| **`.gitignore`** | API キーを含む設定は git にコミットしないよう注意 |

## まとめ

- MCP = Claude Code を**外部サービスと接続**する標準プロトコル
- ノーコードツールの「App Connection / Integration」に相当
- 設定ファイル（mcp.json）にサーバーを登録すると Claude が外部ツールを使用可能
- スキル ＋ MCP の組み合わせでノーコードツールレベルの自動化ワークフローを実現可能
- API キーの管理と権限の最小化に注意

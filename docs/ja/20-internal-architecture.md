# Claude Code 内部アーキテクチャ — ソースコードが語ること

## Claude Codeはどう作られているのか？

2026年3月31日、Claude Codeのソースコードが公開されました。**512,000行のTypeScript**、1,884ファイル。このコードを分析すると、私たちが毎日使っているツールが内部でどのように動作しているかが正確にわかります。

Claude Codeは3つの設計原則の上に構築されています：

| 原則 | 意味 | 実装例 |
|---|---|---|
| **Safety** | ユーザーのシステムを保護 | Permission Pipeline、ツール実行前の権限確認 |
| **Performance** | 高速な応答と効率的なリソース使用 | Lazy Import、Memoized Context、並列ツール実行 |
| **Extensibility** | 新機能の追加が容易 | MCPプロトコル、Hookシステム、Plugin構造 |

技術スタックは**TypeScript + React Ink**（ターミナルUIレンダリング）+ **Zustand**（状態管理）+ **Bun**（ランタイム）で構成されています。

## ノーコードツールでの対応概念

### 4フェーズ実行モデル = ZapierのTrigger-Filter-Action-Output

Claude Codeは内部的に4つのフェーズを経て動作します。これはZapierのワークフロー構造と正確に対応します：

```
Zapier:
  Trigger → Filter → Action → Output

Claude Code:
  Startup → Query Loop → Tool Execution → Display
```

| Zapier | Claude Code | 説明 |
|---|---|---|
| Trigger（イベント検知） | Startup（初期化） | 環境ロード、CLAUDE.mdパース、権限確認 |
| Filter（条件チェック） | Query Loop（クエリ分析） | ユーザー入力の解釈、使用ツールの決定 |
| Action（実行） | Tool Execution（ツール実行） | ファイル読み込み、コード作成、コマンド実行 |
| Output（結果） | Display（表示） | React InkでターミナルにUI結果をレンダリング |

### 7つの実行モード = Notion AIの多様なビュー

Notionで同じデータベースをテーブル、ボード、ギャラリーなど異なるビューで表示できるように、Claude Codeも同じコアエンジンを**7つのモード**で実行できます：

```
同じエンジン、異なるインターフェース:
  Notion: Table View / Board View / Gallery View / Calendar View
  Claude Code: Interactive / Non-interactive / Pipe / Print / API / MCP / Tool
```

### Context Injection = Cursorの.cursorrules読み込み

Cursorがプロジェクトを開くときに`.cursorrules`ファイルを自動的に読み込んでAIの動作をカスタマイズするように、Claude Codeも起動時に**CLAUDE.md、Memory、環境情報**を自動的にコンテキストに注入します。

```
Cursor起動:
  .cursorrules読み込み → AIにルールを適用

Claude Code起動:
  CLAUDE.mdロード → Memoryロード → 環境検出 → コンテキスト構成
```

## 実際の動作原理

### Startup：6ステップの初期化

Claude Codeを起動すると、プロンプトが表示される前に6つのステップが順番に実行されます：

```
1. 環境検出（OS、Shell、Git状態）
2. 設定ファイルロード（settings.json）
3. CLAUDE.mdパース（プロジェクト + グローバル）
4. Auto Memoryロード
5. MCPサーバー接続
6. Permission設定確認
```

### 7つの実行モード

| モード | 説明 | 使用例 |
|---|---|---|
| **Interactive** | ターミナルで対話的に使用 | `claude`を実行して直接対話 |
| **Non-interactive** | コマンド1つを実行して終了 | `claude -p "このファイルを分析して"` |
| **Pipe** | stdin/stdoutでデータ受け渡し | `cat file.ts \| claude -p "レビューして"` |
| **Print** | システムプロンプトのみ出力 | デバッグ/検証用 |
| **API** | 外部からHTTPで呼び出し | 他のプログラムからClaude Codeを使用 |
| **MCP** | MCPサーバーとして動作 | 他のAIエージェントがClaude Codeをツールとして使用 |
| **Tool** | SDKツールとして動作 | プログラム的な統合 |

### 4フェーズ実行モデルの詳細

```
Phase 1: Startup
  ┌─────────────────────────────────────┐
  │ 環境検出 → 設定ロード → CLAUDE.md   │
  │ → Memory → MCP → Permission        │
  └─────────────┬───────────────────────┘
                │
Phase 2: Query Loop
  ┌─────────────▼───────────────────────┐
  │ ユーザー入力を受信                    │
  │ → システムプロンプト + コンテキスト結合│
  │ → API呼び出し（SSEストリーミング）    │
  └─────────────┬───────────────────────┘
                │
Phase 3: Tool Execution
  ┌─────────────▼───────────────────────┐
  │ レスポンスにツール呼び出しがあるか？   │
  │ ├─ Yes → 権限確認 → ツール実行       │
  │ │        → 結果をコンテキストに追加   │
  │ │        → Phase 2に戻る            │
  │ └─ No  → Phase 4へ                 │
  └─────────────┬───────────────────────┘
                │
Phase 4: Display
  ┌─────────────▼───────────────────────┐
  │ React InkでターミナルUIをレンダリング │
  │ → Markdownフォーマット               │
  │ → 次の入力を待機                     │
  └─────────────────────────────────────┘
```

### Agent Loop：11ステップ

Phase 2-3が繰り返されるのが**Agent Loop**です。詳細には11ステップに分かれます：

| ステップ | 処理内容 |
|---|---|
| 1 | ユーザーメッセージの受信 |
| 2 | システムプロンプトの組み立て（CLAUDE.md + Memory + 環境） |
| 3 | APIリクエスト送信（SSEストリーミング） |
| 4 | レスポンストークンの受信開始 |
| 5 | ツール呼び出しの検出 |
| 6 | 権限確認（Permission Pipeline） |
| 7 | Hook実行（PreToolUse） |
| 8 | ツール実行 |
| 9 | Hook実行（PostToolUse） |
| 10 | 結果をコンテキストに追加 |
| 11 | 追加のツール呼び出しが必要か判断 → あればステップ3へ、なければ応答完了 |

### ソースディレクトリ構造

```
claude-code/src/
├── utils/          564ファイル — ユーティリティ関数（最大のディレクトリ）
├── components/     389ファイル — React Ink UIコンポーネント
├── tools/          287ファイル — ツール定義（Read、Write、Bashなど）
├── hooks/          156ファイル — Hookシステム
├── mcp/            134ファイル — MCPクライアント/サーバー
├── permissions/     98ファイル — 権限管理
├── memory/          87ファイル — Memoryシステム
└── agents/          69ファイル — サブエージェント、Agent Loop
```

`utils/`が564ファイルで最大のディレクトリである理由は、コンテキスト管理、トークン計算、ファイルシステム操作、キャッシュなどのコアインフラがここに集中しているためです。

## 実践シナリオ

### 「メッセージを入力すると内部で何が起こるのか」

`src/auth/ フォルダのログインフローを分析して`と入力したときの全プロセス：

```
1. [Query Loop] メッセージ受信
   → "src/auth/ フォルダのログインフローを分析して"

2. [Query Loop] システムプロンプト組み立て
   → CLAUDE.mdルール + Auto Memory + 現在のGit状態 + OS情報

3. [Query Loop] APIリクエスト（SSEストリーミング）
   → コンテキスト全体をClaudeに送信

4. [Tool Execution] Claudeがツール呼び出しを決定
   → "まずsrc/auth/ディレクトリを確認する必要がある" → Globツール呼び出し

5. [Permission] 権限確認
   → Globは読み取り専用 → 自動承認

6. [Hook] PreToolUseチェック
   → 登録されたHookなし → パス

7. [Tool Execution] Glob実行
   → src/auth/login.ts、src/auth/session.ts、src/auth/middleware.tsを発見

8. [Query Loop] 結果をコンテキストに追加 → 再度API呼び出し
   → "これらのファイルを読む必要がある" → Readツール3回呼び出し

9. [Tool Execution] Read 3回並列実行（安全なツールのため）
   → 3つのファイル内容を同時に読み込み

10. [Query Loop] ファイル内容をコンテキストに追加 → 再度API呼び出し
    → Claudeが分析結果を生成

11. [Display] React InkでMarkdownフォーマットしてターミナルにレンダリング
    → ユーザーに分析結果を表示
```

このプロセスでAgent Loopは**3回繰り返されました**（Glob 1回 + Read 3回並列 + 最終応答）。これがClaude Codeが単なるチャットボットではなく**エージェント**である理由です — 自ら判断し、ツールを選択し、結果に基づいて次のアクションを決定します。

## まとめ

- Claude Code = **512K行のTypeScript**、1,884ファイルで構成されたエージェントシステム
- 技術スタック：TypeScript + React Ink（TUI）+ Zustand（状態管理）+ Bun（ランタイム）
- **3つの設計原則**：Safety、Performance、Extensibility
- **4フェーズ実行モデル**：Startup → Query Loop → Tool Execution → Display
- **7つのモード**：Interactive、Non-interactive、Pipe、Print、API、MCP、Tool
- **11ステップのAgent Loop**がコア — ツール呼び出しが不要になるまで繰り返す
- ソース構造：utils（564）> components（389）> tools（287）の順に大きいディレクトリ

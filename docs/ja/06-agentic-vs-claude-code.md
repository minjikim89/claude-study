# Agentic Framework vs. Claude Code — 概念対応比較

## なぜ比較するのか？

AI エージェントのエコシステムには CrewAI、LangGraph、AutoGen といった「Agentic Framework」が存在します。これらはコードでエージェントを定義し、オーケストレーションする方式です。

Claude Code はこういったフレームワークとまったく異なるアプローチを取ります。**コードの代わりにマークダウン**、**ランタイムの代わりにコンテキスト注入**、**設定の代わりに自然言語**を使います。

しかし解決しようとしている問題は同じなので、概念同士を対応させることで Claude Code の構造を素早く理解できます。

## 根本的な思想の違い

| | **Agentic Framework** | **Claude Code** |
|---|---|---|
| **エージェント定義** | コード（Python など）でクラス / オブジェクトを生成 | マークダウンファイルで指示内容を記述 |
| **実行方式** | フレームワークのランタイムがエージェントコードを実行 | Claude がマークダウンを読んで指示に従って行動 |
| **ツール連携** | エージェントごとに tool 関数をバインド | Claude がデフォルトで持つツール（Bash、Read、Write など）を使用 |
| **入門の敷居** | Python コーディング ＋ フレームワーク API の習得が必要 | マークダウンを書ける能力があればよい |
| **例え** | 専門家を**雇って**ツールを持たせて働かせる | すでにいる秘書に**レシピカードを手渡す** |

## 概念別の詳細対応

### 1. システムプロンプト / グローバル設定

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
System Prompt / Global       →  CLAUDE.md
  Role                            「日本語で応答する」
  Backstory                       「AI Native Camp プロジェクト」
  Constraints                     「STOP PROTOCOL に従う」
```

Agentic フレームワークでエージェントの性格・役割・制約をコードで設定する部分が、Claude Code では **CLAUDE.md ファイル 1 つ**に置き換わります。3 段階のレベル（グローバル / プロジェクト / ローカル）で適用範囲を調整できます。

### 2. タスクごとのプロンプト

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Task-specific Prompt         →  スキル（SKILL.md）
  Task Instructions               レシピ本文
  Reference Data                  references/ フォルダ
  Scripts                         scripts/ フォルダ
```

フレームワークで各エージェントに割り当てる Task オブジェクトが、Claude Code では**スキル**になります。スキルはマークダウンファイルなので非エンジニアでも作成できます。

### 3. 記憶体系

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Long-term Memory             →  CLAUDE.md + Auto Memory
Working Memory               →  会話コンテキスト
Shared Memory                →  Agent Teams タスクリスト
```

フレームワークでベクター DB や外部ストレージで実装する長期記憶が、Claude Code では**ファイルベースの記憶体系**に置き換わります。別インフラなしでマークダウンファイルがそのままメモリになります。

### 4. ツール使用（Tool Calling）

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Custom Tools（関数定義）      →  内蔵ツール（Bash、Read、Write、Glob、Grep など）
External API 連携             →  MCP（Model Context Protocol）
```

フレームワークでは tool 関数を直接コーディングしてエージェントにバインドしますが、Claude Code は**ファイルシステム操作やターミナル実行などのツールをデフォルトで内蔵**しています。外部サービスの連携が必要な場合は **MCP** で拡張します。

### 5. 作業委任 / マルチエージェント

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Agent Delegation             →  Subagent（独立空間、結果のみ報告）
Multi-agent Orchestration    →  Agent Teams（独立セッション、直接通信）
Sequential Workflow          →  Subagent の順次呼び出し
Parallel Workflow            →  Agent Teams の並列実行
```

## 全体アーキテクチャの一覧

```
Claude Code アーキテクチャ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
永続記憶       →  CLAUDE.md + Auto Memory     （常にロード）
レシピ         →  スキル                       （呼び出し時にロード）
外部連携       →  MCP                          （外部サービスのプラグ）
作業委任       →  Subagent                     （独立空間、結果のみ報告）
チーム協働     →  Agent Teams                  （独立セッション、直接通信）
```

## ノーコードツールユーザー視点のマッピング

Zapier、Make、n8n といったノーコード自動化ツールを使ってきた方なら：

| ノーコードツールの概念 | Claude Code の対応 | 説明 |
|---|---|---|
| **ワークフローテンプレート** | スキル | 繰り返し作業をテンプレート化 |
| **グローバル設定 / Preferences** | CLAUDE.md | すべての作業に適用されるデフォルト値 |
| **変数 / データ保存** | Auto Memory | 学習した内容を自動保存 |
| **外部アプリ連携（Integration）** | MCP | 外部サービスへのアクセス |
| **並列実行（Parallel Path）** | Agent Teams | 複数の作業を同時実行 |
| **サブワークフロー** | Subagent | 作業の一部を委任 |

## まとめ

- Agentic Framework は**コード中心**、Claude Code は**テキスト（マークダウン）中心**
- 同じ問題を解決するが入門の敷居がまったく異なる
- Claude Code のすべての設定は結局**「コンテキストウィンドウにテキストを注入すること」**に帰結する
- フレームワークの複雑な設定が Claude Code ではマークダウンファイル数枚でシンプルに解決される

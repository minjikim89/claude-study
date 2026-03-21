# フック — イベントベースの自動実行

## フックとは？

スキルが**ユーザーが直接呼び出すレシピ**なら、フックは**特定のイベントが発生したときに自動的に実行されるスクリプト**です。

「ファイルを保存するたびにフォーマットを実行する」「コミット前にリントチェック」といった自動化がフックで実現できます。

## ノーコードツールでの対応概念

| ノーコードツール | フックの対応 |
|---|---|
| **Zapier のトリガー** | フックイベント（何が発生したとき） |
| **Make の Watch モジュール** | 特定のイベントを検知して自動実行 |
| **GitHub Actions の on: push** | イベントベースの自動ワークフロー |
| **Notion の自動化ルール** | 「ページ作成時 → テンプレート適用」のような自動ルール |

**核心的な違い**：ノーコードツールは GUI でトリガーを設定しますが、フックは**設定ファイル（settings.json）にシェルコマンドを登録**する方式です。

## 仕組み

```
Claude Code の作業フロー：

  ユーザーリクエスト → Claude ツール実行 → [フックチェック] → 結果を返す
                                               │
                                               ▼
                                      イベントが一致したら？
                                      ├─ Yes → シェルコマンド実行 → 結果を Claude に渡す
                                      └─ No  → パス
```

フックは Claude Code のツール実行の**前後**に介入できます：

| タイミング | 説明 | 用途 |
|---|---|---|
| **PreToolUse** | ツール実行の**前** | 実行のブロック、条件付き許可、警告 |
| **PostToolUse** | ツール実行の**後** | フォーマット、リント、ロギング、通知 |
| **Notification** | Claude が通知を送るとき | デスクトップ通知、Slack 通知の連携 |
| **Stop** | Claude が応答を終えるとき | 最終検証、自動後処理 |

## 設定方法

`~/.claude/settings.json` またはプロジェクトの `.claude/settings.json` に追加：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'File write detected: $CLAUDE_FILE_PATH'"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### 設定の構造

| フィールド | 説明 | 例 |
|---|---|---|
| `matcher` | どのツールに反応するか | `"Write"`、`"Bash"`、`"Read"` |
| `type` | フックの種類 | `"command"`（シェルコマンドを実行） |
| `command` | 実行するシェルコマンド | `"npx prettier --write ..."` |

### 使用可能な環境変数

フックコマンドの中で Claude Code が提供する環境変数を使用できます：

| 変数 | 説明 |
|---|---|
| `$CLAUDE_FILE_PATH` | 対象ファイルのパス |
| `$CLAUDE_TOOL_NAME` | 実行されたツール名 |
| `$CLAUDE_TOOL_INPUT` | ツールに渡された入力（JSON） |

## 実践シナリオ

### 1. ファイル保存時の自動フォーマット

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**効果**：Claude がファイルを作成するたびに Prettier が自動でコードフォーマットを整えます。ノーコードツールの例えでは「ドキュメント保存時に自動整列」と同じです。

### 2. 危険なコマンドのブロック

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -q 'rm -rf'; then echo 'BLOCKED: rm -rf is not allowed' >&2; exit 1; fi"
          }
        ]
      }
    ]
  }
}
```

**効果**：`rm -rf` のような危険なコマンドを Claude が実行しようとしたとき、自動的にブロックします。

### 3. 作業完了通知

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"$CLAUDE_NOTIFICATION_MESSAGE\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

**効果**：Claude が長い作業を終えると macOS の通知を表示します。別の作業をしていても完了を見逃しません。

### 4. ファイル変更時のリントチェック

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint \"$CLAUDE_FILE_PATH\" --fix 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

## スキル vs フック — 自動化方式の違い

| | **スキル** | **フック** |
|---|---|---|
| **トリガー** | ユーザーが `/コマンド` で呼び出す | イベント発生時に自動 |
| **定義方式** | マークダウン（SKILL.md） | JSON 設定 ＋ シェルコマンド |
| **実行主体** | Claude が指示を読んで実行 | シェルが直接コマンドを実行 |
| **用途** | 複雑なマルチステップ作業 | シンプルで繰り返す自動処理 |
| **例え** | 料理レシピ（自分で始める） | センサー ＋ 自動スプリンクラー（条件が満たされたら自動） |
| **使用例** | 「/weekly-report」 | 「ファイル保存 → 自動フォーマット」 |

## ノーコードツールユーザーのための整理

Zapier / Make を使ったことがある方はこう理解すればよいです：

```
ノーコードツールの構造：
  Trigger（イベント検知） → Action（実行） → Result（結果）

Claude Code フックの構造：
  イベント（ツール実行前後） → Shell Command（実行） → 結果を Claude に渡す
```

違いは**アクションがシェルコマンド**であること。GUI でクリックする代わりにターミナルコマンドを書きます。しかし原理は同じです：「X が起きたら自動的に Y をする。」

## まとめ

- フック = **イベントベースの自動実行**（ツール使用の前後にシェルコマンドを実行）
- PreToolUse（実行前）、PostToolUse（実行後）、Notification、Stop の 4 つのタイミング
- `settings.json` に JSON で設定
- スキル（手動呼び出し）＋ フック（自動実行）の組み合わせで完全な自動化を実現
- ノーコードツールの Trigger → Action パターンと同じ構造

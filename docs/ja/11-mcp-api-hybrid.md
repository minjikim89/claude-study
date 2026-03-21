# MCP + APIハイブリッドパターン — MCPでできないことはどうする？

## 問題の状況

MCPサーバーは外部サービスの**すべてのAPIをカバーしているわけではありません**。例えばNotion MCPはページの作成・更新・検索をサポートしていますが、**削除（アーカイブ）はサポートしていません**。公式のREST APIでは可能な機能なのに、MCPには含まれていないのです。

このギャップが生まれる理由:
- MCPサーバーの開発者がすべてのAPIエンドポイントを実装していない
- 危険な操作（削除、権限変更など）を意図的に除外している
- サービスAPIのアップデートがMCPサーバーに反映されるまでタイムラグがある

## ノーコードツールにおける対応の概念

| ノーコードツール | MCP + APIハイブリッドの対応 |
|---|---|
| **Zapier**: 基本アクション + Webhooks by Zapier | MCPツール + Bash/PythonでAPIを直接呼び出し |
| **Make**: 内蔵モジュール + HTTPモジュール | MCPツール + curl/requestsで補完 |
| **n8n**: 専用ノード + HTTP Requestノード | MCP + スキルでAPI呼び出しをラッピング |

核心は同じです: **基本提供の機能で80%を処理し、残りの20%は汎用のHTTP呼び出しで補完**する構造。

## 実務パターン3種類

### Pattern 1: MCP優先 + API補完（最も一般的）

```
日常作業（80%）              特殊作業（20%）
  ├─ ページ作成               ├─ ページ削除（アーカイブ）
  ├─ DBクエリ                 ├─ 権限変更
  ├─ ページ更新               ├─ バルク操作
  ├─ 検索                     └─ 高度なフィルタリング
  │
  └→ MCPツールで処理           └→ API直接呼び出しで処理
```

**メリット**: MCPの便利さ（Claudeが自然言語でツールを呼び出す）を最大限に活用しつつ、不足している部分のみを補完
**デメリット**: 2つの方法が混在するため、使い方が統一されない

### Pattern 2: すべてAPIで統一

MCPを使わず、すべての作業をAPI呼び出しスクリプトで処理します。

**選択するケース**:
- MCPがサポートする機能が非常に限られている場合
- 細かいエラーハンドリングが必要な場合
- 大規模なバッチ自動化パイプラインを構築する場合

**デメリット**: Claudeのツール統合（tool calling）のメリットを失います。「Notionで検索して」と言えば自動でMCPツールを選んでくれる便利さがなくなります。

### Pattern 3: カスタムMCPサーバーでギャップを埋める

不足しているAPIを自分でMCPサーバーとしてラッピングし、Claudeがすべての作業をツールとして呼び出せるようにする方法です。

```
Notion公式MCP              自作の補完MCP
  ├─ create-page               ├─ archive-page（削除）
  ├─ update-page               ├─ bulk-update
  ├─ search                    └─ set-permissions
  └─ query-database
```

**メリット**: 最もクリーンな使用体験（すべての機能がツールとして統合）
**デメリット**: MCPサーバーの開発が必要（Python/TypeScript）、メンテナンスの負担

### どのパターンを選ぶべきか？

```
MCPがサポートしていない機能がある
  │
  ├─ たまに必要（月1〜2回）
  │   └→ Pattern 1: そのときどきAPIを呼び出せばよい
  │
  ├─ 頻繁に必要（週に複数回）
  │   └→ Pattern 1 + スキル: API呼び出しをスキルでラッピングして再利用
  │
  └─ コアワークフローの一部
      └→ Pattern 3: カスタムMCPサーバーで統合
```

## 実装例: Notion MCP + APIハイブリッド

### Notion MCPがサポートすること / しないこと

| 作業 | MCPサポート | APIサポート | 備考 |
|------|:--------:|:--------:|------|
| ページ作成 | O | O | |
| ページ更新 | O | O | |
| ページ検索 | O | O | |
| DBクエリ | O | O | |
| コメント取得/作成 | O | O | |
| ページ削除（アーカイブ） | X | O | `archived: true` PATCH |
| ページの完全削除 | X | X | APIも未サポート |
| DBプロパティの削除 | X | O | propertyを `null` にPATCH |
| ブロック削除 | X | O | block DELETEエンドポイント |

### シナリオ: 「完了したタスクをアーカイブして」

このワークフローはMCPだけでは完成できません:
1. **DBから完了タスクを取得** → MCPで可能（query-database）
2. **各タスクをアーカイブ** → MCPは未サポート → API直接呼び出しが必要

### 実装方法 A: Claudeに直接API呼び出しを依頼

別途設定なしにClaude に自然言語で依頼:

```
NotionのTasks DBのDoneがtrueの項目を取得して、
各ページをNotion APIでarchive処理して。
APIキーは環境変数のNOTION_API_KEYを使って。
```

Claudeが実行するフロー:

```
Step 1: MCPツールでDBクエリ
  → notion.query-database (filter: Done = true)
  → 結果: page_idの一覧

Step 2: BashでAPIを直接呼び出し（MCPが未サポートのため）
  → curl -X PATCH https://api.notion.com/v1/pages/{page_id} \
      -H "Authorization: Bearer $NOTION_API_KEY" \
      -H "Notion-Version: 2022-06-28" \
      -d '{"archived": true}'
```

**メリット**: すぐに使える、設定不要
**デメリット**: 毎回APIキーの場所と呼び出し方法を説明する必要がある

### 実装方法 B: スキルでラッピングして再利用（推奨）

頻繁に使うハイブリッド作業をスキルとして作っておけば、毎回説明する必要なくスラッシュコマンドで呼び出せます。

```
.claude/skills/
  └─ notion-archive/
      └─ SKILL.md
```

**SKILL.mdの例:**

```markdown
---
name: notion-archive
description: Notionで完了したタスクをアーカイブ。"アーカイブ"、"完了の整理"という依頼に使用。
---

# Notion 完了タスクのアーカイブ

## 環境
- NOTION_API_KEY: 環境変数から読み込む
- Tasks DB ID: 6d1161325e944014b3c369983796ef15

## 実行手順

### ステップ1: 完了タスクの取得（MCPを使用）
Notion MCPのquery-databaseツールでTasks DBからDone = trueの項目を取得。

### ステップ2: アーカイブの確認
取得したタスクの一覧をユーザーに見せ、アーカイブを進めるか確認する。
アーカイブは元に戻せないため、必ず事前に確認すること。

### ステップ3: APIでアーカイブを実行
各ページに対してNotion REST APIを呼び出し、archived: trueに変更。

curl -X PATCH "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"archived": true}'

### ステップ4: 結果の報告
成功/失敗の件数をテーブルで出力。
```

**使い方:**

```
/notion-archive
または
"完了したタスクを整理して"
```

### 実装方法 C: Pythonスクリプトとして分離

繰り返し実行が多い場合やロジックが複雑な場合は、別スクリプトとして分離します:

```python
# scripts/notion_archive.py
import os
import requests

NOTION_API_KEY = os.environ["NOTION_API_KEY"]
TASKS_DB_ID = "6d1161325e944014b3c369983796ef15"
HEADERS = {
    "Authorization": f"Bearer {NOTION_API_KEY}",
    "Notion-Version": "2022-06-28",
    "Content-Type": "application/json",
}

def query_done_tasks():
    """Done = trueのタスクを取得"""
    url = f"https://api.notion.com/v1/databases/{TASKS_DB_ID}/query"
    payload = {
        "filter": {
            "property": "Done",
            "checkbox": {"equals": True}
        }
    }
    resp = requests.post(url, headers=HEADERS, json=payload)
    resp.raise_for_status()
    return resp.json()["results"]

def archive_page(page_id: str):
    """ページをアーカイブ処理"""
    url = f"https://api.notion.com/v1/pages/{page_id}"
    resp = requests.patch(url, headers=HEADERS, json={"archived": True})
    resp.raise_for_status()
    return resp.json()

def main():
    tasks = query_done_tasks()
    print(f"Found {len(tasks)} completed tasks")

    for task in tasks:
        title = task["properties"]["Name"]["title"][0]["plain_text"]
        result = archive_page(task["id"])
        status = "archived" if result.get("archived") else "failed"
        print(f"  {status}: {title}")

if __name__ == "__main__":
    main()
```

スキルからこのスクリプトを呼び出します:

```markdown
## 実行
python scripts/notion_archive.py を実行
```

## パターン別比較まとめ

| 基準 | A: 直接依頼 | B: スキルでラッピング | C: スクリプト分離 |
|------|:-----------:|:------------:|:---------------:|
| 初期設定 | なし | スキルファイル1つ | スキル + Pythonファイル |
| 再利用性 | 低い（毎回説明） | 高い（スラッシュコマンド） | 高い（スクリプト実行） |
| エラー処理 | Claudeが任意判断 | スキルに明示可能 | コードで精密に制御 |
| 推奨状況 | 1回限りの作業 | 定期的な繰り返し作業 | 複雑なロジック、バッチ処理 |

## まとめ

- MCPは外部サービスAPIの**一部分**のみをサポート — すべての機能をカバーしていない
- 不足している部分は**API直接呼び出し（curl、Python requests）**で補完する**ハイブリッドパターン**が最も一般的
- 頻繁に使うハイブリッド作業は**スキルでラッピング**すれば毎回説明する必要なく再利用可能
- 複雑なロジックやエラー処理が必要な場合は**別スクリプトとして分離**する
- MCPの限界 = **自分でスキルを作るべき理由**。汎用ツールのギャップを自分だけのワークフローで埋めることが真の自動化

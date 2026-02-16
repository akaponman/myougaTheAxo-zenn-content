---
title: "Hooks — ツール実行の自動制御"
---

## Hooksとは

Hooksは、Claude Codeのツール実行の前後に任意のコマンドを自動実行する仕組みです。コード品質の自動担保、セキュリティチェック、カスタムワークフローに使えます。

## 対応イベント一覧

| イベント | 発火タイミング | 主な用途 |
|---------|-------------|---------|
| `PreToolUse` | ツール実行**前** | バリデーション、ブロック |
| `PostToolUse` | ツール実行**後**（成功時） | フォーマッター、通知 |
| `PostToolUseFailure` | ツール実行**後**（失敗時） | エラーログ |
| `Notification` | 通知送信時 | カスタム通知 |
| `Stop` | エージェント応答完了時 | サマリー生成 |
| `SubagentStart` | サブエージェント起動時 | ログ |
| `SubagentStop` | サブエージェント完了時 | ログ |
| `SessionStart` | セッション開始時 | 初期化処理 |
| `SessionEnd` | セッション終了時 | クリーンアップ |

## 設定構造

`.claude/settings.json` に記述します：

```json
{
  "hooks": {
    "イベント名": [
      {
        "matcher": "ツール名パターン",
        "hooks": [
          {
            "type": "command",
            "command": "実行するコマンド",
            "timeout": 600
          }
        ]
      }
    ]
  }
}
```

**matcher** はツール名にマッチします：
- `"Bash"` — Bashツールのみ
- `"Write"` — Writeツールのみ
- `"Edit|Write"` — EditまたはWriteツール
- `"mcp__*"` — 全MCPツール
- `""` （空文字） — 全ツールにマッチ

## パターン1: ファイル保存時に自動フォーマット

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_PROJECT_DIR\"",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## パターン2: 危険なコマンドをブロック

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python scripts/validate_command.py"
          }
        ]
      }
    ]
  }
}
```

`scripts/validate_command.py`:
```python
import json, sys

# 標準入力からツール情報を受け取る
data = json.load(sys.stdin)
command = data.get("tool_input", {}).get("command", "")

BLOCKED = ["rm -rf /", "DROP TABLE", "git push --force"]
for b in BLOCKED:
    if b in command:
        print(f"BLOCKED: '{b}' は禁止されています", file=sys.stderr)
        sys.exit(2)  # exit 2 = ブロック
```

**終了コードの意味：**
- `0` — 成功（続行）
- `2` — エラー（ツール実行をブロック、stderrの内容がエラーとして表示）
- その他 — 警告（verbose時のみ表示、ツール実行は続行）

## パターン3: テスト自動実行（PostToolUse）

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "pytest tests/ -x -q --tb=short 2>/dev/null || true",
            "timeout": 60,
            "statusMessage": "テスト実行中..."
          }
        ]
      }
    ]
  }
}
```

## パターン4: AIにフィードバックさせる（prompt型）

hookの`type`には`command`以外に`prompt`も使えます：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "このコマンドが本番環境に影響する可能性がないか確認してください。問題があれば指摘してください。"
          }
        ]
      }
    ]
  }
}
```

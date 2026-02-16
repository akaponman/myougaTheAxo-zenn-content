---
title: "設定ファイルの階層と権限"
---

## 設定ファイルの場所と優先順位

Claude Codeの設定は複数レベルで管理されます。上位が優先：

| 優先度 | ファイル | スコープ |
|--------|---------|---------|
| 1（最高） | 管理者設定 | システム全体（IT管理者向け） |
| 2 | コマンドライン引数 | 実行時 |
| 3 | `.claude/settings.local.json` | プロジェクト個人（gitignore推奨） |
| 4 | `.claude/settings.json` | プロジェクト共有（コミット可） |
| 5（最低） | `~/.claude/settings.json` | ユーザー全体 |

## 管理者設定の場所

| OS | パス |
|----|------|
| macOS | `/Library/Application Support/ClaudeCode/settings.json` |
| Linux | `/etc/claude-code/settings.json` |
| Windows | `C:\ProgramData\ClaudeCode\settings.json` |

## 権限（Permissions）の設定

Claude Codeが実行できるツールを細かく制御できます：

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(pytest *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env*)",
      "Read(./secrets/**)",
      "Write(./.env)"
    ]
  }
}
```

**パターン解説：**
- `Bash(npm run *)` — `npm run` で始まるコマンドを許可
- `Read(./.env*)` — `.env`で始まるファイルの読み取りを拒否
- `Read(./secrets/**)` — secretsディレクトリ配下を再帰的に拒否

## 実践例: セキュリティ重視のチーム設定

`.claude/settings.json`（プロジェクト共有）:
```json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)",
      "Bash(npm run build)"
    ],
    "deny": [
      "Read(./.env*)",
      "Read(./credentials/**)",
      "Bash(rm -rf *)",
      "Bash(git push --force *)"
    ]
  }
}
```

## ファイル除外

特定のファイルをClaude Codeの視界から除外するには、`deny`ルールを使います：

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./config/secrets.json)",
      "Read(./node_modules/**)"
    ]
  }
}
```

## 環境変数の設定

設定ファイルから環境変数を注入できます：

```json
{
  "env": {
    "NODE_ENV": "development",
    "DATABASE_URL": "sqlite:///dev.db"
  }
}
```

---
title: "MCPサーバー — 外部ツール連携"
---

## MCPとは

Model Context Protocol (MCP) は、Claude Codeに外部ツールやデータソースを接続するプロトコルです。データベース、API、ファイルシステムなど、あらゆるものをClaudeの「手」にできます。

## 設定ファイルの場所

| ファイル | スコープ | 用途 |
|---------|---------|------|
| `.mcp.json`（プロジェクトルート） | プロジェクト共有 | チーム全員で使うMCPサーバー |
| `~/.claude.json` | ユーザー個人 | 個人のAPIキーを使うサーバー |

## 設定方法

### 方法1: CLI（推奨）

```bash
# SQLiteサーバーを追加
claude mcp add --transport stdio my-db -- npx @modelcontextprotocol/server-sqlite ./data/app.sqlite3

# HTTPサーバーを追加
claude mcp add --transport http api-server https://api.example.com/mcp

# 環境変数付きで追加
claude mcp add --transport stdio --env GITHUB_TOKEN=ghp_xxx github -- npx @modelcontextprotocol/server-github

# 一覧表示
claude mcp list

# 削除
claude mcp remove my-db
```

### 方法2: 設定ファイル直接編集

`.mcp.json`:
```json
{
  "mcpServers": {
    "project-db": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "./data/app.sqlite3"]
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

## 環境変数の展開

`.mcp.json`内で環境変数を使えます：

```json
{
  "mcpServers": {
    "api": {
      "type": "http",
      "url": "${API_BASE_URL:-https://localhost:3000}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  }
}
```

`${VAR:-default}` 構文でデフォルト値も指定可能。

## 主要なMCPサーバー一覧

| サーバー | パッケージ | 用途 |
|---------|-----------|------|
| SQLite | `@modelcontextprotocol/server-sqlite` | DBの読み書き |
| GitHub | `@modelcontextprotocol/server-github` | Issue/PR操作 |
| Filesystem | `@modelcontextprotocol/server-filesystem` | ファイル操作 |
| Postgres | `@modelcontextprotocol/server-postgres` | PostgreSQL操作 |
| Memory | `@modelcontextprotocol/server-memory` | ナレッジグラフ |
| Fetch | `@modelcontextprotocol/server-fetch` | Web取得 |
| Brave Search | `@modelcontextprotocol/server-brave-search` | Web検索 |

## 実践例: プロジェクトDB + GitHub連携

```json
{
  "mcpServers": {
    "app-db": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "./data/production.sqlite3"]
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

これで「データベースの中身を見て」「このIssueをクローズして」が自然言語で可能に。

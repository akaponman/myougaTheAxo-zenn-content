---
title: "CLAUDE.md — AIへの「プロジェクト仕様書」"
---

## CLAUDE.mdとは

CLAUDE.mdは、Claude Codeがプロジェクトのコンテキストを理解するための設定ファイルです。プロジェクトルートに置くと、毎回の会話で自動的に読み込まれます。

**これがないとどうなるか：**
- Claude Codeは毎回ゼロからプロジェクトを理解しようとする
- 不要なファイル変更やスタイル違反が発生する
- 同じ説明を何度もする羽目になる

## CLAUDE.mdの配置場所と優先順位

CLAUDE.mdは複数の場所に置けます。上位のものが優先されます：

| 場所 | スコープ | 用途 |
|------|---------|------|
| `~/.claude/CLAUDE.md` | ユーザー全体 | 個人の好み（エディタ設定、コードスタイル等） |
| `CLAUDE.md`（プロジェクトルート） | プロジェクト共有 | チーム共通の規約・アーキテクチャ |
| 親ディレクトリの`CLAUDE.md` | ワークスペース | モノレポや複数PJの共通ルール |

**ポイント:** gitリポジトリのルートまで遡って探索されます。ワークスペース全体に適用したいルールは、リポジトリの親ディレクトリに置きましょう。

## パターン1: ミニマル構成（個人プロジェクト向け）

```markdown
# プロジェクト名

## 概要
Pythonで書かれた家計簿CLIツール。SQLiteでデータ保存。

## 技術スタック
- Python 3.13
- SQLite3
- Click (CLI フレームワーク)

## 開発規約
- テストは pytest で実行: `pytest tests/`
- フォーマットは ruff: `ruff format .`
- 型アノテーション必須
```

**ポイント:** 最小限でも「技術スタック」「テスト実行方法」「フォーマッター」の3つがあれば、Claude Codeの出力品質が格段に上がります。

## パターン2: モノレポ構成（複数パッケージ）

```markdown
# MyProject Monorepo

## 構成
| パッケージ | パス | 技術 | 概要 |
|-----------|------|------|------|
| API | packages/api/ | FastAPI, SQLAlchemy | REST API |
| Frontend | packages/web/ | Next.js, TypeScript | Webフロント |
| Shared | packages/shared/ | TypeScript | 共通型定義 |

## ビルド・テスト
- API: `cd packages/api && pytest`
- Frontend: `cd packages/web && pnpm test`
- 全体: `pnpm -r test`

## 規約
- APIの変更は必ずOpenAPI specを更新
- sharedの型変更はAPI・Frontend両方のテストを確認
- PRは1パッケージに閉じること（cross-package変更は要相談）
```

## パターン3: チーム開発構成（組織向け）

```markdown
# TeamProject

## アーキテクチャ
[ここにシステム構成図をテキストで書く]

## コーディング規約
- 最小限の変更: 要件に明示されたものだけ変更
- 過剰設計禁止: 1回限りの処理に抽象化を入れない
- 未使用コードは完全削除（互換性ハック不要）
- コメントは「なぜ」だけ書く（「何を」は書かない）

## セキュリティ
- SQLはパラメータバインディング必須
- ユーザー入力は全てバリデーション
- .envファイルはコミット禁止

## Git運用
- ブランチ: feature/xxx, fix/xxx, docs/xxx
- コミットメッセージ: Conventional Commits
- main直push禁止
```

## パターン4: AI行動制御（やってほしくないことリスト）

```markdown
## 禁止事項
- README.mdやドキュメントを勝手に作成しない
- 既存コードにdocstring/型アノテーションを追加しない（変更したコードのみ）
- git push は明示的に指示された場合のみ
- .envファイルの中身をコミットに含めない

## 優先事項
- 既存ファイルの編集 > 新規ファイル作成
- シンプルな実装 > 抽象化された実装
- 動くコード > 完璧なコード
```

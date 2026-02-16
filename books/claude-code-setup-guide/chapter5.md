---
title: "カスタムスキル（スラッシュコマンド）"
---

## スキルとは

スキルは再利用可能なプロンプトテンプレートです。`/deploy`や`/review`のようなスラッシュコマンドとして呼び出せます。

## ディレクトリ構成

```
.claude/skills/skill-name/SKILL.md    # プロジェクトスキル（コミット可）
~/.claude/skills/skill-name/SKILL.md  # 個人スキル（全PJで使える）
```

ファイル名は必ず `SKILL.md`（大文字）です。ディレクトリ名がスキル名になります。

## 基本的なスキルの作成

`.claude/skills/deploy/SKILL.md`:

```markdown
---
name: deploy
description: アプリケーションをデプロイする
allowed-tools: Bash, Read
---

以下の手順でデプロイを実行してください：

1. `npm run build` でビルド
2. `npm run test` でテスト
3. テストが全て通ったら `./deploy.sh` を実行
4. デプロイ結果を報告
```

使い方: `/deploy`

## 引数付きスキル

`.claude/skills/fix-issue/SKILL.md`:

```markdown
---
name: fix-issue
description: GitHubのIssueを修正する
argument-hint: "[issue番号]"
allowed-tools: Bash, Read, Edit, Write, Grep, Glob
---

GitHub Issue #$ARGUMENTS を修正してください。

1. `gh issue view $ARGUMENTS` でIssue内容を確認
2. 関連コードを調査
3. 修正を実装
4. テストを実行して確認
5. コミットメッセージにIssue番号を含める
```

使い方: `/fix-issue 42`

## 動的コンテキスト（プリプロセッシング）

`` !`command` `` 構文でコマンド出力をプロンプトに埋め込めます：

`.claude/skills/review-pr/SKILL.md`:

```markdown
---
name: review-pr
description: 現在のPRをレビューする
allowed-tools: Read, Grep, Glob
---

以下のPR差分をレビューしてください：

!`gh pr diff`

PRの説明：
!`gh pr view`

レビュー観点：
- セキュリティ上の問題はないか
- テストは十分か
- コーディング規約に準拠しているか
```

## レガシー形式（commands/）

単純なコマンドは旧形式でも作れます：

```
.claude/commands/test.md        →  /test
~/.claude/commands/my-util.md   →  /my-util
```

ファイルの中身がそのままプロンプトになります。`$ARGUMENTS`で引数を受け取れます。

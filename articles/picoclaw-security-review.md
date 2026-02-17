---
title: "PicoClawは安全？中華製AIエージェントのセキュリティを徹底検証した"
emoji: "🔍"
type: "tech"
topics: ["PicoClaw", "セキュリティ", "AI", "OpenClaw"]
published: true
---

## はじめに

2026年2月にリリースされ、わずか1週間でGitHub Stars 12,000超を達成した超軽量AIエージェント「PicoClaw」。

わずか10MB RAMで動作し、$10のRISC-Vボードでも動くという話題性から急速に広まっているけど…

**「これ、本当に安全なの？」**

開発元は**中国・深セン（Shenzhen）のSipeed社**。RISC-V開発ボードで知られるハードウェアメーカーだ。

この記事では、PicoClawのセキュリティを検証した結果をまとめる。

## PicoClawの概要

| 項目 | 内容 |
|------|------|
| 開発元 | Sipeed（中国・深セン） |
| ライセンス | MIT |
| 言語 | Go |
| RAM | 10MB未満 |
| 起動時間 | 1秒以下 |
| リリース | 2026年2月9日 |

OpenClawの超軽量版として、ターミナルベースのAIエージェント。ファイル操作、コマンド実行、Web検索、メッセージング連携（Discord/Telegram/Slack/QQ/LINE）が可能。

## 発見されたセキュリティ問題

### 1. 設定ファイルのパーミッションが甘い（深刻度: 高）

PicoClawの設定ファイルが**0644パーミッション**で作成される。APIキーやボットトークンを含むファイルが、同じマシン上の全ユーザーから読み取り可能。

正しくは`0600`（オーナーのみ）であるべき。これはセキュリティの基本中の基本。

**auth_store、cron_storeも同様の問題あり。**

### 2. Slackアクセス制御のバイパス（深刻度: 高）

GitHub Issue #179で報告。`handleSlashCommand`と`handleAppMention`で`IsAllowed()`チェックが欠落。

`allowFrom`ホワイトリストを設定しても、Slackワークスペース内の**誰でも**Botを操作できてしまう。

### 3. ワークスペースパス制限の突破（深刻度: 中）

2月15日のPRで修正されたが、それ以前のバージョンでは`workspace.restricted: true`に設定しても、意図したディレクトリ外のファイルにアクセス可能だった。

### 4. OpenClawエコシステムの問題

PicoClawの親プロジェクトOpenClawは:
- 30,000以上のインスタンスがインターネットに露出
- トークン窃取のRCEチェーンを含む40以上の脆弱性が2026年2月12日に修正
- Aikido社のレポートで「セキュアにしようとすること自体が滑稽」と評される状態

## データ送信先の分析

### PicoClaw自体
- テレメトリ通信: 確認されていない
- OSSでコード監査可能（MIT License）

### 外部サービスへの送信
1. **LLM API**: OpenRouter, Anthropic, OpenAI, DeepSeek, Groq, Zhipu AI等にプロンプトを送信
2. **メッセージング**: 各プラットフォーム（Discord, Telegram, QQ等）にメッセージ送信
3. **Web検索**: 直接外部サイトにリクエスト

**注意**: DeepSeekやZhipu AI（智谱AI）など中国のLLMプロバイダを使う場合、データは中国の法規制下で処理される。

## 安全に使うための対策

### 必須対策

```bash
# 設定ファイルのパーミッション修正
chmod 600 ~/.picoclaw/config.yaml
chmod 600 ~/.picoclaw/auth_store.json
chmod 600 ~/.picoclaw/cron_store.json
```

### 推奨対策

1. **隔離環境で運用**: メインPCではなくサブPC or VMに隔離
2. **ネットワーク制限**: ファイアウォールで通信先をホワイトリスト管理
3. **LLMプロバイダ**: Anthropic/OpenAIなど非中国系プロバイダのみ使用
4. **ホワイトリスト設定**: `allowFrom`を必ず設定（ただしSlackは現状バイパス可能）
5. **通信監視**: Wireshark等で定期的にパケットキャプチャ

### 隔離ネットワーク構成例

```
[メインPC] ←→ [ルーター] ←→ [インターネット]
                  ↓
            [検証用PC（PicoClaw）]
              - 全アウトバウンド通信をデフォルトブロック
              - メインPC経由のプロキシのみ許可
              - 通信ログを完全記録
```

## 結論

| 判定 | 状況 |
|------|------|
| ✅ 使っていい | 隔離環境 + ネットワーク制限 + 通信監視あり |
| ❌ 使ってはダメ | メインPC直接 / APIキー共存 / デフォルト設定のまま |

PicoClaw自体にバックドアがあるかはOSSなので検証可能。しかし**「設定ファイルのパーミッションが0644」**という時点で、セキュリティに対する意識の低さが見える。

便利なツールだが、**隔離なしでの使用は推奨しない**。

検証環境の構築手順は[noteの有料記事](https://note.com/myougaTheAxo)で詳しく解説しているので、実際に検証したい方はそちらを参照してほしい。

## 参考

- [PicoClaw GitHub](https://github.com/sipeed/picoclaw)
- [Issue #179 - Slack allow list bypass](https://github.com/sipeed/picoclaw/issues/179)
- [Aikido - Why Trying to Secure OpenClaw is Ridiculous](https://www.aikido.dev/blog/why-trying-to-secure-openclaw-is-ridiculous)
- [SOPHOS - OpenClaw experiment warning](https://www.sophos.com/en-us/blog/the-openclaw-experiment-is-a-warning-shot-for-enterprise-ai-security)

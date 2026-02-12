# Claude Code チャット統合プロジェクト

## 概要

SlackやDiscordからClaude Codeの機能を利用できるようにするシステムの構築。

## 目的

ターミナルを開かずに、普段使っているチャットツール（Slack/Discord）から以下の操作を行えるようにする：

- コードの編集
- Gitコミットの作成
- プルリクエスト（PR）の作成
- その他のClaude Code機能

## 期待する使い方（例）

```
ユーザー: @claude-bot src/app.tsのログイン関数にバリデーションを追加して

Bot: 了解しました。src/app.tsを確認して、バリデーションを追加します...
     [編集完了] 以下の変更を行いました：
     - メールアドレスの形式チェックを追加
     - パスワードの長さチェックを追加

ユーザー: @claude-bot それをコミットしてPRを作成して

Bot: コミットとPRを作成しました。
     PR: https://github.com/user/repo/pull/123
```

## システム構成

```
┌─────────────────┐
│  Slack/Discord  │
│    ユーザー      │
└────────┬────────┘
         │ メッセージ
         ▼
┌─────────────────┐
│   ボットサーバー  │
│  (Node.js等)    │
└────────┬────────┘
         │ コマンド実行
         ▼
┌─────────────────┐
│   Claude Code   │
│   (CLI)         │
└────────┬────────┘
         │ ファイル操作・Git操作
         ▼
┌─────────────────┐
│  リポジトリ      │
│  (ローカル)      │
└─────────────────┘
```

## 必要なコンポーネント

### 1. ボットサーバー
- **役割**: Slack/Discordからのメッセージを受信し、Claude Codeを実行
- **技術選択肢**: Node.js, Python, Go など
- **必要な機能**:
  - Slack/Discord APIとの連携
  - Claude Code CLIの呼び出し
  - 結果の整形と返信

### 2. Claude Code CLI
- **インストール**: サーバーにClaude Codeをインストール
- **実行モード**: ヘッドレスモード（`-p` オプション）を使用
- **認証**: Anthropic APIキーの設定

### 3. リポジトリ環境
- **Git設定**: ユーザー名、メールアドレス
- **GitHub認証**: SSH鍵またはPersonal Access Token
- **対象リポジトリ**: クローン済みのリポジトリ

### 4. Slack/Discordアプリ
- **Slack**: Slack Appを作成、Bot Token取得
- **Discord**: Discord Botを作成、Bot Token取得

## セキュリティ考慮事項

### 認証・認可
- [ ] 許可されたユーザーのみがコマンドを実行できるようにする
- [ ] 特定のチャンネルのみで動作するように制限
- [ ] 危険なコマンドの制限（`rm -rf` など）

### シークレット管理
- [ ] APIキーを環境変数で管理
- [ ] `.env`ファイルをGitにコミットしない
- [ ] 本番環境ではシークレットマネージャーを使用

### 監査ログ
- [ ] 誰が何を実行したかを記録
- [ ] エラー発生時の通知

## 実装ステップ

### Phase 1: 基本構築
1. ボットサーバーのセットアップ
2. Slack/Discordアプリの作成
3. Claude Code CLIのインストールと認証設定
4. 簡単なコマンド実行テスト

### Phase 2: 機能実装
1. メッセージ受信→Claude Code実行のフロー構築
2. 結果の整形と返信機能
3. エラーハンドリング

### Phase 3: セキュリティ強化
1. ユーザー認証の実装
2. コマンド制限の実装
3. 監査ログの実装

### Phase 4: 運用準備
1. ドキュメント整備
2. テスト
3. 本番デプロイ

## 技術的な詳細

### Claude Code ヘッドレスモード

```bash
# 基本的な使い方
claude -p "やりたいこと" --allowedTools "Bash,Edit,Write,Read"

# 出力形式を指定
claude -p "やりたいこと" --output-format json

# 作業ディレクトリを指定
cd /path/to/repo && claude -p "PRを作成して"
```

### Slack Bot 実装例（Node.js）

```javascript
const { App } = require('@slack/bolt');
const { exec } = require('child_process');

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET,
});

app.message(async ({ message, say }) => {
  const prompt = message.text;

  exec(`claude -p "${prompt}" --output-format json`, (error, stdout) => {
    if (error) {
      say(`エラーが発生しました: ${error.message}`);
      return;
    }
    say(stdout);
  });
});
```

## 代替案・参考情報

### MCP (Model Context Protocol)
- AnthropicのMCPを使った統合も検討可能
- https://modelcontextprotocol.io/

### 既存ツール
- GitHub Copilot in CLI
- その他のAIコーディングアシスタント

## 未決定事項

- [ ] Slack と Discord どちらを優先するか
- [ ] 対象リポジトリの管理方法（複数リポジトリ対応？）
- [ ] サーバーのホスティング先（AWS, GCP, 自宅サーバー等）
- [ ] コスト試算

## 次のアクション

1. プラットフォーム（Slack/Discord）の選定
2. サーバー環境の準備
3. プロトタイプの作成

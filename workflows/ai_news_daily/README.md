# Daily AI News to Slack Workflow

## 概要
AI関連の最新ニュースを自動で収集し、OpenAIで日本語要約した上で、毎朝Slackチャンネルに自動投稿するn8n用ワークフローです。

## 主な機能
- NewsAPIから「artificial intelligence」関連の最新ニュース（英語）を取得（最大5件）
- 各記事のタイトル・URL・説明を抽出
- OpenAI（gpt-3.5-turbo）で日本語200文字以内に要約（Markdown形式、箇条書き）
- 指定Slackチャンネル（例: #news-ai）へ自動投稿
- **記事が0件の場合は「本日のAIニュースはありません」とSlackに通知**
- **Slack投稿失敗時は管理者用チャンネル（#news-admin）にエラー通知**

## 構成イメージ
```
[定時トリガー] → [NewsAPI取得] → [記事分割] → [記事0件判定]
   ├─→ [0件時: Slack通知]
   └─→ [要約プロンプト生成] → [OpenAI要約] → [Slack投稿] → [投稿失敗時: 管理者通知]
```

## 必要なもの
- n8n（v1.94.0以降推奨）
- NewsAPIのAPIキー
- OpenAI APIキー
- Slack Bot Token（投稿先チャンネルへの権限付与）

## セットアップ手順
1. NewsAPI, OpenAI, Slackの認証情報をn8nに登録
2. 本ワークフロー（ai_news_daily_slack.workflow.json）をn8nにインポート
3. 各ノードの認証情報・パラメータを設定
4. 必要に応じて投稿チャンネルや取得件数、スケジュールを調整
5. 管理者通知用チャンネル（#news-admin）を用意し、Botを招待

## 注意事項
- APIキーや認証情報は絶対に公開しないでください
- NewsAPIの無料枠には制限があります
- OpenAIのAPI利用料が発生します
- 本ワークフローはサンプルです。実運用前に十分なテストを行ってください
- エラー通知先チャンネルや通常投稿チャンネルは運用に合わせて変更してください 
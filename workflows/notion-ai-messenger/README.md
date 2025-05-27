# Notion AI Messenger Workflow

## 概要
Notionに保存したデータをAI（OpenAI等）で分析・要約し、その結果をSlackやLINEに自動送信するn8nクラウド用ワークフローです。

## 主な機能
- Notionデータベース/ページからのデータ取得
- OpenAI等によるAI分析・要約・分類
- Slackチャンネル/LINEグループへの自動通知
- エラー時の通知・ログ

## 構成イメージ
```
[Notion DB] → [n8n: Notionノード] → [n8n: OpenAIノード] → [n8n: Functionノード(整形)]
   ├─→ [n8n: Slackノード]
   └─→ [n8n: Functionノード(JSON変換)] → [n8n: HTTPノード(LINE送信)]
```

## 必要なもの
- n8nクラウド（v1.94.0以降）
- Notion API連携設定
- OpenAI APIキー
- Slack Bot Token
- LINEチャネルアクセストークン

## セットアップ手順
1. `.env.example`を参考にAPIキー等を取得・設定
2. n8nクラウドで本ワークフロー（notion-ai-messenger.json）をインポート
3. 各ノードの認証情報・パラメータを設定
4. 必要に応じてスケジュールやWebhookを調整

## 参考リポジトリ
- https://github.com/hollaugo/notion-ai-knowledge
- https://github.com/takusandayooo/slack-ai-qa-bot
- https://github.com/YiChih1979/markdown2json
- https://github.com/enescingoz/awesome-n8n-templates

## 注意事項
- APIキーや認証情報は絶対に公開しないでください
- 本ワークフローはサンプルです。実運用時は十分なテストを行ってください 
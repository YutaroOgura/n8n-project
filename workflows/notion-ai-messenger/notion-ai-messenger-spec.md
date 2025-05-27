# Notion AI Messenger ワークフロー仕様書

## 1. 概要
Notionに保存されたデータをAI（OpenAI等）で分析し、SlackやLINEに自動送信するn8nクラウド用ワークフローの設計仕様です。

## 2. ワークフロー全体像
- 定期実行またはWebhookでトリガー
- Notion APIでデータ取得
- OpenAI等で要約・分析
- Slack/LINE用にメッセージ整形
- Slackノードで送信
- LINEはFunctionノードでJSON変換→HTTPノードで送信
- エラー時は通知・ログ

## 3. ノード構成例
1. **[Cron/Webhook]**: 定期実行または外部トリガー
2. **[Notion]**: データベースやページからデータ取得
3. **[OpenAI]**: 取得データを要約・分析
4. **[Function]**: メッセージ整形（Slack/LINE用）
5. **[Slack]**: チャンネルやDMに送信
6. **[Function]**: LINE用JSON変換
7. **[HTTP]**: LINE Messaging APIにPOST
8. **[Error/Catch]**: 失敗時の通知・ログ

## 4. 各ノードの役割・設定例
- **Notionノード**: Database IDやクエリ条件を指定
- **OpenAIノード**: モデル・プロンプトを柔軟に設定
- **Functionノード**: SlackはBlock Kit形式、LINEはmarkdown2json変換
- **Slackノード**: Bot Token/Channel ID指定
- **HTTPノード**: LINEアクセストークン/ユーザーID指定

## 5. エラー処理
- Catchノードで失敗時に管理者へ通知
- ログ記録やリトライ設計も推奨

## 6. 拡張案
- 分析結果をNotionへ逆書き込み
- ユーザーからの質問受付→AIでNotion検索→Slack/LINEで回答
- 他サービス（メール等）への多重通知

## 7. 参考
- https://github.com/hollaugo/notion-ai-knowledge
- https://github.com/takusandayooo/slack-ai-qa-bot
- https://github.com/YiChih1979/markdown2json 
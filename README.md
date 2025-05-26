# N8n Slack ChatGPT Workflow

## 概要
このプロジェクトは、Slackメッセージを受信し、OpenAI GPTを使用してキーワード抽出とインターネット検索を行い、検索結果を基に回答を生成してSlackに返信する自動応答システムです。

## 機能
- Slackメッセージの受信と処理
- OpenAI GPTによるキーワード抽出
- Tavily APIを使用したWeb検索
- ChatGPTによる回答生成
- Slackへの自動返信

## 必要条件
- n8n: Version 1.94.0以降
- Node.js: 16.x以降推奨
- 必要なAPI認証情報:
  - OpenAI API Key
  - Tavily API Key
  - Slack Bot Token

## セットアップ手順
1. 必要なAPI認証情報を取得
2. n8nの環境構築
3. ワークフローのインポート
4. 環境変数の設定
5. ワークフローの有効化

## 使用方法
1. Slackチャンネルでメッセージを送信
2. ボットが自動的に応答を生成
3. スレッド形式で回答が返信される

## 注意事項
- APIキーは適切に管理してください
- 環境変数は`.env`ファイルで管理し、Gitにコミットしないでください
- ワークフローの設定は`n8n-workflow-specification.md`を参照してください

## ライセンス
MIT License 
# N8n Project

## 概要
このプロジェクトは、n8nを使用した様々なワークフローのコレクションです。各ワークフローは独立したディレクトリに格納され、仕様書とJSONファイルで構成されています。

## プロジェクト構造
```
n8n-project/
├── workflows/
│   ├── slack-chatgpt/          # Slack ChatGPT アシスタント
│   │   ├── slack-chatgpt-assistant.json
│   │   └── slack-chatgpt-assistant-spec.md
│   └── [other-workflows]/      # その他のワークフロー
```

## ワークフロー一覧

### Slack ChatGPT アシスタント
- **概要**: Slackメッセージを受信し、OpenAI GPTを使用してキーワード抽出とインターネット検索を行い、検索結果を基に回答を生成してSlackに返信する自動応答システム
- **機能**:
  - Slackメッセージの受信と処理
  - OpenAI GPTによるキーワード抽出
  - Tavily APIを使用したWeb検索
  - ChatGPTによる回答生成
  - Slackへの自動返信
- **必要条件**:
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

## 注意事項
- APIキーは適切に管理してください
- 環境変数は`.env`ファイルで管理し、Gitにコミットしないでください
- 各ワークフローの詳細な設定は、対応する仕様書（`*-spec.md`）を参照してください

## ライセンス
MIT License 
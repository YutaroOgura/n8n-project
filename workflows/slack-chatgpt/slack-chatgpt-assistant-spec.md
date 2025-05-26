# n8n Slack ChatGPT Workflow 仕様書

## 概要

このワークフローは、Slackメッセージを受信し、OpenAI GPTを使用してキーワード抽出とインターネット検索を行い、検索結果を基に回答を生成してSlackに返信する自動応答システムです。

## システム構成

### 対応バージョン
- **n8n**: Version 1.94.0以降
- **Node.js**: 16.x以降推奨

### 外部API依存関係
- **Slack API**: Webhook受信、メッセージ送信
- **OpenAI API**: GPT-3.5-turbo による自然言語処理
- **Tavily Search API**: Web検索機能

## ワークフロー詳細

### 1. Slack Webhook（入力ポイント）

**ノード名**: `Slack Webhook`  
**タイプ**: `n8n-nodes-base.webhook`  
**目的**: Slackからのイベントを受信

#### パラメータ
```json
{
  "httpMethod": "POST",
  "path": "slack-chatgpt",
  "responseMode": "responseNode"
}
```

#### 受信データ形式
```json
{
  "body": {
    "type": "url_verification|event_callback",
    "challenge": "string",
    "event": {
      "type": "message",
      "text": "ユーザーメッセージ",
      "channel": "チャンネルID",
      "ts": "タイムスタンプ",
      "user": "ユーザーID"
    }
  }
}
```

### 2. メッセージ整形

**ノード名**: `メッセージ整形`  
**タイプ**: `n8n-nodes-base.function`  
**目的**: Slackイベントデータの解析と正規化

#### 処理ロジック
1. **Challenge応答**: Slack URL検証要求への対応
2. **メッセージ抽出**: イベントデータからメッセージ情報を抽出

#### 出力データ構造
```json
{
  "text": "ユーザーが入力したメッセージテキスト",
  "channel": "C1234567890",
  "thread_ts": "1234567890.123456",
  "user": "U1234567890"
}
```

### 3. キーワード抽出

**ノード名**: `キーワード抽出`  
**タイプ**: `n8n-nodes-base.httpRequest`  
**目的**: OpenAI APIを使用してメッセージからキーワードを抽出

#### API仕様
- **エンドポイント**: `https://api.openai.com/v1/chat/completions`
- **メソッド**: POST
- **認証**: Bearer Token（HTTP Header Auth）

#### リクエスト構造
```json
{
  "model": "gpt-3.5-turbo",
  "messages": [
    {
      "role": "user",
      "content": "以下の文章から重要なキーワードを3個抽出してください：\n\n{ユーザーメッセージ}\n\n出力形式: keyword1, keyword2, keyword3"
    }
  ],
  "temperature": 0.3,
  "max_tokens": 100
}
```

#### レスポンス形式
```json
{
  "choices": [
    {
      "message": {
        "content": "keyword1, keyword2, keyword3"
      }
    }
  ]
}
```

### 4. Tavily検索

**ノード名**: `Tavily検索`  
**タイプ**: `n8n-nodes-base.httpRequest`  
**目的**: 抽出されたキーワードでWeb検索を実行

#### API仕様
- **エンドポイント**: `https://api.tavily.com/search`
- **メソッド**: POST
- **認証**: API Key（HTTP Header Auth）

#### リクエスト構造
```json
{
  "query": "抽出されたキーワード",
  "max_results": 3
}
```

#### レスポンス形式
```json
{
  "results": [
    {
      "title": "検索結果のタイトル",
      "url": "https://example.com/article",
      "content": "記事の要約"
    }
  ]
}
```

### 5. プロンプト生成

**ノード名**: `プロンプト生成`  
**タイプ**: `n8n-nodes-base.function`  
**目的**: 検索結果を統合して最終的な回答用プロンプトを作成

#### 処理ロジック
```javascript
const results = $json.results || [];
const summary = results.map(r => `- ${r.title}: ${r.url}`).join("\n") || '検索結果なし';
const userInput = $node["メッセージ整形"].json.text || '';

const prompt = `質問: ${userInput}

参考文献:
${summary}

これに対する親切で正確な回答を出力してください。`;

return [{ json: { prompt: prompt } }];
```

### 6. ChatGPT応答

**ノード名**: `ChatGPT応答`  
**タイプ**: `n8n-nodes-base.httpRequest`  
**目的**: 統合プロンプトを使用して最終回答を生成

#### API仕様
- **エンドポイント**: `https://api.openai.com/v1/chat/completions`
- **メソッド**: POST
- **認証**: Bearer Token（HTTP Header Auth）

#### リクエスト構造
```json
{
  "model": "gpt-3.5-turbo",
  "messages": [
    {
      "role": "user",
      "content": "統合されたプロンプト"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 500
}
```

### 7. Slackへ返信

**ノード名**: `Slackへ返信`  
**タイプ**: `n8n-nodes-base.httpRequest`  
**目的**: 生成された回答をSlackチャンネルに送信

#### API仕様
- **エンドポイント**: `https://slack.com/api/chat.postMessage`
- **メソッド**: POST
- **認証**: Bearer Token（Slack Bot Token）

#### リクエスト構造
```json
{
  "channel": "チャンネルID",
  "text": "ChatGPTからの回答",
  "thread_ts": "元メッセージのタイムスタンプ"
}
```

## 設定手順

### 1. 必要な認証情報

#### OpenAI API認証
- **タイプ**: HTTP Header Auth
- **Name**: `Authorization`
- **Value**: `Bearer sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

#### Tavily API認証
- **タイプ**: HTTP Header Auth
- **Name**: `X-API-Key`
- **Value**: `tvly-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

#### Slack Bot Token
- **タイプ**: HTTP Header Auth
- **Name**: `Authorization`
- **Value**: `Bearer xoxb-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### 2. Slack App設定

#### 必要なスコープ
```
chat:write
chat:write.public
```

#### Event Subscriptions
- **Request URL**: `https://your-n8n-instance.com/webhook/slack-chatgpt`
- **Subscribe to bot events**: `message.channels`, `message.groups`, `message.im`

### 3. 環境変数

JSONファイル内の以下の値を実際の値に置き換える：

```json
{
  "YOUR_OPENAI_HEADER_AUTH_ID": "実際のOpenAI認証ID",
  "YOUR_TAVILY_CREDENTIAL_ID": "実際のTavily認証ID",
  "YOUR_SLACK_BOT_TOKEN": "実際のSlackボットトークン"
}
```

## データフロー

```mermaid
graph TD
    A[Slack Message] --> B[Webhook Receiver]
    B --> C[Message Formatting]
    C --> D[Keyword Extraction]
    D --> E[Web Search]
    E --> F[Prompt Generation]
    F --> G[ChatGPT Response]
    G --> H[Slack Reply]
```

## エラーハンドリング

### 1. Slack Challenge応答
```javascript
if ($json.body?.type === "url_verification") {
  return [{ json: { challenge: $json.body.challenge } }];
}
```

### 2. 検索結果なしの場合
```javascript
const summary = results.map(r => `- ${r.title}: ${r.url}`).join("\n") || '検索結果なし';
```

### 3. 空メッセージの処理
```javascript
const userInput = $node["メッセージ整形"].json.text || '';
```

## パフォーマンス考慮事項

### 1. レート制限
- **OpenAI API**: 3,500 RPM (requests per minute)
- **Slack API**: 1 message per second per channel
- **Tavily API**: プランに依存

### 2. トークン使用量
- **キーワード抽出**: 約50-100トークン/リクエスト
- **応答生成**: 約200-500トークン/リクエスト

### 3. レスポンス時間
- **平均処理時間**: 3-5秒
- **タイムアウト設定**: 30秒推奨

## トラブルシューティング

### よくあるエラー

#### 1. "Install this node to use it"
- **原因**: ノードタイプまたはバージョンの非互換
- **解決**: typeVersionを1に設定

#### 2. "The value 'xxx' is not supported"
- **原因**: パラメータ値の非互換
- **解決**: APIドキュメントで正確な値を確認

#### 3. 認証エラー
- **原因**: 無効なAPIキーまたはトークン
- **解決**: 認証情報の再確認と更新

### ログ出力例

#### 正常実行
```
[INFO] Slack message received: "東京の天気は？"
[INFO] Keywords extracted: "東京, 天気, 予報"
[INFO] Search completed: 3 results found
[INFO] Response generated successfully
[INFO] Reply sent to Slack channel
```

#### エラー例
```
[ERROR] OpenAI API rate limit exceeded
[WARN] No search results found for query
[ERROR] Slack API authentication failed
```

## セキュリティ考慮事項

### 1. APIキー管理
- 環境変数またはn8n Credentialsでの安全な保存
- 定期的なローテーション

### 2. データプライバシー
- Slackメッセージの一時的な処理のみ
- 外部APIへの最小限のデータ送信

### 3. アクセス制御
- 特定のSlackチャンネルのみでの動作
- ボット権限の最小化

## 拡張可能性

### 1. 多言語対応
- プロンプトの多言語化
- 地域別検索エンジンの利用

### 2. カスタマイズ
- 業界特化型キーワード抽出
- 企業内ナレッジベースとの連携

### 3. 高度な機能
- 画像解析対応
- 音声メッセージ処理
- スレッド会話の文脈理解
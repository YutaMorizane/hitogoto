# ユニット依存関係

## 依存関係マトリクス

以下にユニット間の依存関係マトリクスを表に示す。

| 依存元 ↓ ＼ 依存先 → | UOW-1 | UOW-2 | UOW-3 | UOW-4 | 外部サービス |
| --- | --- | --- | --- | --- | --- |
| UOW-1（フロントエンド） | - | ○（REST経由） | ○（REST経由） | ○（WebSocket経由） | Transcribe Streaming（直接） |
| UOW-2（AI処理） | - | - | ○（データ取得） | - | Amazon Bedrock |
| UOW-3（データ管理） | - | - | - | - | DynamoDB, Cognito |
| UOW-4（インフラ+RT基盤） | - | ○（Lambda呼出） | ○（Lambda呼出） | - | API Gateway, STS |

## 依存方向図

```
UOW-1 (フロントエンド)
  │
  ├──[REST API]──→ UOW-3 (データ・セッション管理)
  │                    │
  ├──[REST API]──→ UOW-2 (AI処理) ←──[Lambda呼出]── UOW-4
  │                    │
  │                    └──[データ取得]──→ UOW-3
  │
  ├──[WebSocket]──→ UOW-4 (インフラ+リアルタイム基盤)
  │                    │
  │                    └──[Lambda呼出]──→ UOW-3
  │
  └──[直接接続]──→ Amazon Transcribe Streaming
```

## 通信パターン詳細

| 経路 | プロトコル | パターン | レイテンシ要件 |
| --- | --- | --- | --- |
| UOW-1 → UOW-3 | REST API (HTTPS) | 同期リクエスト/レスポンス | - |
| UOW-1 → UOW-2 | REST API (HTTPS) | 同期リクエスト（生成開始トリガー） | - |
| UOW-4 → UOW-1 | WebSocket | 非同期プッシュ（ストリーミング） | 500ms以内 |
| UOW-4 → UOW-2 | Lambda呼び出し | 同期（内部） | - |
| UOW-4 → UOW-3 | Lambda呼び出し | 同期（内部） | - |
| UOW-2 → UOW-3 | 直接呼び出し（shared/db） | 同期 | - |
| UOW-1 → Transcribe | WebSocket（直接） | ストリーミング | 2秒以内 |

## 開発順序の制約

ユニット間の依存関係に基づく推奨開発順序：

1. **UOW-4（インフラ）**: 最初に着手。他全ユニットが依存するAWSリソースを定義
2. **UOW-3（データ管理）**: インフラ構築後に着手。DynamoDBテーブル・Cognito設定が必要
3. **UOW-2（AI処理）**: データ管理と並行可能。Bedrock APIの呼び出しは独立してテスト可能
4. **UOW-1（フロントエンド）**: バックエンドAPIが揃った後に結合。ただしモックAPIで先行開発可能

### 並行開発の可能性

- UOW-1はモックAPI/モックWebSocketで先行開発可能
- UOW-2はBedrock APIのみに依存するため、UOW-3/4と並行開発可能
- UOW-3はDynamoDB Local等でローカル開発可能
- UOW-4のCDKスタック定義は他ユニットと並行して進められる

## デプロイ依存関係

| デプロイ順序 | ユニット | 理由 |
| --- | --- | --- |
| 1 | UOW-4（CDKスタック） | Lambda、API Gateway、DynamoDB等のリソースを先に作成 |
| 2 | UOW-3 + UOW-2（Lambda関数） | CDKで定義されたLambdaにコードをデプロイ |
| 3 | UOW-1（フロントエンド） | S3にビルド成果物をデプロイ、CloudFrontキャッシュ無効化 |

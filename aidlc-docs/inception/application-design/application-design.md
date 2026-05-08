# アプリケーション設計書（統合）

## 設計概要

本書はHITOGOTO ～しらんけどエージェント～ のアプリケーション設計を統合的にまとめたものである。詳細は各個別設計書を参照すること。

## アーキテクチャ概要

### 全体構成

```
+------------------+     REST API      +------------------+
|                  | -----------------> |                  |
|   フロントエンド   |                    |  データ・セッション  |
|   (React SPA)   | <--- WebSocket --- |  管理 (Lambda)   |
|                  |                    |                  |
+------------------+     REST API      +------------------+
        |            -----------------> |                  |
        |                               |   AI処理         |
        |            <--- WebSocket --- |   (Lambda)       |
        |                               |                  |
        |                               +------------------+
        |
        |  WebSocket(直接)
        v
+------------------+
| Transcribe       |
| Streaming        |
+------------------+
```

### 設計判断サマリ

| 判断項目 | 採用方針 | 根拠 |
| --- | --- | --- |
| ページ構成 | マルチページ（ダッシュボード/会議/設定/履歴） | 機能の明確な分離、ユーザー体験の整理 |
| プロンプト管理 | 共通プロンプト + モード別パラメータ注入 | 拡張性と保守性のバランス。モード追加時にパラメータ追加のみで対応可能 |
| AI生成通信 | ハイブリッド（REST送信 + WebSocketストリーミング受信） | リクエストの信頼性（REST）とリアルタイム性（WebSocket）を両立 |
| DynamoDB設計 | テーブル分離（ユーザー/会議/発言/ダメ人間度） | 各テーブルのアクセスパターンが異なるため独立管理が適切 |
| 音声処理 | クライアント直接Transcribe接続（STS一時認証情報使用） | レイテンシ最小化（サーバー経由の往復を排除） |

## コンポーネント一覧

詳細は `components.md` を参照。

| ユニット | コンポーネント数 | 主要コンポーネント |
| --- | --- | --- |
| UOW-1: フロントエンド | 10 | 会議画面、音声キャプチャ、WebSocket接続、状態管理 |
| UOW-2: AI処理 | 5 | プロンプトマネージャー、代弁案/リカバリー案/サマリジェネレーター、モード推奨 |
| UOW-3: データ・セッション管理 | 5 | ユーザー管理、会議セッション、発言履歴、ダメ人間度、同意管理 |
| UOW-4: インフラ+リアルタイム基盤 | 4 | CDKスタック、WebSocket接続マネージャー、音声処理ブリッジ、CI/CD |

## サービス層

詳細は `services.md` を参照。

| サービス | オーケストレーション対象 |
| --- | --- |
| S-001: 代弁サービス | 入力受付→プロンプト構築→生成→承認→記録 |
| S-002: リカバリーサービス | 反応検知→生成→プッシュ配信 |
| S-003: 会議セッションサービス | 開始→進行→終了→サマリ生成 |
| S-004: 認証サービス | 認証→トークン管理→一時認証情報発行 |
| S-005: リアルタイム通信サービス | 接続管理→イベント配信 |
| S-006: 文字起こしサービス | 文字起こし結果受信→蓄積→文脈提供 |

## 依存関係

詳細は `component-dependency.md` を参照。

### ユニット間の依存方向

```
UOW-1(フロントエンド) ---> UOW-4(インフラ+リアルタイム基盤)
                      ---> UOW-3(データ・セッション管理) [REST API経由]
                      ---> Transcribe Streaming [直接接続]

UOW-4(インフラ+リアルタイム基盤) ---> UOW-2(AI処理) [Lambda呼び出し]
                                ---> UOW-3(データ・セッション管理) [Lambda呼び出し]

UOW-2(AI処理) ---> UOW-3(データ・セッション管理) [データ取得]
              ---> Amazon Bedrock [外部サービス]

UOW-3(データ・セッション管理) ---> Amazon DynamoDB [外部サービス]
                              ---> Amazon Cognito [外部サービス]
```

## DynamoDBテーブル設計（概要）

| テーブル名 | PK | SK | 用途 |
| --- | --- | --- | --- |
| Users | user_id | - | ユーザープロファイル、設定 |
| Sessions | session_id | - | 会議セッション情報、参加者 |
| Statements | session_id | statement_id | 発言履歴、責任帰属、訂正履歴 |
| Transcripts | session_id | timestamp | 文字起こし結果（話者識別付き） |
| DameScore | user_id | date | ダメ人間度スコア、推移データ |
| Connections | connection_id | - | WebSocket接続管理 |

## 関連ドキュメント

- コンポーネント定義: `components.md`
- コンポーネントメソッド: `component-methods.md`
- サービス層定義: `services.md`
- コンポーネント依存関係: `component-dependency.md`

---

## 将来のAmazon Bedrock AgentCore移行パス

### 現行アーキテクチャ（Phase 1）

Phase 1ではBedrock InvokeModel API + Lambda + API Gateway WebSocketの構成を採用する。理由は以下の通りである：

- レイテンシ要件（3〜5秒）に対してLambda直接呼び出しの方が制御が容易
- ハッカソン期間の制約上、既知のAWSサービスの方がリスクが低い
- チームの学習コストを最小化できる

### AgentCore移行の判断基準

以下の条件が満たされた場合にAgentCore移行を検討する：

- Phase 1のMVPデモが安定稼働している
- チームがAgentCoreの学習に時間を割ける
- AgentCoreのコールドスタートレイテンシがリアルタイム要件を満たすことが検証できた

### Phase別移行計画

| Phase | 移行対象 | AgentCoreサービス | 移行内容 |
| --- | --- | --- | --- |
| Phase 2 | パーソナルDB（FR-008） | Memory | DynamoDBによる自前実装 → AgentCore Memoryサービス（短期+長期記憶）に置換 |
| Phase 2 | 依存度エスカレーション（FR-013） | Runtime | エージェント自律度の段階的制御にRuntimeのセッション管理を活用 |
| Phase 3 | デジタルツイン・外部連携 | Gateway | 外部サービス（Slack、JIRA等）との連携にMCP Gatewayを活用 |
| Phase 3 | AI処理全体（UOW-2） | Runtime | Lambda + API Gateway → AgentCore Runtimeへの載せ替え |
| 将来 | 認証基盤 | Identity | Cognito直接連携 → AgentCore Identityによる統合認証管理 |
| 将来 | 監視・トレース | Observability | CloudWatch → AgentCore Observability（OTEL互換）に統合 |

### 移行を容易にするための設計方針（Phase 1で遵守）

1. **AI処理の独立性確保**: UOW-2（AI処理）を他ユニットから疎結合に保つ。Bedrock API呼び出しをC-AI-002内にカプセル化し、将来のRuntime移行時に影響範囲を最小化する。
2. **プロンプト管理の抽象化**: C-AI-001（プロンプトマネージャー）をインターフェース化し、将来AgentCoreのMemoryサービスからコンテキストを取得する形に差し替え可能にする。
3. **セッション管理の抽象化**: C-DATA-002（会議セッション管理）のインターフェースを、将来AgentCore Runtimeのセッション管理に置換可能な形で設計する。
4. **WebSocket通信の分離**: C-INFRA-002（WebSocket接続マネージャー）を独立モジュールとし、将来AgentCore Runtimeの双方向ストリーミングに置換可能にする。

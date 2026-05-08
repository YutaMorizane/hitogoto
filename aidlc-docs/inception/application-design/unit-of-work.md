# ユニット定義

## ユニット一覧

| ユニットID | 名称 | 担当範囲 | 主要技術 |
| --- | --- | --- | --- |
| UOW-1 | フロントエンド | React UI、音声取得、WebSocket接続、状態管理 | Vite + React + TypeScript + Zustand + Serendie DS |
| UOW-2 | AI処理 | Bedrock呼び出し全般（代弁案、リカバリー案、サマリ、モード推奨） | Python + Amazon Bedrock (Claude) |
| UOW-3 | データ・セッション管理 | DynamoDB操作、認証、会議CRUD、ダメ人間度算出、同意管理 | Python + DynamoDB + Cognito |
| UOW-4 | インフラ + リアルタイム基盤 | CDKスタック、WebSocket管理、Transcribe連携、CI/CD | Python + AWS CDK + API Gateway WebSocket |

## チーム方針

- **担当割り当て**: 機能横断（全員が全ユニットに関与し、ストーリー単位で担当を決める）
- **チーム規模**: 4人

---

## UOW-1: フロントエンド

### 責務

- マルチページ構成のSPA（ダッシュボード/会議/設定/履歴）
- 会議中のHUD形式リアルタイム操作UI
- 音声キャプチャ（マイク + Screen Capture）
- WebSocket接続管理（バックエンドとの双方向通信）
- Transcribe Streamingへの直接接続
- Zustandによる状態管理

### コンポーネント

C-FE-001〜C-FE-010

### 設計制約

- 全UIコンポーネントはSerendie Design Systemに準拠する

---

## UOW-2: AI処理

### 責務

- Amazon Bedrock（Claude）を使用したAI生成全般
- プロンプト管理（共通プロンプト + モード別パラメータ注入）
- 代弁案生成（ストリーミング、5秒以内）
- 即時リカバリー案生成（3秒以内）
- 会議サマリ・意思決定ログ・次アクション生成
- モード自動推奨

### コンポーネント

C-AI-001〜C-AI-005

### 設計制約

- AI処理を他ユニットから疎結合に保つ（将来のAgentCore移行を容易にするため）
- Bedrock API呼び出しをC-AI-002内にカプセル化する

---

## UOW-3: データ・セッション管理

### 責務

- ユーザー認証（Cognito連携）
- 会議セッションのCRUD
- 発言履歴の保存・責任帰属管理
- ダメ人間度の算出・保存
- 透明性・同意管理
- 参加者プロファイル管理

### コンポーネント

C-DATA-001〜C-DATA-005

### DynamoDBテーブル

| テーブル名 | PK | SK | 用途 |
| --- | --- | --- | --- |
| Users | user_id | - | ユーザープロファイル、設定 |
| Sessions | session_id | - | 会議セッション情報、参加者 |
| Statements | session_id | statement_id | 発言履歴、責任帰属、訂正履歴 |
| Transcripts | session_id | timestamp | 文字起こし結果（話者識別付き） |
| DameScore | user_id | date | ダメ人間度スコア、推移データ |
| Connections | connection_id | - | WebSocket接続管理 |

---

## UOW-4: インフラ + リアルタイム基盤

### 責務

- AWS CDKによる全リソース定義・デプロイ
- WebSocket接続のサーバー側管理（接続/切断/メッセージルーティング）
- Transcribe Streaming用一時認証情報の発行
- CI/CDパイプライン（AWS Codeシリーズ）
- CloudFront + S3によるフロントエンドホスティング

### コンポーネント

C-INFRA-001〜C-INFRA-004

---

## コード構成（モノレポ）

```
hitogoto/
├── frontend/                      ← UOW-1
│   ├── src/
│   │   ├── pages/                 ← ページコンポーネント
│   │   │   ├── Dashboard.tsx
│   │   │   ├── Meeting.tsx
│   │   │   ├── Settings.tsx
│   │   │   └── History.tsx
│   │   ├── components/            ← 共通UIコンポーネント
│   │   │   ├── HudOverlay.tsx
│   │   │   ├── InputPanel.tsx
│   │   │   ├── StatementCard.tsx
│   │   │   ├── ModeSelector.tsx
│   │   │   ├── TranscriptPanel.tsx
│   │   │   └── StatusBar.tsx
│   │   ├── hooks/                 ← カスタムフック
│   │   ├── stores/                ← Zustandストア
│   │   ├── services/              ← API/WebSocket接続
│   │   └── utils/                 ← ユーティリティ
│   ├── package.json
│   ├── vite.config.ts
│   └── tsconfig.json
├── backend/                       ← UOW-2 + UOW-3
│   ├── functions/                 ← Lambda関数（関数単位で分離）
│   │   ├── generate_statement/    ← AI代弁案生成
│   │   ├── generate_recovery/     ← リカバリー案生成
│   │   ├── generate_summary/      ← サマリ生成
│   │   ├── recommend_mode/        ← モード推奨
│   │   ├── create_session/        ← 会議セッション作成
│   │   ├── end_session/           ← 会議セッション終了
│   │   ├── save_statement/        ← 発言保存
│   │   ├── update_attribution/    ← 責任帰属変更
│   │   ├── dame_score/            ← ダメ人間度算出
│   │   ├── save_transcript/       ← 文字起こし結果保存
│   │   ├── ws_connect/            ← WebSocket接続
│   │   ├── ws_disconnect/         ← WebSocket切断
│   │   ├── ws_message/            ← WebSocketメッセージ処理
│   │   └── get_transcribe_creds/  ← Transcribe一時認証情報
│   ├── shared/                    ← 共通ライブラリ
│   │   ├── prompts/               ← プロンプトテンプレート
│   │   ├── models/                ← データモデル定義
│   │   ├── db/                    ← DynamoDB操作ユーティリティ
│   │   └── utils/                 ← 共通ユーティリティ
│   ├── tests/                     ← テスト
│   │   ├── unit/                  ← ユニットテスト
│   │   └── pbt/                   ← プロパティベーステスト
│   └── pyproject.toml
├── infra/                         ← UOW-4
│   ├── stacks/
│   │   ├── api_stack.py           ← API Gateway（REST + WebSocket）
│   │   ├── compute_stack.py       ← Lambda関数
│   │   ├── database_stack.py      ← DynamoDB
│   │   ├── auth_stack.py          ← Cognito
│   │   ├── storage_stack.py       ← S3
│   │   ├── cdn_stack.py           ← CloudFront
│   │   └── pipeline_stack.py      ← CI/CD
│   ├── app.py
│   └── cdk.json
├── docs/                          ← ドキュメント
├── aidlc-docs/                    ← AI-DLCドキュメント
└── .gitignore
```

# ユニット分割プラン

## 概要

実行計画で確定した機能ドメイン軸の4ユニット分割に基づき、ユニット定義・依存関係・ストーリーマッピングを生成する。

## 生成ステップ

- [x] Step 1: ユニット定義（unit-of-work.md）
- [x] Step 2: ユニット依存関係（unit-of-work-dependency.md）
- [x] Step 3: ストーリー→ユニットマッピング（unit-of-work-story-map.md）
- [x] Step 4: 検証（全ストーリーがユニットに割当済みであることを確認）

## 確定済みユニット分割

実行計画およびアプリケーション設計で以下が確定している：

| ユニット | 担当範囲 | コンポーネント |
| --- | --- | --- |
| UOW-1: フロントエンド | React UI、音声取得、WebSocket接続、状態管理 | C-FE-001〜010 |
| UOW-2: AI処理 | Bedrock呼び出し全般 | C-AI-001〜005 |
| UOW-3: データ・セッション管理 | DynamoDB、認証、CRUD、ダメ人間度 | C-DATA-001〜005 |
| UOW-4: インフラ+リアルタイム基盤 | CDK、WebSocket管理、Transcribe、CI/CD | C-INFRA-001〜004 |

## 質問事項

### Question 1
グリーンフィールドプロジェクトのディレクトリ構成について、どの構成を採用しますか？

A) モノレポ構成（1リポジトリに全ユニットを配置。ルートにfrontend/、backend/、infra/を配置）
B) マルチレポ構成（ユニットごとに独立リポジトリ）
C) モノレポ + パッケージ管理（npm workspacesやpoetry等でユニット間依存を管理）
D) Other (please describe after [Answer]: tag below)

[Answer]: A) モノレポ構成（1リポジトリに全ユニットを配置。ルートにfrontend/、backend/、infra/を配置）

### Question 2
バックエンドのディレクトリ構成について、UOW-2（AI処理）とUOW-3（データ管理）をどのように配置しますか？

A) 完全分離（backend/ai-processing/、backend/data-management/ として独立）
B) 共通ルート配下にモジュール分離（backend/src/ai/、backend/src/data/）
C) Lambda関数単位で分離（backend/functions/generate-statement/、backend/functions/create-session/ 等）
D) Other (please describe after [Answer]: tag below)

[Answer]: C) Lambda関数単位で分離（backend/functions/generate-statement/、backend/functions/create-session/ 等）

### Question 3
4人チームでのユニット担当割り当てについて、どのような方針ですか？

A) 1人1ユニット固定（各自が主担当ユニットを持つ）
B) フロントエンド2人 + バックエンド2人（UOW-1に2人、UOW-2〜4に2人）
C) 機能横断（全員が全ユニットに関与し、ストーリー単位で担当を決める）
D) Other (please describe after [Answer]: tag below)

[Answer]: C) 機能横断（全員が全ユニットに関与し、ストーリー単位で担当を決める）


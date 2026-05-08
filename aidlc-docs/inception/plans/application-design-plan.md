# アプリケーション設計プラン

## 概要

要件定義書（FR-001〜FR-013、NFR-001〜006）とユーザーストーリー（US-001〜US-033）に基づき、HITOGOTOのアプリケーション設計を行う。

## 設計ステップ

- [x] Step 1: コンポーネント定義（components.md）
- [x] Step 2: コンポーネントメソッド定義（component-methods.md）
- [x] Step 3: サービス層定義（services.md）
- [x] Step 4: コンポーネント依存関係定義（component-dependency.md）
- [x] Step 5: 統合設計書作成（application-design.md）

## 設計方針

### コンポーネント分割の基本方針

実行計画のユニット分割（機能ドメイン軸）に沿い、以下の4ドメインでコンポーネントを整理する：

- **UOW-1（フロントエンド）**: UI層のコンポーネント
- **UOW-2（AI処理）**: LLM呼び出し・プロンプト管理のコンポーネント
- **UOW-3（データ・セッション管理）**: データ永続化・認証・ビジネスロジックのコンポーネント
- **UOW-4（インフラ + リアルタイム基盤）**: 通信基盤・音声処理・インフラ定義のコンポーネント

---

## 質問事項

以下の質問に回答してください。各質問の [Answer]: タグの後に選択肢の文字を記入してください。

### Question 1
フロントエンドのページ構成について、どのような画面構成を想定していますか？

A) シングルページ構成（会議画面のみ。ログイン後は即会議画面）
B) マルチページ構成（ダッシュボード、会議画面、設定画面、履歴画面を分離）
C) ダッシュボード + 会議画面の2画面構成（設定は会議画面内に統合）
D) Other (please describe after [Answer]: tag below)

[Answer]: B) マルチページ構成（ダッシュボード、会議画面、設定画面、履歴画面を分離）

### Question 2
AI処理コンポーネントの設計パターンについて、プロンプト管理をどのように構成しますか？

A) モードごとに独立したプロンプトテンプレートを持つ（代弁/フォロー/言い換え/しらんけど各々）
B) 共通プロンプト + モード別パラメータ注入（ベースプロンプトにモード指示を追加）
C) AI-DLCの判断に任せる
D) Other (please describe after [Answer]: tag below)

[Answer]: C) AI-DLCの判断に任せる

### Question 3
フロントエンドとバックエンド間の通信パターンについて、AI代弁案の生成はどのパターンで行いますか？

A) REST API（リクエスト→レスポンス待ち。生成完了まで待機）
B) WebSocket経由（リクエスト送信→生成完了時にプッシュ通知）
C) REST APIでリクエスト送信 + WebSocketで結果をストリーミング受信（ハイブリッド）
D) Other (please describe after [Answer]: tag below)

[Answer]: C) REST APIでリクエスト送信 + WebSocketで結果をストリーミング受信（ハイブリッド）

### Question 4
DynamoDBのテーブル設計方針について、どのアプローチを採用しますか？

A) シングルテーブルデザイン（1テーブルに全エンティティを格納、GSIで検索）
B) テーブル分離（ユーザー、会議セッション、発言履歴、ダメ人間度を別テーブル）
C) AI-DLCの判断に任せる
D) Other (please describe after [Answer]: tag below)

[Answer]: B) テーブル分離（ユーザー、会議セッション、発言履歴、ダメ人間度を別テーブル）

### Question 5
会議中のリアルタイム音声処理のアーキテクチャについて、音声データの流れをどう設計しますか？

A) ブラウザ → WebSocket → Lambda → Transcribe（サーバー経由）
B) ブラウザ → 直接Transcribe Streaming（クライアント直接接続、一時認証情報を使用）
C) AI-DLCの判断に任せる
D) Other (please describe after [Answer]: tag below)

[Answer]: C) AI-DLCの判断に任せる


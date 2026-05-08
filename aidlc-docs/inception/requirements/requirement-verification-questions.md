# 要件確認質問

以下の質問に回答してください。各質問の [Answer]: タグの後に選択肢の文字を記入してください。

## Question 1
ハッカソンの開発期間（制限時間）はどの程度ですか？

A) 1日（8時間程度）
B) 2日（16時間程度）
C) 3日以上
D) 未定・不明
E) Other (please describe after [Answer]: tag below)

[Answer]: E)Inception フェーズの成果物が2026/5/10まで。MVPデモまでは2026/5/30までとなります

## Question 2
Phase 1（デモ必須）の実装範囲について、最も重要視する機能はどれですか？

A) AI代弁機能（ユーザーの代わりにAIが発言する）
B) リアルタイム音声文字起こし（Amazon Transcribe Streaming）
C) AI提案モード切替（代弁/フォロー/言い換えの切替）
D) 上記すべてを均等に重要視する
E) Other (please describe after [Answer]: tag below)

[Answer]: A) AI代弁機能（ユーザーの代わりにAIが発言する）

## Question 3
フロントエンドのフレームワークについて、Next.jsとViteのどちらを採用しますか？

A) Next.js（SSR/SSG対応、ルーティング内蔵）
B) Vite + React（軽量SPA、高速ビルド）
C) AI-DLCの判断に任せる
D) Other (please describe after [Answer]: tag below)

[Answer]: B) Vite + React（軽量SPA、高速ビルド）

## Question 4
状態管理ライブラリについて、どちらを採用しますか？

A) Zustand（軽量、シンプル）
B) Redux Toolkit（堅牢、大規模向け）
C) AI-DLCの判断に任せる
D) Other (please describe after [Answer]: tag below)

[Answer]: C) AI-DLCの判断に任せる

## Question 5
認証機能（Amazon Cognito）はPhase 1で実装しますか？

A) はい、Phase 1で実装する（ユーザー登録・ログイン必須）
B) いいえ、Phase 1ではスキップし、固定ユーザーで動作させる
C) 簡易認証のみ（パスワードなしのユーザー識別程度）
D) Other (please describe after [Answer]: tag below)

[Answer]: A) はい、Phase 1で実装する（ユーザー登録・ログイン必須）

## Question 6
デモシナリオにおける「会議」の形態について確認します。実際のTeams等のビデオ会議に参加しながら本アプリを併用する想定ですか？

A) はい、Teams等のビデオ会議と本アプリを同時に使用する（Screen Capture APIで相手の音声を取得）
B) いいえ、本アプリ単体で会議シミュレーションを行う（デモ用にモック音声を使用）
C) 両方対応するが、デモではモック音声を使用する
D) Other (please describe after [Answer]: tag below)

[Answer]: A) はい、Teams等のビデオ会議と本アプリを同時に使用する（Screen Capture APIで相手の音声を取得）

## Question 7
IaC（AWS CDK）によるインフラ構築はPhase 1の範囲に含めますか？

A) はい、CDKでインフラを定義しデプロイする
B) いいえ、Phase 1ではAWSコンソールから手動構築し、CDKは後回し
C) SAM（Serverless Application Model）を使用する
D) Other (please describe after [Answer]: tag below)

[Answer]: A) はい、CDKでインフラを定義しデプロイする

## Question 8
Serendie Design Systemの利用について確認します。このデザインシステムのパッケージ名やインストール方法は把握していますか？

A) はい、npmパッケージとして利用可能（パッケージ名を知っている）
B) いいえ、調査が必要
C) Serendie Design Systemは必須要件ではなく、代替UIライブラリでも可
D) Other (please describe after [Answer]: tag below)

[Answer]: A) はい、npmパッケージとして利用可能（パッケージ名を知っている）

## Question 9
Working Backwardsの成果物（プレスリリース、FAQ、ユーザーマニュアル）はInceptionフェーズで作成しますか？

A) はい、Inceptionフェーズの最後に作成する
B) いいえ、開発完了後に作成する
C) プレスリリースのみInceptionで作成し、他は後回し
D) 今回のAI-DLC開発では省略する
E) Other (please describe after [Answer]: tag below)

[Answer]: C) プレスリリースのみInceptionで作成し、他は後回し

## Question 10
ハッカソンのチーム構成について教えてください。

A) 1人（個人開発）
B) 2〜3人チーム
C) 4〜5人チーム
D) 6人以上
E) Other (please describe after [Answer]: tag below)

[Answer]: E) 4人チーム

## Question 11: セキュリティ拡張
本プロジェクトにセキュリティ拡張ルールを適用しますか？

A) はい — すべてのセキュリティルールをブロッキング制約として適用する（本番グレードのアプリケーション向け推奨）
B) いいえ — すべてのセキュリティルールをスキップする（PoC、プロトタイプ、実験的プロジェクト向け）
C) Other (please describe after [Answer]: tag below)

[Answer]: B) いいえ — すべてのセキュリティルールをスキップする（PoC、プロトタイプ、実験的プロジェクト向け）

## Question 12: プロパティベーステスト拡張
本プロジェクトにプロパティベーステスト（PBT）ルールを適用しますか？

A) はい — すべてのPBTルールをブロッキング制約として適用する（ビジネスロジック、データ変換、シリアライゼーション、ステートフルコンポーネントを持つプロジェクト向け推奨）
B) 部分的 — 純粋関数とシリアライゼーションのラウンドトリップのみPBTルールを適用する
C) いいえ — すべてのPBTルールをスキップする（シンプルなCRUDアプリケーション、UIのみのプロジェクト、薄い統合レイヤー向け）
D) Other (please describe after [Answer]: tag below)

[Answer]: A) はい — すべてのPBTルールをブロッキング制約として適用する（ビジネスロジック、データ変換、シリアライゼーション、ステートフルコンポーネントを持つプロジェクト向け推奨）

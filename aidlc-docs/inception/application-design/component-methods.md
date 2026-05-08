# コンポーネントメソッド定義

※ 詳細なビジネスルールはFunctional Design（Construction Phase）で定義する。ここではメソッドシグネチャと高レベルの目的を定義する。

---

## UOW-2: AI処理

### C-AI-001: プロンプトマネージャー

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| build_prompt(mode, context, participants, target) | モード種別, 会議文脈, 参加者リスト, 宛先 | 構築済みプロンプト文字列 | モード・文脈・参加者に応じたプロンプトを構築 |
| inject_mode_params(base_prompt, mode) | ベースプロンプト, モード種別 | モード適用済みプロンプト | モード別パラメータをベースプロンプトに注入 |
| inject_participant_context(prompt, participants, target) | プロンプト, 参加者リスト, 宛先 | 参加者コンテキスト適用済みプロンプト | 宛先の特性に応じたトーン調整指示を追加 |

### C-AI-002: 代弁案ジェネレーター

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| generate(user_input, prompt, session_context) | ユーザー入力, プロンプト, セッション情報 | AI代弁案テキスト（ストリーミング） | Bedrock APIを呼び出し代弁案を生成 |
| regenerate(user_input, prompt, feedback) | ユーザー入力, プロンプト, フィードバック | 再生成された代弁案テキスト | 却下後の再生成 |

### C-AI-003: リカバリー案ジェネレーター

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| detect_reaction(transcript_segment) | 直近の文字起こしセグメント | 反応種別（反論/困惑/中立） | 相手の反応を分析 |
| generate_recovery(reaction_type, context, original_statement) | 反応種別, 会議文脈, 元の発言 | リカバリー案3パターン（言い換え/撤回/補足） | 即時リカバリー案を生成 |

### C-AI-004: サマリジェネレーター

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| generate_summary(full_transcript, session_info) | 全文字起こし, セッション情報 | サマリ（要点/決定事項/未解決/次アクション） | 会議終了後のサマリ生成 |
| generate_decision_log(full_transcript) | 全文字起こし | 意思決定ログ | 意思決定の抽出 |
| generate_action_items(full_transcript, participants) | 全文字起こし, 参加者リスト | アクションアイテムリスト | 次アクションの抽出 |

### C-AI-005: モード推奨エンジン

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| recommend_mode(context, current_mode) | 会議文脈, 現在のモード | 推奨モード + 推奨理由 | 最適モードの推奨 |

---

## UOW-3: データ・セッション管理

### C-DATA-001: ユーザー管理

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| verify_token(token) | 認証トークン | ユーザー情報 or エラー | トークン検証 |
| get_user_profile(user_id) | ユーザーID | ユーザープロファイル | プロファイル取得 |
| update_user_settings(user_id, settings) | ユーザーID, 設定値 | 更新結果 | 設定更新 |

### C-DATA-002: 会議セッション管理

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| create_session(user_id, meeting_name, participants) | ユーザーID, 会議名, 参加者リスト | セッションID | セッション作成 |
| end_session(session_id) | セッションID | 終了結果 | セッション終了 |
| get_session(session_id) | セッションID | セッション情報 | セッション取得 |
| list_sessions(user_id, limit, offset) | ユーザーID, ページング | セッション一覧 | 履歴一覧取得 |
| save_summary(session_id, summary) | セッションID, サマリデータ | 保存結果 | サマリ保存 |

### C-DATA-003: 発言履歴管理

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| save_statement(session_id, statement) | セッションID, 発言データ | 発言ID | 発言保存 |
| update_attribution(statement_id, attribution) | 発言ID, 帰属先 | 更新結果 | 責任帰属変更 |
| update_statement(statement_id, corrected_content) | 発言ID, 訂正内容 | 更新結果 | 発言訂正 |
| get_statements(session_id) | セッションID | 発言一覧 | 発言履歴取得 |

### C-DATA-004: ダメ人間度管理

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| increment_score(user_id, session_id) | ユーザーID, セッションID | 更新後スコア | スコア加算 |
| calculate_ratio(user_id, session_id) | ユーザーID, セッションID | AI任せ割合 | 割合算出 |
| get_level(user_id) | ユーザーID | ダメ人間度レベル（★表示） | レベル取得 |
| get_history(user_id) | ユーザーID | 推移データ | 推移取得 |
| get_responsibility_log(user_id) | ユーザーID | 責任引き受け履歴 | 履歴取得 |

### C-DATA-005: 同意管理

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| record_consent(session_id, participant, status) | セッションID, 参加者, 同意状態 | 記録結果 | 同意記録 |
| get_consent_status(session_id) | セッションID | 参加者別同意状態 | 同意状態取得 |
| update_data_policy(user_id, policy) | ユーザーID, ポリシー設定 | 更新結果 | データポリシー更新 |

---

## UOW-4: インフラ + リアルタイム基盤

### C-INFRA-002: WebSocket接続マネージャー

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| on_connect(connection_id, user_id) | 接続ID, ユーザーID | 接続結果 | 接続確立 |
| on_disconnect(connection_id) | 接続ID | 切断結果 | 切断処理 |
| send_message(connection_id, message) | 接続ID, メッセージ | 送信結果 | メッセージ送信 |
| broadcast_to_session(session_id, message) | セッションID, メッセージ | 送信結果 | セッション内ブロードキャスト |

### C-INFRA-003: 音声処理ブリッジ

| メソッド | 入力 | 出力 | 目的 |
| --- | --- | --- | --- |
| get_transcribe_credentials(user_id) | ユーザーID | 一時認証情報（AccessKey, SecretKey, SessionToken） | Transcribe用一時認証情報発行 |
| get_audio_config() | なし | 音声設定（言語, サンプリングレート等） | 音声処理設定取得 |

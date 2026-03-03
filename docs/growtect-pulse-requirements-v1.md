# Growtect Pulse — 統合システム要件定義書 v1.0

## コンテキスト（なぜ作るか）

日本の中小企業は「情シス担当者」に依存した属人的なIT運用を行っており、入退社・PC手配・アカウント管理・障害対応が担当者のスキルと記憶に紐づいている。Growtect Pulseはこの構造を破壊し、「管理」という人的負荷をAIとシステムに移管することで、人間が本来の業務に集中できる環境を実現する。

単なるチャットボットやSaaSではなく、**AIを「脳」・既存SaaS群を「手足」として統制する「自律型コーポレートOS（Autonomous Corporate OS）」**として設計する。

---

## 1. システムアーキテクチャ（4層構造）

```
L1 思考・判断層   │ AIアドバイザー（ITIL 4ネイティブな意思決定）
L2 制御・指示層   │ AIエグゼキューター（ワークフロー自動化・API連携）
L3 実装・実行層   │ DRESS CODE / Intune / Google Endpoint（ポリシー自動適用）
L4 物理・実行層   │ KDDIまとめてオフィス API（サイバーフィジカル同期）
```

Pulseは「自社開発のHUB」として機能し、各層の外部システムとAPI通信を行う。

---

## 2. 機能要件

### 2-1. Pulse Desk（従業員向けヘルプデスク窓口）

**インターフェース**
- Slack / Teams / Discord 上での自然言語による申請受付
- フロントエンドの対話・自動回答には **Zooba**（既存SaaS）を活用
  - ZoobaはSlack/Teamsにネイティブ統合され、社内ナレッジ（Notion/Confluenceなど）を参照して一次回答する
  - Zoobaが解決できない依頼のみPulseオーケストレーターへエスカレーション

**AIの主な責務**
- 不足項目のヒアリング（追加情報の自動収集）
- 設定・情報の確認（Read権限による台帳参照）
- 申請アシスト（下書き生成・フォーム補完）
- 進捗通知・完了連絡のSlackスレッド返信

**チケット管理**
- 正本：**Pulse自前DB**（PostgreSQL）に「最小で成立する依頼レコード」を保持
- LMIS（ユニリタ）はインシデント管理・CMDB・承認ワークフローの詳細管理に連携（参照/書き込みはAPI経由）
- ステータス遷移：`受付 → 分析中 → 承認待ち → 承認済み → 実行中 → 完了 / 却下 / 否認 / エスカレーション`
- チケットに紐づく情報：会話サマリー・AI判断根拠・監査ログ・承認者・実行パラメータ

### 2-2. Agent Console（運用者向け管理画面）

- キュー管理（承認待ちタスクの一覧・優先度表示）
- タイムライン表示（チケットの状態遷移履歴）
- AIが作成した下書き・提案の確認・編集
- 引き継ぎ機能（担当者変更）
- 監査ログの閲覧・エクスポート

**主要画面構成（Next.js 15 + shadcn/ui）**

| 画面 | 主要コンポーネント | 備考 |
|---|---|---|
| ダッシュボード | 承認待ちカウント・直近チケット一覧・LLMコスト推移グラフ | トップページ |
| 承認キュー | ActionProposal カード（提案内容・リスクスコア・承認/却下ボタン）| Write 操作ごとに1カード |
| チケット詳細 | 状態遷移タイムライン・AIターン別ログ・会話サマリー | チケット全履歴の確認 |
| WebOps 承認パネル | ドライランスクリーンショット表示・「確定実行」ボタン | WebOps Agent 専用 |
| 監査ログ | actor / event_type / ai_model フィルタ・CSV エクスポート | コンプライアンス対応 |
| CMDB 台帳 | デバイス・ユーザー・SaaS サービス検索・編集 | Read-only（変更は承認ゲート経由）|

### 2-3. AIオーケストレーション（多段エージェント構成）

**オーケストレーター（入口の司令塔）**
- Zooba/Slackからのイベントを受信し、ITIL 4規律に基づいて依頼を分類・ルーティング
- Read操作は自律実行。Write操作は `ActionProposal` として生成し、承認ゲートへ送付
- ITIL規律違反・セキュリティリスク依頼は `reject_request` で即座に却下（監査役AI）

**技術エージェント群**
| エージェント | 責務 |
|---|---|
| InfraOps | ネットワーク障害・端末不調・接続問題の切り分け診断 |
| SystemOps | SaaSアプリケーションの設定確認・運用判断 |
| IAM Agent | Google Workspace / M365 アカウント作成・削除・権限変更 |
| Google Workspace Agent | Google Admin SDK 経由の詳細操作 |
| M365 Agent | Microsoft Graph API 経由の詳細操作 |

**プロセス管理エージェント群**
| エージェント | 責務 |
|---|---|
| Inc/Prob | トラブル止血（一時対応）→ RCA → 恒久対策 → ナレッジ化 |
| Change/Release | 変更影響範囲調査・手順作成・切り戻し・事後チェック管理 |
| Project推進 | 新SaaS導入・全社移行の要件整理・タスク分解・進捗管理 |

**バックオフィス拡張エージェント群**
| エージェント | 責務 |
|---|---|
| AP/Billing Ops | メール/フォルダから請求書受領 → OCR抽出（金額・ベンダー・期限）→ CMDB照合 → 担当者自動振り分け |
| Procurement/VendorOps | KDDI等への発注要件定義・社内承認・発注構造化メール生成・進捗追跡・納品後台帳更新 |
| FinOps-lite | IT支出ダッシュボード・ベンダー/契約/実績/配賦/予算管理・未使用ライセンス検知 |
| **WebOps Agent** | **APIが存在しないレガシー管理画面・SaaSポータルをブラウザ自動化で操作（詳細は 2-3-b 参照）** |

### 2-3-b. WebOps Agent（ブラウザ自動化エージェント）

APIが存在しないレガシーシステム・ベンダーポータル・ネットワーク機器の Web GUI を操作するための「最後の手段（Last Resort）」エージェント。他エージェントが API 経由で完結できない場合にのみ呼び出す。

#### 技術スタック

| コンポーネント | 採用技術 |
|---|---|
| 操作エンジン | Playwright（ヘッドレスブラウザ）|
| 推論・視覚エンジン | Claude claude-sonnet-4-6（Computer Use API）または GPT-4o Vision |
| セッション管理 | Cookie / storage-state の暗号化保存（Fernet + Key Vault）|

AIにブラウザのスクリーンショットおよびDOMを渡し、「どこをクリックするか・何を入力するか」を判断させる。

#### 想定ユースケース

| ユースケース | 概要 |
|---|---|
| レガシーSaaSの設定 | APIが提供されていない古い勤怠管理システムへのユーザー追加 |
| ベンダーポータルの操作 | KDDI・大塚商会などの発注ポータルで API がない画面での機器購入手続き |
| ネットワーク機器の Web GUI | CLI/API が塞がれているルーター・ファイアウォールの管理画面へのログインと設定変更 |

#### リスクと制約（Candor）

| リスク | 内容 |
|---|---|
| Brittleness（即死リスク） | SaaS 側が UI を変更するだけで自動化が無予告で停止する。API と異なりバージョン管理の概念がない |
| MFA / CAPTCHA の壁 | 管理画面ログインに要求される多要素認証・CAPTCHA は AI 単独での突破が困難（かつ規約違反リスクあり） |
| Blast Radius（誤操作の影響） | 「無効化」「削除」ボタンの誤クリックがデータロストに直結する。API のようなパラメータ検証が存在しない |

#### Guardrails（安全組み込み要件）

**① ドライランスクリーンショットによる HITL 承認**
- 「保存」「確定」ボタンを押す直前で処理を一時停止
- 入力済み画面のスクリーンショットを Agent Console へ送信
- Growtect NOC 担当者が目視確認したうえで「確定」指示を出すフローを必須とする

**② Session Injection（人間によるセッション渡し）**
- MFA 突破のため、Growtect 運用担当者が一度ブラウザでログインし、その Session Cookie を暗号化して WebOps Agent に渡す仕組みを構築
- Cookie は操作完了後に即時破棄。保存期間はタスク実行中のみ
- **CAPTCHA への対応方針（規約遵守）**：CAPTCHA バイパスツールの使用は原則禁止。以下の代替手順を採用する：
  1. 担当者が手動でログインし、CAPTCHA 突破後の Cookie を Sesssion Injection で渡す
  2. ベンダーに自動化用の専用アカウント（CAPTCHA 免除 IP ホワイトリスト or API 発行）を交渉する
  3. 上記が不可の場合は WebOps Agent の対象外とし、手動運用を継続。無理な自動化は行わない

**③ Read 優先の原則**
- Write（設定変更）操作は最高リスク扱い。まず Read（スクレイピングによる CMDB 同期）から適用し、Write は Growtect 内部での十分な実績蓄積後に解禁する

### 2-4. 台帳レイヤ（Single Source of Truth）

全エージェントの判断根拠として機能する3つの台帳：

| 台帳 | 管理対象 |
|---|---|
| CMDB（構成管理） | サービス・システム・デバイス・依存関係・オーナー・契約更新日 |
| IAM（権限・アカウント管理） | 従業員・ロール・付与ライセンス数・例外承認ルール |
| Version（リリース台帳） | 環境別バージョン・リリース履歴・切り戻しポイント |

### 2-5. 外部システム連携一覧

| カテゴリ | システム | 連携方式 |
|---|---|---|
| 対話フロント | Zooba | Webhook → Pulse API |
| ITSM/台帳 | LMIS（ユニリタ） | REST API（Salesforce基盤） |
| 調達・物流 | KDDIまとめてオフィス | REST API（発注・追跡） |
| エンドポイント管理 | DRESS CODE | API（ゼロタッチデプロイ・プロファイル適用） |
| エンドポイント管理 | Microsoft Intune | Microsoft Graph API |
| エンドポイント管理 | Google Endpoint | Google Admin SDK |
| IDaaS | Google Workspace | Google Admin SDK / Directory API |
| IDaaS | Microsoft 365 | Microsoft Graph API |

**Read/Write分離の原則**：設定確認はRead権限で自律実行。Write（変更・作成・削除）は承認ゲート通過後のみ実行。

---

## 3. 非機能要件

### 3-1. AI推論エンジン（マルチLLM構成）

| ロール | プロバイダー | 用途 |
|---|---|---|
| Primary | Azure OpenAI（GPT-4o系） | 主推論エンジン。国内リージョン。エンタープライズSLA。 |
| Secondary | AWS Bedrock（Claude claude-opus-4-6） | フォールバック。AzureダウンまたはClaude特性が必要な処理。 |

- プロバイダー障害時は自動フェイルオーバー（`tenacity` リトライ + 切り替えロジック）
- モデルインターフェースを抽象化し、呼び出し側がプロバイダーを意識しない設計
- **入力データはAIの学習に使用されないことを技術的・契約的に保証**（Azure / Bedrock のエンタープライズ契約）

### 3-2. データ保護

- **PIIマスキング**：個人情報（氏名・メールアドレス・電話番号）はAIへ渡す前にマスキング処理
- **データ国内保持**：全データを国内リージョン（Azure Japan East / AWS ap-northeast-1）に限定
- **APIトークン暗号化**：SaaSのAPIトークンは `cryptography.Fernet`（AES-128-CBC）で暗号化保存。本番環境ではAzure Key Vault / AWS KMS（HSM相当）で鍵管理
- **テナント別シークレット分離（Tenant-isolated Secret Management）**：顧客AのAPIキーと顧客BのAPIキーを同一の暗号化コンテキストで扱ってはならない。Key Vault / KMS 上でシークレットのパスをテナントIDで厳密に分離する（例：`secrets/{tenant_id}/google_workspace_api_key`）。テナント単位の鍵ローテーションも独立して実施可能な設計とする

### 3-2-b. マルチテナントデータ分離：PostgreSQL RLS（行レベルセキュリティ）

アプリケーション側の `WHERE tenant_id = ?` クエリだけに頼るデータ分離は、MSPモデルにおいて重大インシデント（他顧客のチケット・CMDB情報の漏洩）のリスクを残す。**DBエンジンレベルで物理的に分離を強制する**ことを必須要件とする。

**実装方針：**

```sql
-- 全テナント分離対象テーブルに RLS を有効化
ALTER TABLE tickets ENABLE ROW LEVEL SECURITY;
ALTER TABLE approvals ENABLE ROW LEVEL SECURITY;
ALTER TABLE audit_logs ENABLE ROW LEVEL SECURITY;
ALTER TABLE cmdb_devices ENABLE ROW LEVEL SECURITY;
ALTER TABLE cmdb_users ENABLE ROW LEVEL SECURITY;
ALTER TABLE cmdb_services ENABLE ROW LEVEL SECURITY;
ALTER TABLE knowledge_chunks ENABLE ROW LEVEL SECURITY;

-- ポリシー定義（例：tickets テーブル）
CREATE POLICY tenant_isolation ON tickets
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);
```

- FastAPI リクエスト処理の冒頭で `SET LOCAL app.current_tenant_id = '{tenant_id}'` を発行し、セッションコンテキストをセット
- 開発者がクエリで `WHERE tenant_id` を書き忘れても、DBが自動的に他テナントのデータへのアクセスをブロック
- アプリケーション用 DB ユーザーには `BYPASSRLS` 権限を付与しない（スーパーユーザーのみバイパス可）

**パフォーマンス検証要件（RLS 特有の注意点）：**
- RLS は高スループット環境でポリシー評価のオーバーヘッドが発生するため、Phase 1 完了後に必ず負荷試験を実施する
- `tenant_id` カラムに B-tree インデックスを必ず付与し、ポリシー評価コストを最小化する
- 結合を多用するクエリは `EXPLAIN ANALYZE` で RLS の影響を確認し、必要に応じて部分インデックスを検討する
- 許容基準：同時 100 リクエスト下で p95 レイテンシが RLS なし時比 1.5 倍以内

### 3-3. 認証・アクセス制御

- **MFA + SSO**：全アクセスにMFAを必須。SAML/OIDC連携でシングルサインオン
- **Just-In-Time（JIT）アクセス制御**：AIがAPI実行する瞬間だけ必要なスコープを動的発行し、作業後に剥奪（ゼロトラスト）
- **承認権限階層**：`none < it_manager < department_head < ciso`

### 3-4. 可用性・整合性

- **マルチLLMフォールバック**：Primary/Secondaryの自動切り替えで業務継続
- **ドリフト検知**：DRESS CODE / Intune から1時間ごとに自動ポーリングし、台帳と物理デバイスの差分を検知して修正提案を生成
- **Celery非同期処理**：Slackの3秒タイムアウト制約をCeleryタスクキューで回避

---

## 4. 運用・コンプライアンス要件

### 4-1. Human-in-the-Loop（HITL）の徹底

- AIによるWrite操作（API実行・設定変更・アカウント操作）は必ず `PENDING_APPROVAL` を経由
- Slackのインタラクティブメッセージ（Block Kit）で承認/却下ボタンを提示
- 承認者は権限レベルを事前登録。権限不足者の承認は拒否
- 承認期限：24時間。期限切れはエスカレーション

**HITL フェーズ別自動化ロードマップ（長期負担軽減）：**

| フェーズ | HITL 適用範囲 | 自動承認の条件 |
|---|---|---|
| Phase 1–2 | 全 Write 操作を HITL 必須 | なし（人間が全件承認） |
| Phase 3 | 低リスク定型操作の自動承認解禁 | LLM Evals スコア ≥ 95%・過去 30 日間のエラー率 0% 実績を積んだアクション種別のみ |
| Phase 4 以降 | 中リスク操作の自動承認拡大 | CISO の書面承認 + 月次レビューによる範囲更新 |

- 自動承認に移行した操作も audit_logs への完全記録は維持する
- 誤操作・インシデント発生時は即時 HITL 必須に戻す「フォールバック規則」を設ける

### 4-2. Policy-as-Code

- 業務ルール・法制度（インボイス制度・電帳法等）をYAMLコードとして管理
- 法改正時は基盤コードを修正するだけで全エージェントの判断基準が即時更新

**YAML ポリシー定義例：**

```yaml
# policies/invoice_act_2023.yaml  （インボイス制度対応）
policy_id: invoice_act_2023
description: "適格請求書等保存方式（2023年10月施行）"
rules:
  - id: require_registration_number
    trigger: ap_billing_ops.invoice_received
    condition: vendor.registration_number is null
    action: reject
    message: "適格請求書発行事業者登録番号が未確認です。手動確認を依頼します。"
  - id: tax_rate_validation
    trigger: ap_billing_ops.invoice_received
    condition: invoice.tax_rate not in [0.08, 0.10]
    action: flag_for_review
    message: "税率が標準値（8%/10%）と異なります。"

# policies/electronic_bookkeeping_act.yaml  （電帳法対応）
policy_id: electronic_bookkeeping_act
description: "電子帳簿保存法（2024年1月義務化）"
rules:
  - id: require_timestamp
    trigger: ap_billing_ops.invoice_stored
    condition: invoice.received_at is null
    action: reject
    message: "受領日時の記録が必須です（電帳法 第4条）。"
  - id: immutable_storage
    trigger: ap_billing_ops.invoice_stored
    action: enforce_immutable
    message: "電子取引データは改ざん防止措置が必要です。S3 Object Lock / Azure Immutable Blob に保存。"
```

- ポリシー YAML は Git 管理し、Pull Request レビューを経て本番反映
- 法改正時の影響範囲は `policy_id` で追跡可能。法務チームが YAML を直接レビューする運用を想定

### 4-3. 監査ログの完全性

- 「誰が・いつ・何を見て・何を承認・実行したか」を `audit_logs` テーブルに全記録
- AI分析ターン単位でモデル名・入出力トークン数・推論サマリーを保存
- ログの改ざん不可設計（追記のみ）

### 4-4. コスト提示（Cost-Aware Logic）

- AIが提案する操作には、それによるSaaSコスト・API利用料のランニングコスト増減を必ず併記

### 4-5. LLM Evals（エージェント品質評価）

Reception Agent・Knowledge Agentは自社実装であるため、「回答が正しいか（ハルシネーションしていないか）」を自動テストする仕組みを整備する。Phase 1 には含めないが、Phase 3 以降の運用要件として以下を計画する。

| 評価観点 | 指標 | ツール |
|---|---|---|
| RAG回答の忠実性 | Faithfulness（検索結果に基づいているか） | Ragas |
| 検索精度 | Context Precision / Recall | Ragas |
| 意図分類の正確性 | 分類カテゴリの混同行列 | pytest + LLM-as-Judge |
| ハルシネーション検出 | Answer Relevancy | Ragas / DeepEval |

**テストデータの整備方針：**
- Phase 1〜2 で蓄積した実際のチケット（匿名化済み）をゴールデンデータセットとして使用
- Growtect エンジニアが「正解」にラベリングしたデータをテストケースとして管理
- CI/CD パイプラインに組み込み、モデル更新やプロンプト変更時に自動で評価スコアを算出

---

## 5. テクノロジースタック

| 層 | 技術 |
|---|---|
| AI推論（Primary） | Azure OpenAI（GPT-4o-mini / GPT-4o） |
| AI推論（Secondary） | AWS Bedrock（Claude claude-opus-4-6） |
| バックエンド | Python 3.12 + FastAPI + Celery |
| DB | PostgreSQL 15（Supabase互換）+ pgvector |
| メッセージキュー | Redis |
| Slack連携 | Slack Bolt for Python |
| フロントエンド（Agent Console） | Next.js 15 + shadcn/ui |
| インフラ（開発） | Docker Compose |
| インフラ（本番） | Azure Container Apps / AWS ECS |
| 暗号化 | cryptography（Fernet）+ Azure Key Vault / AWS KMS |
| ログ | structlog（JSON形式） |
| マイグレーション | Alembic |
| ブラウザ自動化（WebOps Agent） | Playwright + Claude Computer Use API / GPT-4o Vision |

---

## 6. DBスキーマ（主要テーブル）

```
tickets               チケット（依頼レコード）正本
  id, subject, body, status, priority
  slack_channel_id, slack_thread_ts
  requester_email, requester_user_id
  ai_analysis (JSONB)

approvals             Human-in-the-loop承認レコード
  id, ticket_id, approver_user_id
  action_proposal (JSONB)  ← AIが生成したWrite操作提案
  status, approver_comment
  slack_message_ts, expires_at

audit_logs            全操作の監査ログ（追記のみ）
  id, ticket_id, actor_type (ai/human/system), actor_id
  event_type, event_data (JSONB)
  ai_reasoning, ai_model_version

cmdb_devices          デバイス台帳
  id, hostname, serial_number, asset_tag
  device_type, os_type, os_version
  status, assigned_user_id
  intune_device_id, metadata (JSONB)

cmdb_users            ユーザーアカウント台帳
  id, email, full_name, employee_id
  department, slack_user_id, google_user_id
  approval_authority (none/it_manager/department_head/ciso)

cmdb_services         SaaSサービス台帳
  id, name, vendor, license_count, license_used
  api_credentials_encrypted, api_base_url
  contract_url, service_config (JSONB)

cmdb_user_service_accounts   ユーザー×サービス紐付け
  id, user_id, service_id, role_in_service, is_active
```

---

## 7. 実装フェーズ計画

### Phase 1 — Pulse Desk MVP（ヘルプデスク中核）

**目標**：Slackから依頼を受け、AIが分析・提案し、人間が承認してサブエージェントが実行する最小ループ

| ステップ | 実装内容 | 確認方法 |
|---|---|---|
| Step 1 | Docker環境・FastAPI起動・PostgreSQL・Redis | `GET /health` が200 |
| Step 2 | CMDB台帳スキーマ + Alembic + REST API | デバイス登録・取得API動作 |
| Step 3 | Slack Bolt受付 + チケット作成 | @PulseBot メンション → DBにチケット生成 |
| Step 4 | Orchestratorエージェント + Celery非同期 | AI分析 → 監査ログ記録 |
| Step 5 | 承認フロー（Block Kit）+ 権限チェック | 承認/却下ボタン → 状態遷移 |
| Step 6 | IAM / InfraOps サブエージェント + Google Workspace SDK | 「アカウント作成」依頼 → 承認 → Google Admin API実行 |

### Phase 2 — バックオフィス自動化

- AP/Billing Ops：メール受信 → OCR（Claude Vision）→ CMDB照合 → 振り分け
- Procurement/VendorOps：発注フロー → KDDI API連携
- LMIS連携：詳細インシデント管理・CMDB同期

### Phase 3 — ガバナンス・フルスタック

- Agent Console（Next.js）：承認キュー・タイムライン・監査ログUI
- Policy-as-Code（YAML規則エンジン）
- FinOps-lite：IT支出ダッシュボード・ライセンス最適化
- DRESS CODE連携：ドリフト検知・ゼロタッチデプロイ
- マルチLLMフォールバック完全実装

### Phase 4 — スケールアウト・自律化拡張

**トラフィック増大への対応**
- Azure Container Apps / AWS ECS のオートスケーリング設定（CPU 70% トリガー）
- Celery ワーカーの水平スケール（テナント数増加に伴うキュー分割設計）
- pgvector インデックスを HNSW へ移行し、RAG 検索レイテンシを維持

**エージェント拡張**
- WebOps Agent の Write 操作を HITL 実績に基づき段階的に自動承認解禁
- 新規 SaaS 連携の追加を標準化：「エージェント追加テンプレート（FastAPI + Celery + SDK）」を整備し、1 エージェントあたり 2 週間以内での追加を目標とする

**マルチリージョン対応（必要に応じて）**
- 顧客データ主権要件（GDPR等）に対応するため、リージョン別テナント分離設計を検討
- 現時点では国内リージョンのみだが、海外展開時のアーキテクチャ変更を最小化する設計を Phase 3 で仕込む

---

## 8. 重要設計決定（ADR）

| 決定事項 | 選択 | 理由 |
|---|---|---|
| チケット正本 | Pulse自前DB | LMIS依存リスク回避。Pulseがデータ主権を持つ |
| AI推論 | マルチLLM（Azure + Bedrock） | 単一障害点排除・コンプライアンス（データ非学習保証） |
| 対話フロント | Zooba連携 | Zoobaが一次回答をさばき、Pulseは複雑処理に特化 |
| Write実行 | HITL必須（承認ゲート） | ハルシネーションによる誤操作・誤発注を防止 |
| 非同期処理 | Celery + Redis | Slackの3秒タイムアウト制約の回避 |
| 暗号化 | Fernet + Key Vault/KMS | SaaS APIトークンの平文保存を禁止 |

---

## 9. 検証方法（テスト戦略）

### 9-1. E2E テスト（手動）

ステージング環境で担当者が手動実行。CI/CD には組み込まない（Slack / 外部 API の副作用があるため）。

1. **Slackメンション受付** → チケットが `RECEIVED` でDB保存される
2. **Celeryタスク実行** → `ANALYZING` に遷移 → Azure OpenAI APIが呼ばれる → 監査ログに記録
3. **ActionProposal生成** → `PENDING_APPROVAL` に遷移 → SlackにBlock Kitメッセージが届く
4. **承認ボタン押下** → 権限チェックOK → `APPROVED` → 実行タスクキューイング
5. **実行完了** → `COMPLETED` → Slackスレッドに完了通知
6. **却下ボタン** → 否認理由モーダル → `DENIED` → チケットクローズ
7. **ITIL違反依頼** → `reject_request` → Slackに却下理由を返信

### 9-2. 自動テスト（CI/CD に組み込む）

PR マージ時に GitHub Actions / Azure DevOps Pipelines で自動実行。

| テスト種別 | ツール | 対象 | 自動化 |
|---|---|---|---|
| ユニットテスト | pytest | エージェントロジック・ポリシー評価・RLS ポリシー SQL | ✅ 必須 |
| 統合テスト | pytest + testcontainers | FastAPI ↔ PostgreSQL ↔ Redis（Docker Compose） | ✅ 必須 |
| LLM Evals | Ragas / DeepEval | RAG 忠実性・ハルシネーション検出 | ✅ Phase 3 以降 |
| 負荷テスト | Locust | 同時 100 リクエスト下の RLS レイテンシ | ✅ Phase 2 完了後 |
| セキュリティスキャン | Bandit（Python）+ Trivy（コンテナ） | 既知脆弱性・シークレットの平文混入検出 | ✅ 必須 |
| WebOps 動作確認 | Playwright テストスクリプト | ドライランスクリーンショット生成・Session Injection | 手動（副作用あり）|

### 9-3. ロールバック基準

- 本番デプロイ後 15 分以内にエラー率 > 1% を検知した場合、前バージョンへ自動ロールバック
- DB マイグレーション（Alembic）は必ず `downgrade` スクリプトを作成してからマージを許可

---

## 11. ドキュメント出力手順

### MDファイル
```
/home/user/Factory/docs/growtect-pulse-requirements-v1.md
```
上記パスにコピー（Factory配下で版管理）。

### PDFファイル
pandocが未インストールのため、Python（markdown + weasyprint）で生成：
```
/home/user/Factory/docs/growtect-pulse-requirements-v1.pdf
```

---

## 10. 未確定・今後詰める事項

Phase 1 MVP の開発開始前に **P1（必須）** を解消することを必須とする。

| 優先度 | 事項 | アクション | 期限 |
|---|---|---|---|
| P1 | Zooba → Pulse 間の Webhook 仕様詳細 | Zooba 社に API ドキュメント・サンドボックス環境を要求 | Phase 1 開始前 |
| P1 | LMIS REST API のエンドポイント仕様 | ユニリタ担当者と技術 MTG を設定 | Phase 1 開始前 |
| P2 | DRESS CODE の API 提供有無・仕様 | ベンダー確認（API 未提供なら WebOps Agent で代替） | Phase 2 開始前 |
| P2 | KDDI まとめてオフィスの API 仕様・契約要件 | KDDI 担当者に発注 API の有無を確認。未提供なら Procurement/VendorOps をメール送信型に変更 | Phase 2 開始前 |
| P3 | 本番インフラ：Azure Container Apps vs AWS ECS 最終選定 | コスト試算・SLA 比較を行い最終決定 | Phase 3 開始前 |

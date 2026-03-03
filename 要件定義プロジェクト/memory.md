# Growtect Pulse — セッション横断メモリ

> このファイルはセッションをまたいで引き継ぐべき重要な決定事項・背景情報を記録する。
> 更新日：2026-03-03

---

## プロダクト概要

**Growtect Pulse** — 自律型コーポレートOS（Autonomous Corporate OS）
日本の中小企業の情シス属人化問題を解決し、AIとシステムが管理負荷を担うことで人間が本来業務に集中できる環境を実現する。

- **リポジトリ**：`yozeki-Growtect/Factory`
- **開発ブランチ**：`claude/growtect-pulse-design-A2T8n`
- **要件定義書（技術仕様）**：`docs/growtect-pulse-requirements-v1.md`
- **PM要件定義書**：`要件定義プロジェクト/outputs/requirements-pm-v1.md`

---

## 確定済み設計決定

### アーキテクチャ

| 決定事項 | 決定内容 | 理由 |
|---|---|---|
| チケット正本 | Pulse 自前 DB（PostgreSQL） | LMIS 依存リスク回避。データ主権をPulseが持つ |
| 対話フロント | Pulse Bot（自前実装・Slack Bolt） | Zooba を2025年に要件から完全削除。外部SaaS依存を排除 |
| AI推論 | マルチLLM（Azure OpenAI Primary + AWS Bedrock Secondary） | 単一障害点排除・データ非学習保証 |
| Write実行 | HITL必須（承認ゲート） | ハルシネーションによる誤操作防止 |
| 非同期処理 | Celery + Redis | Slack 3秒タイムアウト制約の回避 |
| 暗号化 | Fernet + Key Vault / KMS | SaaS APIトークンの平文保存禁止 |

### エージェント責務の境界

- **IAM Agent**：判断層（意図解釈・承認判断・ルーティング）
- **Google Workspace Agent / M365 Agent**：実行層（SDK/API経由の実際の操作）
- IAM Agent が直接 SDK を呼ぶことはない
- **Policy Evaluation Agent**：全エージェントから呼ばれるポリシーゲートウェイ

### セキュリティ

- **JIT アクセス制御**：動的スコープ発行は Google/M365 の OAuth 制約で実現困難なため、「操作完了後に即時 revoke、次回実行時に再取得」方式を採用
- **PostgreSQL RLS**：マルチテナントの物理分離をDBエンジンレベルで強制
- **テナント別シークレット分離**：`secrets/{tenant_id}/...` のパス構造で Key Vault/KMS 上で分離

### SLA / RTO / RPO

| 指標 | Phase 1〜2 | Phase 3 以降 |
|---|---|---|
| 稼働率 | 99.5% | 99.9% |
| RTO | 1時間以内 | 30分以内 |
| RPO | 1時間以内 | 15分以内 |

### Policy-as-Code

- YAML = 人間可読。法務・コンプライアンス担当者がレビュー
- 評価・実行 = Policy Evaluation Agent が担当（OPA/Rego は不採用）
- Git PR レビューを経て本番反映。マージで自動リロード

### Agent Console UI

- **リスクバッジ**：🟢低 / 🟡中 / 🔴高 の3段階色分け＋影響ユーザー数・対象サービスをカード表面に表示
- **承認チャネルの使い分け**：
  - 低リスク定型操作 → Slack Block Kit
  - 中〜高リスク・WebOps操作 → Slack に DeepLink 通知 → Agent Console で承認
- **ダッシュボード**：ログインロール（it_manager / department_head / ciso）別にパーソナライズ

### HITL 自動化ロードマップ

| フェーズ | 自動承認の条件 |
|---|---|
| Phase 1〜2 | なし（全件 HITL） |
| Phase 3 | LLM Evals スコア ≥ 95% かつ過去30日エラー率 0% のアクション種別のみ |
| Phase 4 | CISO 書面承認 + 月次レビューで範囲更新 |

### 承認エスカレーション

`it_manager` → `department_head` → `ciso` → Growtect NOC（手動対応）

---

## 未解決事項（要追跡）

| 優先度 | 事項 | 期限 |
|---|---|---|
| P1 | LMIS REST API エンドポイント仕様（ユニリタと技術MTG） | Phase 1 開始前 |
| P2 | DRESS CODE API 提供有無 | Phase 2 開始前 |
| P2 | KDDI まとめてオフィス API 仕様 | Phase 2 開始前 |
| P3 | 本番インフラ選定（Azure Container Apps vs AWS ECS） | Phase 3 開始前 |
| P2 | テナントオンボーディングフローの設計 | Phase 2 開始前 |
| P2 | データ保持ポリシーの決定（監査ログ・チケット保存年数） | Phase 2 開始前 |

---

## 事業現況（2026-03-03 更新）

### 会社・収益
- **法人名**：Growtect（稼働中）
- **現在の売上**：NTT東日本向けコンサルティング 月額105万円（0.8人月）
  - 内容：アウトソーシングサービスのコンセプト・価格設計支援
  - 追加コンサル1名採用予定 → 月額200万円規模へ拡大予定
- **Dual Engine Strategy**：Engine 1（コンサル Cash Cow）が既に稼働中

### チーム
- **Founder / CEO**：小関陽介（ITIL Expert × JBS6→10億デバイス事業責任者 × Sansan情シス改革）
- **CTO候補**：東京工業大学主席 × アクセンチュア出身エンジニア（Join交渉中）

### プロダクト現状
- **ステージ**：コンセプト段階
- **PoC顧客**：未確定（これから探す）
- NTT東日本は現時点でPoC候補ではないが、将来の戦略的PoC先として位置付け

### 資金調達方針
- 現在：自己資本 + コンサル収益で運営（外部投資家なし）
- 方針：「CTO Join確定 × PoC顧客確定」後にSeed調達（¥50M〜¥150M想定）

---

## 事業戦略のキーナラティブ（2026-03-03 確定）

### Why Now（4つの特異点）
1. AIの実行能力が臨界点を超えた（2024〜2025年）
2. Agentic Sprawlの痛みが「今期の予算課題」になった
3. 日本の労働崖が現場の実害として顕在化（2025〜2026年）
4. コーポレートナレッジは時間と比例して深くなる（先行者がデータの堀を掘る）

### Why Me（小関陽介の必然性）
- L4（物理現場）→ L3（自動化）→ L2（事業責任）→ L1（戦略）の順でキャリアを積んだ
- 「壊れた運用を仕組みに変える」という問いを20年繰り返してきた
- 思考と物理の両方を現場・事業・経営の全レイヤーで経験した人間にしか、この断絶は見えない

### Moat（3つの堀）
1. ITIL-Native AI（ガバナンスをコード化したAI）
2. Physical Infrastructure Integration（KDDI物流網との独占的API連携）
3. Cyber-Physical Data Loop（コーポレートナレッジの複利蓄積）

---

## コミット履歴（主要）

| コミットハッシュ | 内容 |
|---|---|
| `5adb8cc` | Zooba を要件定義書から完全削除 |
| `731ae0c` | 構造的修正4件（IAM境界・エスカレーション先・セクション順序・タイポ） |
| `1d72d22` | 設計判断②③・UI設計を反映（SLA/JIT/Policy-as-Code/Agent Console UI） |
| `8da4167` | 要件定義プロジェクト構造・memory.md・競合分析・PM要件定義書を新設 |
| `5eba512` | VCレビュー結果（vc-review-v1.md）・事業戦略書（business-strategy-v1.md）を追加 |
| `797dd2d` | チームセクション（小関陽介の職務経歴）を追記 |
| `51bb7ce` | Why Now / Why Me セクションを追加 |
| `30c2902` | CTO候補プロフィールを追加 |

---

## 競合・市場ポジショニング

詳細は `要件定義プロジェクト/inputs/competitive-analysis.md` を参照。

**要点**：国内中小企業向けの「IT管理を丸ごと自動化するMSPプラットフォーム」というポジションに直接競合する国産プロダクトは2026年3月時点で確認されていない。
ServiceNow は大企業向け・高コストで中小企業のアクセシビリティが低い。

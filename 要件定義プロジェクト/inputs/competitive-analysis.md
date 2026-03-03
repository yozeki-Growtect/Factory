# 競合・市場分析

> 調査日：2026-03-03

---

## 市場の現状

「AIエージェントによるITヘルプデスク・コーポレートIT管理の自動化」は2025〜2026年にかけて急速に注目が高まっている領域。AIエージェント市場は2024年の50億ドルから2025年末には130億ドル規模へ成長が予測されており、エンタープライズ向けでは ServiceNow が業界標準の地位を固めつつある。

ただし**日本の中小企業（従業員数50〜500名規模）向け**の「IT管理を丸ごと自律実行するMSPプラットフォーム」というポジションには、2026年3月時点で直接競合するプロダクトは見当たらない。

---

## 競合マップ

### Tier 1：直接競合（同じ課題を解こうとしている）

#### ServiceNow AI Platform
- **概要**：エンタープライズITSMの最大手。2025年に「ServiceNow AI Platform」へブランド転換。AI Control Tower・AI Agent Fabric を発表。2025年12月に会話型AIプラットフォーム Moveworks を買収。
- **強み**：業界実績・統合範囲の広さ・エンタープライズSLA・グローバルサポート
- **弱み**：導入コストが高額（年間数千万〜数億円規模）・実装に専門SIが必要・中小企業にはオーバースペック・日本語対応が不完全・カスタマイズに時間がかかる
- **Growtect Pulse との差別化**：ServiceNow は「ツールを提供しIT部門が使う」モデル。Growtect Pulse は「ITを丸ごと運用するMSP」モデル。Growtect 自身がオペレーターとなり顧客の情シスを代替する。

#### Freshservice（Freshworks）
- **概要**：中堅市場向けITSM。AI機能として Freddy AI を内蔵。月額課金で ServiceNow より安価。
- **強み**：導入が容易・コスト低め・Freddy AI による自動化
- **弱み**：自動「実行」ではなく自動「提案・分類」止まり。日本市場での実績が薄い。CMDB・台帳管理が弱い。
- **Growtect Pulse との差別化**：Freshservice は従来型チケット管理の延長線上。Pulse はエージェントが実際に Google Workspace / M365 / KDDI などを操作して完結する。

#### Jira Service Management（Atlassian）
- **概要**：開発者寄りのITSM。Rovo AI エージェントを2025年に発表。Confluence/Jira との統合が強み。
- **強み**：開発組織との親和性・既存Atlassian資産の活用
- **弱み**：非エンジニアには難易度が高い・バックオフィス系（発注・請求書処理等）は守備範囲外
- **Growtect Pulse との差別化**：Pulse のターゲットは「情シス担当がいない中小企業」。Jira は「IT部門がある企業」向け。

### Tier 2：隣接競合（一部機能が重なる）

#### Moveworks（ServiceNow に買収済み）
- IT向けの会話型AIプラットフォーム。SlackやTeamsでの自然言語インタラクションに強み。
- ただし ServiceNow に統合され中小企業向けの独立プロダクトとしては実質消滅方向。

#### Glean
- エンタープライズ検索＋AIエージェント。社内ナレッジ検索に強い。
- IT操作の実行には対応しておらず、「調べる」機能に特化。

#### Microsoft 365 Copilot + Azure OpenAI
- Microsoft エコシステム上での自動化。Power Automate との統合で一部の業務自動化は可能。
- ただし「中小企業が使いこなせる形」での提供にはギャップが大きい。設定コストが高い。

#### Google Workspace + Gemini
- Gemini for Workspace として AI 統合が進む。
- Power Automate 同様、「設定できる技術者がいる前提」でのツール。自律実行の概念はまだ薄い。

### Tier 3：国内プレイヤー（参考情報）

| プロダクト | カテゴリ | IT管理自動化との関連 |
|---|---|---|
| SmartHR | HRSaaS | 入退社フローは一部重なるが IT管理は対象外 |
| マネーフォワード クラウド | バックオフィスSaaS | 請求書処理は一部競合。IT管理は対象外 |
| 楽楽精算 / 楽楽販売 | 経費・受発注SaaS | バックオフィス拡張機能と一部重複。自律実行なし |
| KDDIまとめてオフィス | 中小企業向けITインフラ | Pulse の連携先・補完関係 |

**→ 日本国内に「AIエージェントが中小企業のITを丸ごと自律運用するMSP」という直接競合は2026年3月時点で確認されていない。**

---

## Growtect Pulse の差別化ポイント

### ① 日本市場ファースト
- 電子帳簿保存法・インボイス制度に対応した Policy-as-Code を標準装備
- 日本語での自然言語インタラクション（Slack / Teams）
- KDDI まとめてオフィスとの調達連携（日本固有の商流）

### ② MSP モデルによる完全代替
- SaaS ツールを「顧客が自分で使う」のではなく、**Growtect がオペレーターとして運用を代行**
- 情シス担当者ゼロでも IT インフラが回る状態を提供
- ServiceNow / Freshservice は「自社 IT 部門が使う道具」であり根本的にモデルが異なる

### ③ 自律「実行」
- 競合の多くは「提案・分類・回答」止まり
- Pulse は承認後に Google Workspace / M365 / KDDI API を**実際に呼んで完結**させる
- WebOps Agent により API のないレガシーシステムにも対応

### ④ 中小企業に現実的なコスト
- ServiceNow の 1/10〜1/20 の価格帯でのサービス提供を想定
- 初期導入コストを Growtect が吸収し、月額サブスクリプション型で提供

---

## 市場機会

- 日本の中小企業数：約350万社（うち情シス担当者がいない企業が大多数）
- 中小企業のITサポートコスト：月額10〜50万円の外部IT支援費が一般的
- AI導入による自動化可能な業務の割合：ヘルプデスク・アカウント管理・調達で60〜70%の自動化が実現可能との調査結果あり（Moveworks 等の試算）

---

## 参考情報

- [ServiceNow AI Agent（2025年Knowledge発表）](https://prtimes.jp/main/html/rd/p/000000158.000029239.html)
- [AI tools for ITSM 2026 guide](https://monday.com/blog/service/ai-tools-for-it-service-management/)
- [Top AI Agent Platforms for SMBs in 2025](https://thejourneyplatform.com/blog-posts/top-8-ai-agent-platforms-for-smbs-in-2025)
- [SaaS業界とAIエージェント 2025年動向](https://enterprisezine.jp/news/detail/23150)

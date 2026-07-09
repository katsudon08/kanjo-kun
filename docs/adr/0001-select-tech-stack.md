# 0001. 技術スタックの選定

## ステータス

承認（PR #4。初版の選定に対し koutyuke のレビュー指摘を反映して再確定）

## 文脈

勘定くんは、はしご飲みの立替を「誰が誰にいくら」の債務グラフとして可視化し、送金サービスへ遷移して精算まで導く Web アプリ。MVP は Web のみ（ネイティブは後続フェーズ）。実装着手の前提として、技術スタックを恒常リファレンスとして確定する必要があった。

初版（PR #4）で確定したスタックに対し、レビュアー koutyuke から FE 選定とモノレポ構成について 5 件の指摘があり、各項目をトレードオフを検討して再確定した。判断軸は次の 3 点:

- 立ち上げ期は最普及・シンプルな定番を選び、後で差し替える。
- 2 名体制でのチーム合意のしやすさ（レビュアーの意向）。
- 本アプリ（React Flow を用いたグラフ/キャンバス系 UI）への適合。

## 決定

### 全体構成

React SPA（TypeScript / Vite）＋ Go API（net/http + Connect）＋ PostgreSQL を分離型モノレポで構成する。FE↔BE 契約は Connect-RPC + buf（protobuf）を型の単一の源（SSOT）とする。

### モノレポ構成（指摘①）

`apps/{frontend,backend}` + `packages/proto` を採用する（root フラット構成から変更）。t3/Turborepo の慣習に沿い、将来の共有パッケージにも自然に拡張できる。`packages/proto` は当面 `.proto` + buf 設定のみを保持し、生成物は各アプリ配下（`apps/frontend/src/shared/gen`・`apps/backend/internal/gen`）へ出力するため、消費される TS パッケージではない。したがって **pnpm workspaces は当面導入しない**。

### フロントエンド

- 状態管理: **Jotai**（指摘②。Zustand から変更）。アトミックで、React Flow のグラフ/派生状態・粒度の細かい状態に適合する。FSD の各スライスの `model` セグメントに atom を co-locate する。
- フォーム: **TanStack Form**（指摘④。React Hook Form から変更）。TanStack Router + Query と合わせてエコシステムを統一し、型安全性を得る。
- バリデーション: **Zod v4**（指摘③。Valibot への変更提案があったが維持）。v4 で bundle 懸念が解消（Zod Mini 3.94kB）し、エコシステムと事例が最大。TanStack Form とは Standard Schema 経由で連携する。
- lint/format・ツールチェーン: **oxlint + oxfmt を vp（Vite+）経由で採用**（指摘⑤。Biome から変更）。Vite 採用済みの本プロジェクトと方向性が一致する。
- その他: React + TypeScript / Vite、TanStack Router + Query、Tailwind CSS + shadcn/ui、React Flow。設計は Feature-Sliced Design（FSD）。

### バックエンド

Go / net/http + Connect / GORM / goose / caarlos0/env + godotenv / PostgreSQL。

### 横断・ツール分担

- **Taskfile（go-task）**: リポ root の言語横断タスクの入口。FE タスクは vp へ委譲する。
- **vp（Vite+）**: `apps/frontend` 内に限定した FE ツールチェーンの入口（`vp dev`/`build`=Vite、`vp test`=Vitest、`vp lint`=oxlint、`vp fmt`=oxfmt）。runtime / package-manager 管理機能は使わない。
- **mise**: Go / Node のバージョン固定の単一ソース（各サブプロジェクトに配置）。
- **pnpm**: JS パッケージ管理。
- テスト: Vitest（vp 経由）+ Testing Library、Playwright（E2E）、Storybook、Go は testing + testify + testcontainers-go。
- CI/CD: GitHub Actions、Docker（Go マルチステージビルド）。Docker Compose でローカル PostgreSQL。

## 結果

- **良い点**: FE が TanStack（Router / Query / Form）＋ Vite 線（Vite / Vitest / oxlint / oxfmt を vp で統合）で一貫し、レビュアーの意向にも沿う。`apps/` + `packages/` により将来の拡張余地（共有パッケージ・workspaces 導入）が明確。
- **懸念 / リスク**: **vp（Vite+）と oxfmt は 2026 年央時点で beta（1.0 前）**。特にフォーマッタは全ファイルに触れるため安定性リスクがある。これは「最普及・成熟の定番」方針からは外れる選択であり、Vite 線への統一というメリットと引き換えの意図的なトレードオフとして受容する。
- **見直しトリガ**: **vp / oxfmt が 1.0 に到達した時点で正式運用可否を再評価する**。深刻な不安定が生じた場合は Biome へ戻す退避路を残す（設定資産が小さいうちなら低コストで差し替え可能）。
- **スコープ外**: デプロイ先ベンダ（FE ホスティング / BE 実行 / マネージド DB）は本 ADR の対象外。形のみ確定し、実デプロイ時に別 ADR で決定する。

## 関連

- [../architecture.md](../architecture.md) — 本決定に基づく技術スタック・アーキテクチャの定義
- [../directory-structure.md](../directory-structure.md) — `apps/` + `packages/` 構成の詳細

# ディレクトリ構成 — 勘定くん

物理配置のリファレンス。設計の意図・不変条件は [architecture.md](architecture.md) を参照。

## 全体ツリー

```
kanjo-kun/
├── apps/
│   ├── backend/        # Go API サーバ（自前の go.mod・Go の設定を同梱）
│   └── frontend/       # React SPA（Vite・FSD・JS/TS の設定を同梱）
├── packages/
│   └── proto/          # protobuf と buf 設定（FE↔BE 契約）
├── docs/               # ドキュメント
├── Taskfile.yml        # 言語横断タスクの入口（dev/build/test/lint/migrate）
├── docker-compose.yml  # ローカル PostgreSQL
├── .gitignore
├── CLAUDE.md
└── LICENSE
```

**設定ファイルは言語ごとに分離する。** FE と BE は言語が異なるため、言語固有の設定は各サブプロジェクトに置き、root には言語横断のもの（Taskfile・docker-compose）だけを置く。

- `apps/frontend/` は**単独の pnpm プロジェクト**。`packages/proto/` は当面 `.proto` と buf 設定のみを保持し、生成物は各アプリ配下（`apps/frontend/src/shared/gen`・`apps/backend/internal/gen`）へ出力するため、消費される TS パッケージではない。したがって pnpm workspaces はまだ導入しない（Go は pnpm の対象外）。`packages/` 配下を TS パッケージとして import する段階で workspaces を導入する。
- `apps/backend/` は自前の `go.mod` を持つ単一 Go モジュール。
- 横断コマンド（両言語の build/test/lint 等）は Taskfile に集約する（Turborepo/Nx は使わない）。FE のツールチェーンは `apps/frontend` 内の vp（Vite+）に集約し、Taskfile の FE タスクは vp へ委譲する。

## apps/backend/

```
apps/backend/
├── go.mod
├── mise.toml           # Go のバージョン
├── .golangci.yml       # golangci-lint 設定
├── cmd/kanjo/main.go   # 入口。依存を結線してサーバ起動
└── internal/           # 非公開パッケージ（ドメイン/アダプタ単位で分割）
    ├── kanjo/          # ドメイン型 + 店舗グラフ・債務グラフの算法（純関数）
    ├── postgres/       # GORM リポジトリ実装・トランザクション
    ├── connect/        # Connect handler（RPC の入口）
    ├── kotra/          # ことら送金アダプタ（phase-2）
    ├── paypay/         # PayPay 送金アダプタ（phase-2）
    ├── ocr/            # レシート AI 自動入力（phase-2）
    ├── mock/           # テスト用モック
    └── gen/            # buf 生成の Connect stub（編集しない）
```

- `internal/` 内は技術レイヤ（handlers/services/repositories）ではなく**ドメイン/アダプタ単位**で分割する。依存の向きとドメインの純粋性は [architecture.md](architecture.md) の「設計上の不変条件」に従う。
- `pkg/` は使わない。MVP は上記より浅く始めてよい（単一パッケージのファイル分割から始め、必要になったら分割）。

## apps/frontend/

FSD のレイヤ構成。import は下位方向のみ、同一レイヤのスライス間は public API 経由。

```
apps/frontend/
├── package.json
├── pnpm-lock.yaml
├── mise.toml           # Node のバージョン
├── tsconfig.json
├── vite.config.ts      # Vite 設定（vp が配下で使用）
├── .oxlintrc.json      # oxlint 設定（format は oxfmt / vp fmt）
├── .storybook/         # Storybook 設定
└── src/
    ├── app/            # プロバイダ（Query/Router）・グローバルスタイル・ルート結線
    ├── pages/          # 画面（ルートレベルの合成）
    ├── widgets/        # 大きな UI ブロック（店舗グラフ・債務グラフ = React Flow）
    ├── features/       # ユーザー操作（会計追加・支払い記録・メンバー編集）
    ├── entities/       # ドメインエンティティ（event/store/accounting/member/debt/settlement）
    └── shared/         # 業務非依存: ui / lib / api / config / gen（生成クライアント）
```

- 各スライス内は `ui` / `model` / `api` / `lib` の segment で分ける。状態（Jotai の atom）は `model`。
- Storybook の stories はコンポーネントに co-locate する（`*.stories.tsx`）。
- スライス粒度の詳細は別途定める。
- FE のツールチェーンは vp（Vite+）に集約する（`vp dev`/`build`=Vite、`vp test`=Vitest、`vp lint`=oxlint、`vp fmt`=oxfmt）。ランタイム/パッケージ管理は mise・pnpm が担い、vp のそれらの機能は使わない。

## packages/proto/

```
packages/proto/
├── buf.yaml            # buf のモジュール・lint 設定
├── buf.gen.yaml        # コード生成設定（Go / TS の出力先）
└── kanjo/v1/*.proto    # サービス・メッセージ定義（バージョン付き）
```

`buf generate` で Go stub（`apps/backend/internal/gen/`）と TS クライアント（`apps/frontend/src/shared/gen/`）を生成する。生成物は編集しない。buf は proto を入力に両言語へ生成する契約ツールなので、proto と同梱する。

## ドメイン・機能とディレクトリの対応

| ドメイン / 機能 | backend | frontend |
| --- | --- | --- |
| 店舗グラフ（はしご/店舗/ブランチ） | `internal/kanjo` | `entities/{event,store}` + `widgets` |
| 債務グラフ（会計/メンバー/債務） | `internal/kanjo` | `entities/{accounting,member,debt}` + `widgets` |
| 精算（部分支払い・モック送金） | `internal/kanjo` + `internal/postgres` | `features` + `entities/settlement` |
| 入力（認証なし・誰でも編集可） | `internal/connect` | `features` |

MVP 機能（グラフ構築・均等割り・矢印列挙・精算モック・認証なし）は上記で充足する。

## 拡張（phase-2）の差し込み位置

| 追加機能 | 位置 |
| --- | --- |
| 債務簡約（簡易/複雑） | `internal/kanjo` に算法を追加 |
| 実送金連携（ことら/PayPay） | `internal/kotra`・`internal/paypay` を実装 |
| 割り勘の比重 / 定数額 | `internal/kanjo` の割り勘をストラテジ差し替え |
| レシート AI 自動入力 | `internal/ocr` を実装 |
| 支払い源管理 | `internal/kanjo` にドメイン型を追加 |
| 合流（マージ）で DAG 化 | `internal/kanjo` のグラフ表現を一般化 |

# 勘定くん

はしご飲みで積み重なる「代表者の立替 → メンバーの精算」の債務関係を可視化し、ことら送金/PayPay 等へ直接遷移して精算を完結させる割り勘アプリ。

## サービス概要

代表者が立て替えた会計をレシート等から取り込み、「誰が誰にいくら」の債務グラフ（依存関係）として可視化する。はしご（複数店舗）で複雑化した債務を最小回数の送金へ簡約し、送金サービスへ遷移して精算を完結させる。

## 主要ドメイン用語

- **会計**: 1店舗・1回分の支払い（店名・会計時刻・合計金額・メニュー明細）
- **店舗（ノード）**: はしご内の各店舗訪問（1会計を内包）。代表者1名と参加メンバーを持ち、会計はその店の参加メンバー内で割り勘（MVPは均等割り）
- **ブランチ（分岐）**: ある店舗から次の店舗へ派生するエッジ。グループの分岐（別々のN次会）を表す。Git のブランチに着想（エッジであってノードではない）
- **はしご(イベント)**: 1回の飲み会。店舗（ノード）とブランチ（分岐）からなるグラフ（MVPは分岐ツリー）
- **代表者**: その会計を立て替えて支払った人
- **メンバー**: 割り勘の参加者
- **支払い源**: 代表者が立替に用いた原資（単一/複数）
- **依存関係(債務グラフ)**: 「誰が誰にいくら」を表す有向グラフ
- **矢印の列挙**: 債務（依存関係）を「誰→誰 ¥いくら」の矢印として店舗ごとに一覧化したもの
- **割り勘比重 / 定数金額**: 参加者ごとの負担割合 / 比重に依らない固定負担額
- **精算**: 送金により債務を解消すること（ことら送金/PayPay へ遷移）
- **レシート自動入力**: AIでレシートを会計データ(JSON)へ変換するサブスク機能

用語の詳細は [docs/domain.md](docs/domain.md) を参照。

## 技術スタック

Web SPA（React + TypeScript / Vite）＋ Go API（Connect-RPC / buf）＋ PostgreSQL を**モノレポ**で構成。FE は Feature-Sliced Design、BE はドメインルート構成。技術スタックの確定内容・アーキテクチャは [docs/architecture.md](docs/architecture.md) を参照。

## ディレクトリ構成

`apps/backend/`（Go）＋ `apps/frontend/`（React SPA・FSD）＋ `packages/proto/`（protobuf）の分離型モノレポ。全体像は [docs/architecture.md](docs/architecture.md)、各ディレクトリの役割・配置ルールは [docs/directory-structure.md](docs/directory-structure.md) を参照。

- **Git運用**: GitHub Flow。`main` + 機能ブランチ → PR でマージ。
- **コミット / ブランチ / PR**: 形式は縛らない。変更内容が分かれば自由でよい（厳密な規約・検証は設けない）。ブランチ名に対応 issue 番号を含めると追跡しやすい（任意の推奨）。
- **禁止事項（必ず守る）**:
  - **コミット / push / PR は、ユーザーの明示的な許可を得てから行う**（許可なく勝手に実行しない。レビュー前に勝手にコミットしない）。
  - `main` への直コミット / 直 push は禁止（必ず機能ブランチ → PR 経由）。
  - Co-Authored-By 等のトレーラ・署名（共著者表記・ツール署名）は付与しない（コミット・PR・issue いずれも）。

詳細は [docs/workflow.md](docs/workflow.md) を参照。

## よく使うコマンド

（横断コマンドは `Taskfile.yml` に集約予定。整備は後続 issue）

## ドキュメント

ドキュメントの案内は [docs/README.md](docs/README.md) を参照。主なもの:

- [docs/architecture.md](docs/architecture.md) — 技術スタック・アーキテクチャ・ディレクトリ構成の定義
- [docs/directory-structure.md](docs/directory-structure.md) — ディレクトリ構成の詳細
- [docs/requirements.md](docs/requirements.md) — 要件定義
- [docs/domain.md](docs/domain.md) — ドメイン用語・ドメインモデル
- [docs/workflow.md](docs/workflow.md) — 開発フロー
- [docs/adr/](docs/adr/) — アーキテクチャ決定記録（ADR）
